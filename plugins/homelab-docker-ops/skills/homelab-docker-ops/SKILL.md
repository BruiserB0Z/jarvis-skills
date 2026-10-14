---
name: homelab-docker-ops
description: Run, fix and extend containers on a self-hosted Docker host (a home lab or small-office server, with or without Portainer and Compose) where you operate other people's images rather than build your own. Use this skill whenever the work touches a Docker host you look after - deploying a new container or stack, a container that cannot reach another one, a monitor that goes red after a reboot, data or settings that vanish after a restart or re-create, a container that should run on a schedule, logs filling a disk, a locally built image, a Portainer stack, reaching a remote Docker engine, or a security look at the host. Reach for it before the first docker command, even when the task looks like a one-liner, because the failures it covers are silent ones - the container runs, reports success, and is quietly wrong.
---

# Home-lab Docker operations

Most Docker guidance is written for developers packaging their own application. This is for the other job: keeping a handful of third-party containers healthy on a small host that one person looks after. The failures here are rarely crashes. They are things that work today and break quietly later: an address that moves at the next restart, a history file that dies with its container, a log that grows for a year.

## Start here

1. **Read `references/site.md` if it exists.** It holds the facts of the host this skill is installed for: how to connect, what runs there, where each stack's files live, and what is already known to be wrong. If there is no site file, gather those facts with the inventory commands below before changing anything, and offer to write one.
2. **Look before you touch.** Run the read-only inventory first. A note or a diagram of the host is a claim about the past; `docker inspect` is the present.
3. **Say which things you saw and which you are predicting.** "The second scan said 'no changes'" and "this should survive a reboot" are different kinds of statement, and the owner needs to know which one they are getting.

## Read-only inventory

These change nothing and answer most questions. Prefix with `sudo` where the host needs it.

```bash
docker ps -a --format 'table {{.Names}}\t{{.Status}}\t{{.Image}}'
docker compose ls -a
docker network ls
docker volume ls
docker info --format 'logging={{.LoggingDriver}} root={{.DockerRootDir}}'
```

One line per container with what usually matters (where it came from, restart rule, networks, published ports):

```bash
docker inspect -f '{{.Name}} | project={{index .Config.Labels "com.docker.compose.project"}} | restart={{.HostConfig.RestartPolicy.Name}} | user={{.Config.User}} | nets={{range $k,$v := .NetworkSettings.Networks}}{{$k}}({{$v.IPAddress}}) {{end}}' NAME
docker inspect -f '{{range .Mounts}}{{.Type}} {{if .Name}}{{.Name}}{{else}}{{.Source}}{{end}} -> {{.Destination}}{{println}}{{end}}' NAME
```

An empty `project=` means the container was made by hand with `docker run`; a name means a Compose project made it, and its file is the thing to change, not the container.

## The five quiet failures

### 1. Data that dies with the container

A container's own filesystem is a scratch layer. When the container is removed, that layer goes with it; only volumes and bind mounts survive. `--rm`, a re-create by Compose, or an image update all remove the container.

The trap is that an image's documentation usually tells you about the obvious data folder and not about everything the program writes. A tool may keep its cache on a declared volume and its history, licence or state in a sibling folder nobody mounted. It then runs perfectly, prints "saved", and starts from nothing next time.

- **Find every place it writes.** Read the image's Dockerfile for `VOLUME` lines, then the program's own configuration defaults and environment variables for paths (`*_DIR`, `*_PATH`, `~/.something`). A `VOLUME` list that is shorter than the list of paths the program writes is the warning sign.
- **Prove persistence with a second run.** Run it, run it again, and check the second run remembers the first. For a long-lived container, re-create it and check. Nothing else counts as proof.
- **Mind who owns the folder.** Many images run as their own unprivileged user. A bind-mounted host folder keeps its host ownership, so it must be handed to that user id (`chown UID:GID`) or writes fail. A new, empty named volume is different: Docker fills it from the image, ownership included.
- **Bind mount or named volume?** Use a bind mount when a person or a backup needs to see the files on the host; a named volume when only the container does.

### 2. Containers that cannot find each other

On Docker's built-in `bridge` network, containers reach each other by IP address only, and those addresses are handed out afresh at start-up. Anything pointed at `172.17.0.x` works until the next reboot changes the order.

On a network you created (and every Compose project gets one of its own), containers find each other **by name**. That is the fix: put both containers on a shared user-defined network and use the container name.

- Check what is already shared before creating anything: `docker network inspect NETWORK -f '{{range .Containers}}{{.Name}} {{end}}'`. A suitable network often exists.
- Join a running container without re-creating it: `docker network connect NETWORK CONTAINER`. This survives restarts. It does not survive a re-create, so for a Compose-made container put the network in the Compose file.
- **Test from inside the container that makes the call, never from your own machine:**
  ```bash
  docker exec CALLER getent hosts TARGET
  docker exec CALLER curl -s -o /dev/null -w '%{http_code}\n' -m 8 http://TARGET:PORT/
  ```
  A URL that answers from your desk proves nothing about what the calling container can reach. The host's own LAN address in particular often fails from inside a container on that same host.

### 3. A container that should run on a schedule

Three shapes work. Pick by how the owner wants to see and trigger it.

| Shape | How | Good when |
|---|---|---|
| Parked container | Create it once with `restart: "no"`. It runs, exits 0, and stays in the list. `docker start NAME` runs it again. | The owner wants a Start button (Portainer) and a schedule. |
| Scheduled `docker run --rm` | Cron runs the whole command each time. | Nothing needs to be visible between runs. |
| The program's own watch mode | A long-lived container that sleeps between runs. | The interval is short and exact timing does not matter. It re-runs at every restart. |

For the parked container, schedule it with one file in `/etc/cron.d`:

```
0 11 * * 0 root /usr/bin/docker start NAME
```

- A file in `/etc/cron.d` needs the user field (`root` above). Without it the line is silently ignored. The file must have Unix line endings, be owned by root, and have no dot in its name.
- Check the host's time zone (`timedatectl`) before writing the hour, and the real path of docker (`which docker`).
- **Prove the schedule fires without waiting for it:** install a temporary copy of the line set two minutes ahead, wait, then read `journalctl -u cron` for the `CMD` line and `docker inspect -f '{{.State.StartedAt}} {{.State.ExitCode}}' NAME`. Remove the temporary file in its own step, after reading the result.
- Any state the job keeps between runs is subject to failure 1.

### 4. Logs that grow for ever

Docker's default log driver, `json-file`, does no rotation. A chatty container quietly builds a file of hundreds of megabytes under `/var/lib/docker/containers/`.

```bash
for c in $(docker ps -a --format '{{.Names}}'); do echo "$c $(du -h "$(docker inspect -f '{{.LogPath}}' $c)" | cut -f1)"; done
```

- Per container, in Compose: `logging: { driver: json-file, options: { max-size: "10m", max-file: "3" } }`.
- Host-wide, in `/etc/docker/daemon.json`: set `"log-driver": "local"` (rotates by default), or `json-file` with `log-opts`. Options there must be strings.
- **A host-wide setting only applies to containers created after Docker restarts.** Existing containers keep their old logging until they are re-created, so plan that as a change, with the owner's yes.

### 5. Images you cannot reproduce

- A tag such as `latest` is a moving label. What runs after the next pull may not be what ran before. Prefer a version tag, and record what is running (`docker image inspect -f '{{.Id}}' IMAGE`).
- For an image built on the host from someone's source: record the source commit, tag it clearly as local, and set `pull_policy: never` in Compose so nothing tries to fetch it. A registry may refuse an image its README says is public; ask the registry before writing instructions around it.
- Read the Dockerfile before building anything from a stranger's repository: what user it runs as, what it downloads, whether dependencies are pinned.

## Deploying something new

Work in three steps, and keep the host itself free of half-finished work.

1. **Plan.** From the image's own Dockerfile and documentation, list: every path it writes (failure 1), the user id it runs as, ports, what it talks to on the network, and any secret it needs. Decide how it will be started, updated and removed.
2. **Stage.** Write the Compose file on your own machine. Keep one folder per application on the host (`~/app/` holding the Compose file beside its data folders) so the configuration and the data live together and can be read with `cat`. Copy the finished file over; check line endings.
3. **Deploy and prove.** `docker compose config -q` to validate, then `docker compose up`. Check the exit code or health, read the logs, and run whichever proof the failures above call for (second run, name lookup from inside, a fired schedule). Then write down what shipped and how it was proven.

A stack started with `docker compose` from a file on the host shows in Portainer, with working start, stop and logs, but Portainer cannot edit it; the file on the host stays the place to change it.

## Updating a container

An update is a re-create: the old container is removed and a new one is made from a newer image. Everything in failure 1 applies, and most of the risk is decided before anything stops.

1. **Find out what really runs and what is really available.** A moving tag (`latest`, `lts`, a bare major number) says nothing. Read the running version from the image's label or from the program itself (`docker exec NAME program --version`), and the newest release from the project's own release page. Ask the registry whether the exact version tag exists before writing it anywhere.
2. **Read every release between the two, not only the newest.** Look for the "breaking" and "upgrade" sections and check each one against what this installation actually uses (flags, labels, the names of its environment variables), not against what the note assumes. A database that migrates itself at first start is usually one-way: the old version cannot read it afterwards.
3. **Take two safety nets.** A snapshot of the whole machine if it is a VM; then, per application, a copy of its data volume taken while the container is stopped (`tar -C /var/lib/docker/volumes/VOLUME/_data -czf FILE.tar.gz .`, then mode 600). Check the copy before trusting it: the SHA-256 of the database file inside the archive should equal the live one.
4. **Stage before you stop.** Write or edit the Compose file, validate it, and pull the new image, all while the old container still runs. Validate with `docker compose config -q` and never without the `-q`: the plain command prints the resolved file, secrets included. For a container made by hand, a Compose file that names the same volume and network with `external: true` adopts its data. A value that was set by hand can go into an `.env` file beside it straight from the old container, without being displayed.
5. **Switch, and keep the way back open.** Stop, copy the data, and for a hand-made container rename the old one aside (`docker rename NAME NAME-parked`) rather than removing it; then start the new one. Remove the parked container only when the owner has seen the application working. For a Compose-made container the way back is a dated copy of the Compose file, and `docker compose up -d --no-deps SERVICE` re-creates one service and leaves its neighbours alone.
6. **Prove it from where it is used,** with the same checks before and after: the version it reports, its log from the first line (a migration should say it finished), a request from inside a container that depends on it, and the state of whatever monitors it. Then the owner's own eyes on it.

What bites:

- **A reverse proxy may wait for the health check.** Seen with Traefik's Docker provider: it does not route to a container until its health check has passed once. An image that checks once a minute, with no start period, is unreachable for a minute after every start however fast the program is. Read the image's health settings (`docker image inspect -f '{{json .Config.Healthcheck}}' IMAGE`) before saying how long an outage will last.
- **The old container's log goes with it.** If anything looked odd beforehand, save it first (`docker logs NAME > FILE 2>&1`). That log may be the only place a slow failure shows, such as a certificate renewal that has been refused for days.
- **A snapshot freezes the guest for a moment.** A process that dislikes the clock jumping may exit, and its restart policy brings it back. Expect it, and do not take it for a fault in the update.
- **Moving a hand-made container to Compose usually takes it off the default `bridge` network.** Find what still points at a `172.17.x.x` address first (failure 2), because those stop working.
- **Pin as you go.** Write the exact version into the Compose file, so the next update is a decision and not an accident.

## Security, in proportion

A home lab is not a bank, but the same few facts decide who owns the host.

- **The Docker socket is root.** Any container with `/var/run/docker.sock` mounted, and anyone who can log in to a management UI that has it, controls the whole host. Know which containers have it and why.
- **The `docker` group is root too.** So is passwordless `sudo`. Treat them the same.
- **Do not open the Docker API on a TCP port to save typing.** It would be open to every device on that network. To drive a remote engine, use SSH, which needs no new opening: `docker context create NAME --docker host=ssh://USER@HOST`. The remote user needs access to the socket.
- **Published ports** bind to every interface by default. Use `127.0.0.1:PORT:PORT` when only the host or a reverse proxy on it needs the port.
- **Secrets in environment variables are visible** to anyone who can run `docker inspect`. Prefer a mounted file with tight permissions.
- Where the application tolerates it, add `security_opt: [no-new-privileges:true]`, `cap_drop: [ALL]`, and a read-only root with a `tmpfs` for scratch space. Test after each one; some images need to write at start-up.

## When something is wrong

Work outward from the container, and change one thing at a time.

1. `docker ps -a` - is it running, restarting, or exited, and with what code?
2. `docker logs --tail 50 NAME` - what did it say last?
3. `docker inspect` (formats above) - is it on the network, with the mounts, you think it is?
4. From inside it (`docker exec`) - can it resolve and reach what it needs?
5. Only then change something, and re-run the same check to see the difference.

## References

- `references/site.md` - this installation's host: access, containers, paths, known issues. Read first. Private to the site; leave it out of anything shared.
- `references/official-docs.md` - the pages of Docker's own documentation each rule above rests on. Read when a rule needs checking against the source or the owner asks why.
