# Docker Containers

This is the repository for the docker compose files that can be used in a Homelab.

Not every folder under `docker-apps/` is currently deployed — some are active, some
are running as an interim setup outside this repo, and some were evaluated and
parked or rejected. The [homelab README](https://github.com/ansasi/homelab#apps--status)
is the source of truth for current status; see [TODO.md](TODO.md) for what's
prioritized to be added next.

## Maintenance status

The [homelab README](https://github.com/ansasi/homelab#apps--status) remains the
source of truth for where applications run. This table defines the maintenance
scope of the Compose stacks in this repository:

- **Used & maintained** — the application is in use, including interim LXC
  deployments, and its Compose stack should receive priority correctness,
  security, and upgrade reviews.
- **Maintained only** — the application is not currently used, but the Compose
  stack is kept viable for possible future deployment.
- **Neither** — the application was rejected or decommissioned. Its files are
  retained as reference and are not proactively upgraded or repaired.

| Stack | Repository status | Current position |
|---|---|---|
| [Hermes](docker-apps/ai/hermes) | Used & maintained | Active on the dedicated Raspberry Pi 4 host. |
| [Ollama](docker-apps/ai/ollama) | Maintained only | Deferred while no suitable GPU is available. |
| [Open WebUI](docker-apps/ai/openwebui) | Maintained only | Deferred with the local AI stack. |
| [Home Assistant](docker-apps/automation/homeassistant) | Maintained only | Deferred until there are IoT devices to manage. |
| [ERPNext](docker-apps/business/erpnext) | Neither | Rejected as too heavy and complex; Twenty covers the requirement. |
| [Twenty](docker-apps/business/twenty) | Used & maintained | Used through an interim LXC deployment; migration to Docker or Kubernetes is planned. |
| [Dockge](docker-apps/management/dockge) | Neither | Rejected in favor of Portainer. |
| [Homepage](docker-apps/management/homepage) | Used & maintained | Active on the main Docker host. |
| [Portainer](docker-apps/management/portainer) | Used & maintained | Active on the Docker, Datia, and Hermes hosts. |
| [Watchtower](docker-apps/management/watchtower) | Neither | Decommissioned in favor of Renovate and reviewed deployments. |
| [ARRs](docker-apps/media/arrs) | Used & maintained | Active on the main Docker host. |
| [Audiobookshelf](docker-apps/media/audiobookshelf) | Used & maintained | Used through an interim LXC deployment; migration to Docker or Kubernetes is planned. |
| [Calibre-Web](docker-apps/media/calibre-web) | Used & maintained | Active on the main Docker host. |
| [Jellyfin](docker-apps/media/jellyfin) | Used & maintained | Active in a dedicated LXC for direct iGPU access. |
| [Plex](docker-apps/media/plex) | Maintained only | Deferred and currently superseded by Jellyfin. |
| [Grafana](docker-apps/monitoring/grafana) | Used & maintained | Active on the Docker and Datia hosts. |
| [Netdata](docker-apps/monitoring/netdata) | Maintained only | Deferred while Grafana and Uptime Kuma cover monitoring. |
| [Uptime Kuma](docker-apps/monitoring/uptime-kuma) | Used & maintained | Active on the Docker and Datia hosts. |
| [Pi-hole and dnsproxy](docker-apps/networking/pihole) | Used & maintained | Active on the Raspberry Pi Zero DNS host. |
| [Technitium](docker-apps/networking/technitium) | Maintained only | Deferred while Pi-hole remains the DNS service. |
| [Traefik](docker-apps/networking/traefik) | Used & maintained | Active on the Docker, Datia, and Hermes hosts. |
| [UpSnap](docker-apps/networking/upsnap) | Used & maintained | Active on the Raspberry Pi Zero DNS host. |
| [WireGuard Easy](docker-apps/networking/wireguard) | Maintained only | Deferred while the FritzBox WireGuard service is used. |
| [Immich](docker-apps/storage/immich) | Used & maintained | Active on the main Docker host. |
| [Nextcloud](docker-apps/storage/nextcloud) | Maintained only | Deferred while Proton Docs and Drive are used. |
| [Nextcloud AIO](docker-apps/storage/nextcloud-aio) | Maintained only | Deferred while Proton Docs and Drive are used. |
| [Paperless-ngx](docker-apps/storage/paperless-ngx) | Used & maintained | Used through an interim LXC deployment for evaluation. |
| [Restic](docker-apps/storage/restic) | Maintained only | Not currently used, but retained for a possible future backup workflow. |

Update this table whenever a stack changes maintenance scope. Record deployment
and host changes in the homelab inventory as well.

## CI

All compose files in `docker-apps` are checked in the `compose-lint` workflow.
The workflow runs `docker compose config --quiet` for each `docker-compose.yaml`
on every pull request, every push to `develop`, and manual runs. This validates
and resolves each Compose model, but does not pull images or start containers.
