# Docker Containers

This is the repository for the docker compose files that can be used in a Homelab.

Not every folder under `docker-apps/` is currently deployed — some are active, some
are running as an interim setup outside this repo, and some were evaluated and
parked or rejected. The [homelab README](https://github.com/ansasi/homelab#apps--status)
is the source of truth for current status; see [TODO.md](TODO.md) for what's
prioritized to be added next.

## CI

All compose files in `docker-apps` are checked in the `compose-lint` workflow.
The workflow runs `docker compose config` for each `docker-compose.yaml` on
pushes and pull requests.
