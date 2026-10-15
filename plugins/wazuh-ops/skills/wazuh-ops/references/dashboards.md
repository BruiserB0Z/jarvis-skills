# Dashboards: the examples, and building, sharing and checking your own

Found by use on Wazuh 4.14.8, whose dashboard reports OpenSearch Dashboards 2.19.6 at `/api/status` (October 2026). Check against the version in front of you before leaning on the details.

## The two examples

Every panel uses fields every Wazuh alert carries (`rule.*`, `agent.name`, `timestamp`), plus `syscheck.*` and `data.srcip` where those modules report. The object IDs start with `wazuh-ops-mc-` and `wazuh-ops-nf-`, so they cannot collide with anything already on an install.

**Morning check** (`assets/dashboards/morning-check.ndjson`, opens on the last 24 hours)

| Panel | What it answers |
|---|---|
| High alerts (level 12 and up) | One number. It should be 0; anything else is read first. |
| Medium alerts (level 7 to 11), All alerts | Two numbers, for scale. |
| Alerts over time, by severity | Stacked bars per time slot: low grey, medium amber, high red. A bump at the same hour every night is usually a scheduled job; automatic updates show up as package and file-integrity alerts. |
| Alerts by agent, by severity | Which machine the alerts come from. |
| What is making the noise: top rules | Rule, description, count and level. |
| Files that changed (integrity monitoring) | Query `rule.groups:syscheck`: the file, the agent, the kind of change. |
| Logins, good and failed | Query `rule.groups:authentication_success or rule.groups:authentication_failed`: the event, the agent, the source address. Often empty, which is good news. |

**Noise finder** (`assets/dashboards/noise-finder.ndjson`, opens on the last 7 days)

| Panel | What it answers |
|---|---|
| The five noisiest rules over time | Which rules fill the feed, and whether they come in bursts (a job) or steadily (a setting). |
| Each rule, on each agent | Rule, description, agent, count and level. The same rule on two agents is often one cause. |
| Medium and high alerts, by rule group | Query `rule.level >= 7`: which families of rules the attention-worthy alerts come from. |

## Importing

- **By API (tested):** `POST /api/saved_objects/_import` with the file as a multipart form field named `file` and the header `osd-xsrf: true`. Add `?overwrite=true` only to replace your own earlier copy of the same objects. Both files imported with every object and no errors.
- **In the dashboard:** the Saved objects page, under Dashboards Management, has an Import button that takes the same file. The menu path was not walked for this note.
- The files point at the index pattern whose ID is `wazuh-alerts-*`, which a stock install creates. On an install where that pattern has a generated ID instead, expect a missing-reference message on import and point it at the alerts index pattern (not tested here).

## Building your own

- **By hand:** create each visualization on the alerts index pattern, then a dashboard that holds them. Good for one-offs.
- **As code, for anything you will keep or share:** `POST /api/saved_objects/_bulk_create` with the header `osd-xsrf: true`. Without `?overwrite=true` it refuses to replace an existing ID, so a first build cannot clobber anyone's work. The saved-objects API accepted the dashboard login as HTTP basic authentication.
- **Shape of a visualization:** its definition is a JSON string in `visState`; its query is in `kibanaSavedObjectMeta.searchSourceJSON`, whose `indexRefName` names an entry in `references` pointing at the index pattern. A dashboard lists its panels in `panelsJSON` (grid positions on a 48-column grid) and names each visualization through `references`.
- **The order of `aggs` is the nesting.** A `terms` split placed before the `date_histogram` ranks across the whole time range; placed after, it ranks inside each slot.
- **Labelled bands:** a `filters` aggregation with one labelled query per band. Colours are set by label in `uiStateJSON`: `{"vis":{"colors":{"<label>":"#hex"}}}`. The examples use low `#8c8c8c`, medium `#fab219`, high `#d03b3b`.

## Exporting to share

- `POST /api/saved_objects/_export` with an explicit list of objects and `"includeReferencesDeep": false`. A deep export also carries the index pattern, and with it every field name on the install, custom fields included.
- Drop `updated_at` and `version` from each line; they are the install's own bookkeeping.
- Read the file for names before it goes anywhere: agent names, host names, addresses, user names, e-mail addresses. A definition can hold them in a query, a filter or a title.

## Checking that it renders

- Open the dashboard and read every panel, at the range it opens on and at a longer one. An empty panel is checked against the time range before it is called broken.
- For a scripted check, a browser automation tool can sign in with `POST /auth/login` (a JSON body with `username` and `password`, and the header `osd-xsrf: true`), which sets the session cookie, then load `/app/dashboards#/view/<dashboard id>` and take a screenshot. The credential goes straight from where it is stored into the request, and the screenshots stay private.
