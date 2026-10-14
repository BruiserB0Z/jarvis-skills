# The official pages behind each rule

Wazuh's own documentation, read on 2026-10-12 (the "current" edition, with the manager on 4.14.8). When a rule in `SKILL.md` is questioned, check it here first; if a page has changed, the page wins and the skill gets corrected.

| Rule in the skill | Page | What it says |
|---|---|---|
| Custom IDs run from 100000 to 120000 | https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html | "Use ID numbers between 100000 and 120000 for custom rules to avoid conflicts with out-of-the-box system rules." |
| Where custom rules live | Same page | Small adjustments go in `/var/ossec/etc/rules/local_rules.xml`; "We recommend creating new rule files in /var/ossec/etc/rules/ directory for changes on a larger scale." |
| Never edit the shipped ruleset | Same page ("Changing existing rules") | "Modifications made to any rule file within the /var/ossec/ruleset/rules directory are overwritten during the upgrade process." Copy the rule into `etc/rules/` and add `overwrite="yes"`. |
| What an overwrite cannot change | Same page | "it is currently not possible to overwrite the if_sid, if_group, if_level, if_matched_sid, and if_matched_group labels. These tags are ignored when present in an overwrite rule". |
| Logtest sees a saved file; alerts need a restart | Same page | "saving the changes made to the rule files is enough. However, you need to restart the Wazuh manager to generate alerts based on these changes." |
| Three ways to run logtest, and its sessions | https://documentation.wazuh.com/current/user-manual/ruleset/testing.html | The dashboard (Tools, Ruleset test), the command line tool, or the server API. Each session loads its own rules and keeps a `firedtimes` counter; sessions close after 15 idle minutes by default. |
| What the levels mean | https://documentation.wazuh.com/current/user-manual/ruleset/rules/rules-classification.html | Level 0 is "Ignored... Used to avoid false positives" and does not appear on the security events dashboard; 3 is successful or authorised events; 12 is a high importance event. |

## Statements in the skill that do not come from those pages

Each of these is from running one small installation. They held there; check them against the documentation for the version in front of you before leaning on them elsewhere.

- The dashboard's relay to the server API (`/api/login`, `/api/request`, `/api/console/proxy`) and the `{"body": ..., "origin": "raw"}` shape for file uploads: found by use on 4.14.x, not read in the documentation.
- "Manager first, agents never newer than the manager": the usual Wazuh compatibility rule, not re-read on 2026-10-12.
- The service start order and `dpkg --audit` as a health check.
- "Logtest proves the rule, not delivery", the paired quiet-and-loud rule pattern, and the warning about closing a terminal during an upgrade.
