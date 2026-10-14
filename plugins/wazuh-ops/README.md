# wazuh-ops

A Claude Code skill for a small Wazuh deployment (one manager, a handful of agents) where the same person is both the analyst and the administrator.

It triages an alert in six questions and gives a plain verdict (real, true but harmless, or false positive); checks health in the order things depend on each other; writes custom rules in the right ID range and quiets noise with a narrow child rule paired with a loud one; explains what `wazuh-logtest` proves and what it does not; and deploys rule changes and upgrades with a way back. It also shows how to reach the server API through the dashboard when there is no API user.

Claude uses it whenever Wazuh comes up: an unfamiliar alert, a rule ID, a noisy dashboard, a silent log source, or a planned upgrade.

**Files:** `skills/wazuh-ops/SKILL.md` (the method) and `references/official-docs.md` (the Wazuh pages behind each rule, and which advice comes from experience). Write your own `references/site.md` beside them with your manager, agents and rule ID blocks; it is never published from here.

Install from the [repository root](../../README.md). MIT licence.
