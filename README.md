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
| [Twenty](docker-apps/business/twenty) | Maintained only | Runs in the k3s cluster since 2026-10 (homelab `k8s/apps/business/twenty`); this stack is kept viable as a Docker fallback. |
| [Homepage](docker-apps/management/homepage) | Used & maintained | Active on the main Docker host. |
| [Portainer](docker-apps/management/portainer) | Used & maintained | Active on the Docker and Hermes hosts. |
| [ARRs](docker-apps/media/arrs) | Used & maintained | Active on the main Docker host. |
| [Audiobookshelf](docker-apps/media/audiobookshelf) | Used & maintained | Used through an interim LXC deployment; it moves to the k3s cluster next, starting fresh (homelab roadmap). |
| [Calibre-Web](docker-apps/media/calibre-web) | Maintained only | Moved from the main Docker host to the k3s cluster in 2026-10 (homelab `k8s/apps/media/calibre-web`); this stack is kept viable as a Docker fallback. |
| [Jellyfin](docker-apps/media/jellyfin) | Used & maintained | Active in a dedicated LXC for direct iGPU access. |
| [Plex](docker-apps/media/plex) | Maintained only | Deferred and currently superseded by Jellyfin. |
| [Grafana](docker-apps/monitoring/grafana) | Used & maintained | Central monitoring stack (Grafana, Prometheus, Alertmanager, exporters) for the main Docker host. |
| [Netdata](docker-apps/monitoring/netdata) | Maintained only | Deferred: previously ran on the Raspberry Pi 4, then replaced by node_exporter and the central Prometheus. Kept as an all-in-one alternative to Prometheus + Grafana. |
| [Uptime Kuma](docker-apps/monitoring/uptime-kuma) | Maintained only | Not used: replaced by Gatus in the homelab k3s cluster (2026-10-15), whose checks are in Git. Kept as a Docker alternative. |
| [Pi-hole and dnsproxy](docker-apps/networking/pihole) | Used & maintained | Active on the Raspberry Pi Zero DNS host. |
| [Technitium](docker-apps/networking/technitium) | Maintained only | Deferred while Pi-hole remains the DNS service. |
| [Traefik](docker-apps/networking/traefik) | Used & maintained | Active on the Docker, Vault, and Hermes hosts. |
| [UpSnap](docker-apps/networking/upsnap) | Used & maintained | Active on the Raspberry Pi Zero DNS host. |
| [Vaultwarden](docker-apps/security/vaultwarden/README.md) | Used & maintained | Active on the dedicated Vault VM (homelab `ansible/playbooks/vault.yml`) for the homelab's own secrets; personal passwords stay in Proton Pass. |
| [Immich](docker-apps/storage/immich) | Used & maintained | Active on the main Docker host. |
| [Nextcloud](docker-apps/storage/nextcloud) | Maintained only | Deferred while Proton Docs and Drive are used. |
| [Nextcloud AIO](docker-apps/storage/nextcloud-aio) | Maintained only | Deferred while Proton Docs and Drive are used. |
| [Paperless-ngx](docker-apps/storage/paperless-ngx) | Maintained only | Runs in the k3s cluster since 2026-10 (homelab `k8s/apps/documents/paperless`); this stack is kept viable as a Docker fallback. |
| [Restic](docker-apps/storage/restic) | Maintained only | Not currently used, but retained for a possible future backup workflow. |

Update this table whenever a stack changes maintenance scope. Record deployment
and host changes in the homelab inventory as well.

## CI

All compose files in `docker-apps` are checked in the `compose-lint` workflow.
The workflow runs `docker compose config --quiet` for each `docker-compose.yaml`
on every pull request, every push to `master`, and manual runs. This validates
and resolves each Compose model, but does not pull images or start containers.
