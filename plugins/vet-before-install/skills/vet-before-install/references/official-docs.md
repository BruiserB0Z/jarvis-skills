# The official pages behind each rule

The vendors' own documentation, read on 2026-10-14. When a rule in `SKILL.md` is questioned, check it here first; if a page has changed, the page wins and the skill gets corrected.

## Anthropic: Claude Code skills, plugins and security

| Rule in the skill | Page | What it says |
|---|---|---|
| A skill's `allowed-tools` grants tools without a prompt | https://code.claude.com/docs/en/skills | `allowed-tools`: "Tools Claude can use without asking permission during the turn that invokes this skill." Also: "A skill can grant itself broad tool access, so review the `allowed-tools` of skills checked into a repository before you run Claude Code there." "Deny and ask rules still override `allowed-tools`." |
| Hooks in a skill's frontmatter persist | Same page | `hooks`: "Hooks that Claude Code registers when the skill is invoked and keeps running for the rest of the session." |
| Shell injection (an exclamation mark directly before a backtick command) runs while the skill loads | Same page | "Injected commands never prompt for permission while the skill renders. Claude Code checks each one against your permission rules first." `"disableSkillShellExecution": true` in settings turns them off for user, project, plugin and additional-directory skills. |
| A skill folder can become a plugin | Same page | "add a `.claude-plugin/plugin.json` to a skill folder and it loads as a plugin named `<name>@skills-dir`, so it can bundle agents, hooks, and MCP servers." |
| Plugin hooks run automatically | https://code.claude.com/docs/en/plugins/components | "A hook runs something automatically at a point in Claude Code's lifecycle, such as after every file edit: a shell command, an HTTP request, an MCP tool call, a prompt to a model, or a subagent." |
| Monitors start with the session, with full rights | Same page | "A monitor is a shell command that Claude Code starts in the background when the session starts and keeps running until it ends." "It runs with your full user permissions and outside the sandbox." |
| `bin/` joins the PATH | Same page | "Files in `bin/` at the plugin root are on the `PATH` of the Bash tool's shell while the plugin is enabled, so Claude can run them as bare commands." "Plugin `bin/` directories come after the user's own `PATH` entries, so a plugin can't shadow `git`, `ls`, or another system command." |
| Plugin MCP servers start processes | Same page | "An MCP server gives Claude tools from an external system. Declare it in `.mcp.json` at the plugin root. This one starts a local server from a script inside the plugin". |
| Marketplace installs fetch Node packages | Same page | "For a marketplace-installed plugin, Claude Code installs eligible Node.js package dependencies automatically when it caches the plugin". |
| Anthropic does not audit MCP servers | https://code.claude.com/docs/en/security ("MCP security") | "We encourage either writing your own MCP servers or using MCP servers from providers that you trust." "Anthropic reviews connectors against its listing criteria before adding them to the Anthropic Directory, but does not security-audit or manage any MCP server." Also: plugins can add servers, "so reviewing `.mcp.json` doesn't show every server a session can load." |
| Try risky things in a virtual machine | Same page ("Protect against prompt injection") | "Use virtual machines (VMs) to run scripts and make tool calls, especially when interacting with external web services". |
| An example of a local server done right | https://code.claude.com/docs/en/vs-code ("The built-in IDE MCP server") | The extension's server "binds to `127.0.0.1` on a random port", and "Each extension activation generates a fresh random auth token, writes it to a lock file at `~/.claude/ide/<port>.lock`", readable only by the user running VS Code. Use it as the yardstick for any local server an extension opens. |

## Microsoft: VS Code extensions

| Rule in the skill | Page | What it says |
|---|---|---|
| An extension runs with the editor's permissions | https://code.visualstudio.com/docs/configure/extensions/extension-runtime-security | "The extension host has the same permissions as VS Code itself." |
| What the verified badge proves | Same page | The check mark "indicates that the publisher has proven domain-name ownership to the Marketplace"; the page presents it as an extra signal of trust, not a review of the code. |
| What the Marketplace does | Same page | Each new extension and update is scanned by several antivirus engines; reported and verified malicious extensions are removed, and VS Code uninstalls them if installed. |
| Command-line installs and publisher trust | Same page | Since VS Code 1.97 a first install from a third-party publisher asks the user to trust the publisher; for an extension installed from the command line, "the extension's publisher is not automatically trusted." |
| What to check before installing | Same page | Ratings and reviews, Q and A, issues, repository and licence, and the verified-publisher mark; "Report a concern" for anything suspicious. |

## GitHub: Actions and workflows

| Rule in the skill | Page | What it says |
|---|---|---|
| Pin to a full commit SHA, from the real repository | https://docs.github.com/en/actions/reference/security/secure-use ("Using third-party actions") | "Pin references to actions to a commit SHA in your workflow files." Check the SHA comes from the action's own repository, not a fork. |
| Why a tag is weaker | Same page | "there is risk to this approach even if you trust the author, because a tag can be moved or deleted if a bad actor gains access to the repository storing the action." |
| Read the action's code | Same page | "always audit the source code of the action", to "Ensure that the action is handling the content of your repository and secrets as expected." |
| What "Verified creator" means | Same page | The badge "indicates that the action was written by a team whose identity has been verified by GitHub." |
| Least privilege for the workflow token | Same page | Make sure "the `GITHUB_TOKEN` is granted the minimum required permissions", read-only by default and raised per job only as needed. |

## Statements in the skill that do not come from those pages

Each of these is from experience vetting tools for one small shop. Check them against the case in front of you before leaning on them.

- The red-flag list in section 2 (unrelated network calls, credential reads, obfuscated code, download-then-execute, instructions to hide things).
- "Install counts and stars can be inflated", and the advice to weigh fit over popularity.
- "A repository under a large company's organisation can be a research project": seen on 2026-10-14 with a 0.0.x VS Code extension published from a research team's repository.
- "Automated scanners both over- and under-call": security-teaching skills were flagged High or Critical by marketplace scanners while their files were plain guidance.
- "A mounted Docker socket is root on the host": widely documented by Docker, not re-read on 2026-10-14.
- The verdict categories and the report shape in section 7.
