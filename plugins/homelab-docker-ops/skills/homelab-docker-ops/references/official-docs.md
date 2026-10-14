# The official pages behind each rule

Docker's own documentation, read on 2026-10-12. When a rule in `SKILL.md` is questioned, check it here first; if a page has changed, the page wins and the skill gets corrected.

| Rule in the skill | Page | What it says |
|---|---|---|
| Names only work on a network you made | https://docs.docker.com/engine/network/drivers/bridge/ ("Differences between user-defined bridges and the default bridge") | "Containers on the default bridge network can only access each other by IP addresses... On a user-defined bridge network, containers can resolve each other by name or alias." |
| A running container can join a network | Same page ("Connect a container to a user-defined bridge") | `docker network connect my-net my-nginx` connects an already-running container to an existing network. |
| Containers on a shared network expose every port to each other | Same page | Publishing with `-p` is only needed for access from other networks or from outside Docker. |
| Data in the container dies with it | https://docs.docker.com/engine/storage/volumes/ | "When a container is destroyed, the writable layer is destroyed with it. Using a volume ensures that the data is persisted even if the container using it is removed." |
| An empty volume is filled from the image | Same page ("Mounting a volume over existing data") | An empty volume mounted over a directory that has files gets those files copied in; a non-empty volume hides what was there. |
| Volume or bind mount | Same page ("When to use volumes") | Volumes are managed by Docker and are not a good choice "if you need to access the files from the host"; use bind mounts for that. |
| Default logs are never rotated | https://docs.docker.com/engine/logging/configure/ | "By default, no log-rotation is performed"; the `local` driver is recommended because it rotates by default. |
| Host-wide log settings miss existing containers | Same page | "The changes take effect for newly created containers after the restart. Existing containers don't use the new logging configuration automatically." Options in `daemon.json` must be strings. |
| Reach a remote engine over SSH, not an open port | https://docs.docker.com/engine/security/protect-access/ | `docker context create --docker host=ssh://user@host name`; "The given USERNAME must have permissions to access the docker socket on the remote machine." |
| Who controls the daemon controls the host | https://docs.docker.com/engine/security/ ("Docker daemon attack surface") | Not quoted here: this section was opened but not read word for word on 2026-10-12. Read it before leaning on it. |

Not from Docker's documentation, but from this lab's own experience and proven on the host: the `/etc/cron.d` user-field rule, the parked-container schedule, and testing a connection from inside the calling container.
