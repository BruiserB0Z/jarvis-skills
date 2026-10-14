# vet-before-install

A Claude Code skill for deciding whether a third-party skill, Claude Code plugin, MCP server, editor extension, GitHub Action or small open-source tool should be installed.

Skills, plugins and extensions run with your own permissions, and the risky parts (hooks, MCP servers, startup scripts, automatic dependency installs, permission grants) never appear in a marketplace listing. This skill has Claude find out who really publishes the thing and what each badge actually proves, read every file that will run or steer the agent, map what it can reach, check how it is kept up and whether it fits, plan a safe trial, and give a verdict in a fixed shape: install, install with conditions, mine and skip, decline, or not enough evidence. The decision stays yours.

Claude uses it whenever you ask whether something is safe, or before running an install command or adding a workflow step.

**Files:** `skills/vet-before-install/SKILL.md` (the method) and `references/official-docs.md` (the Anthropic, Microsoft and GitHub pages behind each rule). Write your own `references/site.md` beside them with your install rules and past verdicts; it is never published from here.

Install from the [repository root](../../README.md). MIT licence.
