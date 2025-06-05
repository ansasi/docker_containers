# Docker Containers

This is the repository for the docker compose files that can be used in a Homelab.

## CI

All compose files in `docker-apps` are checked in the `compose-lint` workflow.
The workflow runs `docker compose config` for each `docker-compose.yaml` on
pushes and pull requests.
