---
name: wazuh-ops
description: Operate a small Wazuh deployment (one manager, a handful of agents) as its analyst and its administrator - triage an alert, decide whether it is a false positive, write or tune custom rules and decoders, quiet noise without going blind, test with wazuh-logtest, check health after a reboot or an upgrade, upgrade the manager and agents safely, and reach the server API through the dashboard when there is no API user. Use this skill whenever Wazuh comes up at all - an alert someone does not recognise, "is this one real?", a rule ID, a level, an agent that looks disconnected, a noisy dashboard, a dashboard to build, import or share, a new log source that produces no alerts, a planned Wazuh update, or any change to rules, decoders or ossec.conf. Use it before answering from memory, because Wazuh's behaviour is specific (rule ID ranges, what survives an upgrade, what logtest does and does not prove) and a confident guess here either hides a real alert or buries the owner in false ones.
---

# Wazuh operations

For a small Wazuh installation run by one or two people. Two jobs meet here: the analyst's (what is this alert, and does it matter?) and the administrator's (rules, decoders, upgrades, health). In a small shop they are the same person, so every tuning decision is also a detection decision. Quieting a rule is choosing not to be told something.

## Start here

1. **Read `references/site.md` if it exists.** It holds this installation's facts: where the manager is and how to reach it, the agents, the custom rule files and their ID ranges, known false positives, and what the owner has decided. Without it, gather those facts first (see "Reaching the manager") and offer to write one.
2. **Times are UTC.** Wazuh stamps events in UTC. Convert before telling a person when something happened, and before lining an alert up against what they remember doing.
3. **Say what you saw and what you infer.** "The alert's log line names `who` and `/etc/nsswitch.conf`" is seen. "A package update added that profile" is a guess until checked. Keep them apart in the answer.

## Triage: what is this alert?

Read the alert itself before forming a view. Six things, in this order:

1. **Which agent.** Agent `000` is the Wazuh manager watching itself, not the servers it protects. An alarming alert on `000` is about the monitoring box.
2. **The rule.** ID, level, description, groups, and how many times it fired. A level describes the rule's author's opinion of the event class, not this event.
3. **The full log line.** The description is a label; the log line is the evidence. Read every field: the program, the user, the source, the object acted on.
4. **The exact time**, converted, and **what else happened in that second or minute.** A login, a scheduled job, a package upgrade, a reboot. Most false positives are the system reacting to something the owner did on purpose.
5. **Is it new?** First time ever, first time since a change, or the hundredth this week?
6. **What would the dangerous version look like**, and is anything in the log line consistent with it?

Then give a verdict in plain words: **real** (act), **true but harmless** (it happened, it is fine, here is why), or **false positive** (the rule misread something). State the evidence for the verdict, what was not checked, and whether any action is proposed. An unfamiliar alert that turns out harmless is still worth one line in the site file so the next person does not start from zero.

Do not tune a rule in the same breath as triaging it. Understand first; quiet later, and narrowly.

## Health: is Wazuh actually working?

After a reboot, an upgrade or a scare, check in this order. Each step depends on the one before.

1. **Services on the manager host:** `wazuh-indexer`, then `wazuh-manager`, then `wazuh-dashboard`, plus `filebeat`. If any are down, start them in that order.
2. **Packages are whole:** `sudo dpkg --audit` prints nothing on a healthy Debian or Ubuntu host. If it lists anything, `sudo dpkg --configure -a`.
3. **Versions agree:** the indexer, manager and dashboard packages on the same version.
4. **Agents:** each one `active`, with a recent last keep-alive. A port being open is not the same as an agent being connected.
5. **Fresh events:** an alert stamped after the event you are recovering from. This is the only proof the whole pipe works end to end.

## Writing and changing rules

- **Custom rule IDs live between 100000 and 120000.** Anything else risks colliding with the shipped ruleset.
- **Custom rules live in `/var/ossec/etc/rules/`.** `local_rules.xml` for small adjustments, a named file per subject for anything larger. Give each file its own block of IDs and record the blocks in the site file, so two files never claim the same number.
- **Never edit `/var/ossec/ruleset/`.** An upgrade overwrites it. To change a shipped rule, copy it into a custom file and add `overwrite="yes"`. The `if_sid`, `if_group`, `if_level`, `if_matched_sid` and `if_matched_group` tags cannot be overwritten; they are ignored in an overwrite rule.
- **Quiet with a child rule, not by changing the parent.** A level-0 child (`<if_sid>PARENT</if_sid>` plus the narrowest match that describes the expected event: this user, from this source, doing this) silences the expected case and leaves the rule alive for everything else.
- **Pair every quiet rule with a loud one.** If the expected administrator login from the expected address is quieted at level 0, add a sibling at a high level for the same event from anyone or anywhere else. The quiet rule then becomes an allow-list, and whatever falls outside it is exactly the alert worth having.
- A level-0 rule does not appear on the dashboard at all. That is the point, and the risk: write the match so it cannot swallow more than you meant.

### A log source that produces nothing

If a new log file is being collected and no alert ever appears, check decoding before writing rules. Paste a real line into logtest. If phase 2 shows only a generic decoder and no fields, no rule can match it: the source needs a decoder first (custom decoders live in `/var/ossec/etc/decoders/`), then a base rule at level 0 that later rules hang from.

## Testing: what logtest proves and what it does not

`/var/ossec/bin/wazuh-logtest` (also the dashboard's ruleset test, and the server API) runs a pasted log line through pre-decoding, decoding and rule matching, and shows all three phases.

- Saving a rule file is enough for logtest to see the change. **Alerts need a manager restart.**
- A logtest session keeps its own counters. To test a rule that needs several events in a time window, paste the line the required number of times in one session.
- **Logtest proves the rule. It does not prove delivery.** A rule can pass logtest perfectly while the real events never reach the manager, arrive in a different shape, or carry a field under another name. After deploying, cause or wait for a real event and find its alert. If nothing arrives, turn on JSON archiving for a short while to see the raw incoming events, then turn it off again.
- Test with real lines from the real source, not lines typed from memory.

## Deploying a rule or decoder change

1. Keep a copy of the file as it was.
2. Make the change; run the affected log lines through logtest, including one that must still alert.
3. Validate the manager's configuration before restarting it. A manager that will not start is worse than a noisy one.
4. Restart the manager.
5. Confirm the rule is loaded, then confirm a real event produces the expected result.
6. Record the change, the IDs used, and how it was proven in the site file.

## Upgrades

- **Manager first, agents after.** An agent should never be newer than its manager.
- Read the release notes for the version in between; custom files in `etc/` survive, edits to `ruleset/` do not.
- The indexer package is large and slow to download. **Let the package manager finish in the terminal that started it.** Closing that terminal mid-upgrade can leave services stopped and the host refusing to shut down cleanly.
- Afterwards run the health check above, top to bottom, and write the new version into the site file.

## Reaching the manager

Prefer the route that needs the least privilege for the job.

- **The dashboard** is enough to read alerts and to test rules.
- **The server API through the dashboard**, when the only credential is a dashboard login and there is no API user or shell. The dashboard relays requests to the server API for a signed-in user:
  1. `POST https://DASHBOARD/api/login` with basic authentication, the header `osd-xsrf: true` and the body `{"idHost":"default","force":true}`. Keep the session cookies.
  2. `POST https://DASHBOARD/api/request` with `{"method":"GET","path":"/agents","body":{},"id":"default"}`. `path` is any server API path; `method` may be GET, PUT or DELETE.
  - Useful paths: `/manager/info`, `/agents`, `/rules/files?relative_dirname=etc/rules`, `/decoders/files?relative_dirname=etc/decoders`, `/rules?relative_dirname=etc/rules`, `/manager/configuration/validation`, `/logtest`, `/manager/restart`.
  - Uploading a rule file needs the body in this shape: `{"body":"<the xml>","origin":"raw"}`. Plain JSON is refused.
  - Indexer queries go through `POST https://DASHBOARD/api/console/proxy?path=INDEX_PATH&method=GET` with the same authentication and header.
- **A shell on the manager** for package work, service control and `ossec.conf`.

Load the password straight into the call that uses it. Never print it, the login response, or a command line that contains it.

## Noise, in proportion

- Vulnerability detection on a Linux host produces thousands of records, most of them kernel packages. That is inventory, not an incident. Read it by "what is exposed and reachable", not by count.
- Scanner traffic against a web server (bursts of 404s and 403s) is weather. It matters when a source gets a 200 where it should not.
- If one rule produces most of the dashboard, find out what it is reacting to before lowering it. The usual answer is a scheduled job or an administrator's own routine, which calls for a narrow child rule.

## Dashboards

A dashboard earns its place by answering a question someone asks often: "is anything serious?", "what is making the noise?". Build on standard alert fields, so it survives an upgrade and works on another install.

- **Two examples ship with this skill,** in `assets/dashboards/`: `morning-check.ndjson` (the last day: anything serious, what is noisy, which files changed, who logged in) and `noise-finder.ndjson` (the last week: which rules fill the feed, on which agents, and when). Import them under Dashboards Management, Saved objects, Import. They expect the stock index pattern whose ID is `wazuh-alerts-*`. How they were built, and how to build and share your own: `references/dashboards.md`.
- **Split a chart by `rule.id`, not `rule.description`.** Some descriptions carry the event's own values (an address, a user name), and then every event becomes its own series.
- **Rank before you slice.** In a "top five over time" chart, the top-five split must come before the date histogram. The other way round, each time slot picks its own top five and the legend fills with dozens of rules.
- **Severity bands are queries with labels, not colours alone.** The examples use low (level 0 to 6), medium (7 to 11) and high (12 and up), each named in the legend, so the meaning survives a colour-blind reader and a printout.
- **An empty panel is an answer.** "No results" on failed logins is good news. Check the time range before calling it broken.
- **Look at the result.** A saved object that imports cleanly can still draw nonsense. Open the dashboard and read every panel before calling it done.
- **Share the definition, never a screenshot of a live dashboard.** A screenshot shows agent names, addresses and file paths. An exported file holds only the definitions; read it for names anyway before it leaves the building.

## References

- `references/site.md` - this installation: manager, access, agents, custom rule files and ID blocks, known false positives, the owner's decisions. Read first. Private to the site; leave it out of anything shared.
- `references/official-docs.md` - the pages of Wazuh's own documentation the rules above rest on, and which statements here come from experience instead.
- `references/dashboards.md` - building, exporting, importing and checking dashboards, and what each example panel shows.
- `assets/dashboards/` - the two example dashboards, as importable saved-object files.
