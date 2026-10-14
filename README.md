# Claude skills for small self-hosted setups

Three Claude Code skills for anyone who runs their own infrastructure on a small scale: a home lab, a small office, a one-person business. Each one came out of real work on real machines, and each one has Claude check before it acts.

| Skill | What it is for |
|---|---|
| [homelab-docker-ops](plugins/homelab-docker-ops) | Running other people's containers on a small Docker host, and catching the quiet failures that leave a container running but wrong. |
| [wazuh-ops](plugins/wazuh-ops) | Being both the analyst and the administrator of a small Wazuh install: is this alert real, and how do you quiet noise without going blind? |
| [vet-before-install](plugins/vet-before-install) | Deciding whether a skill, plugin, MCP server, editor extension or GitHub Action is safe to install, by reading what it will actually do. |

## Install

In Claude Code:

```
/plugin marketplace add BruiserB0Z/jarvis-skills
/plugin install vet-before-install@bruiserb0z-jarvis
```

Swap in `homelab-docker-ops` or `wazuh-ops` for the others. Read a skill before you install it. Each one is plain Markdown, and the third explains why that habit matters.

## How each skill is built

- `SKILL.md` holds the general method. Nothing in it is tied to one installation.
- `references/official-docs.md` lists the vendor's own pages each rule rests on, with the quoted line, and says plainly which advice comes from experience instead.
- `references/site.md` is yours to write: your hosts, how you reach them, what you have decided. Each skill reads it first if it exists. None is included here, and `.gitignore` keeps any copy out of this repository.

## Security

Every push runs ChiefGyk3D's [git-your-ship-together](https://github.com/ChiefGyk3D/git-your-ship-together) workflows, pinned to an exact commit: a secret scan over the full history now, and code scanning once the repository is public. If you find a problem, open an issue, or use GitHub's private vulnerability reporting for anything sensitive.

## Licence

MIT. See [LICENSE](LICENSE).

---

Written by Jarvis, an AI assistant built on Claude, with BruiserB0Z.
