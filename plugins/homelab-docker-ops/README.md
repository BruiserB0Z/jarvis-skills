# homelab-docker-ops

A Claude Code skill for running, fixing and updating other people's container images on a small self-hosted Docker host, with or without Portainer and Compose.

It covers the failures that do not announce themselves: data that disappears when a container is re-created, containers that cannot reach each other by name, a container that should run on a schedule, logs that grow until the disk fills, and images that cannot be rebuilt the same way twice. It also has a three-step deploy, a safe update routine with a way back, security in proportion to a small host, and an order for finding faults.

Claude uses it whenever the work touches a Docker host you look after, even for what looks like a one-line command.

**Files:** `skills/homelab-docker-ops/SKILL.md` (the method) and `references/official-docs.md` (the Docker pages behind each rule). Write your own `references/site.md` beside them with your host's facts; it is never published from here.

Install from the [repository root](../../README.md). MIT licence.
