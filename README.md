# Docker Containers

Docker Compose stacks for the homelab. Ansible in the
[homelab repository](https://github.com/ansasi/homelab) deploys them; where each
app runs is in its [README](https://github.com/ansasi/homelab#apps--status), and
plans are in its [ROADMAP.md](https://github.com/ansasi/homelab/blob/master/ROADMAP.md).

## Maintenance status

Not every folder under `docker-apps/` is deployed. This table sets how each stack
is maintained:

- **Used & maintained** — the application is in use, including LXC
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
| [ERPNext](docker-apps/business/erpnext) | Neither | Rejected: too heavy; Twenty covers the need. |
| [Twenty](docker-apps/business/twenty) | Maintained only | Runs in the homelab k3s cluster; kept as a Docker fallback. |
| [Homepage](docker-apps/management/homepage) | Used & maintained | Active on the main Docker host. |
| [Portainer](docker-apps/management/portainer) | Used & maintained | Active on the Docker and Hermes hosts. |
| [ARRs](docker-apps/media/arrs) | Used & maintained | Active on the main Docker host. |
| [Audiobookshelf](docker-apps/media/audiobookshelf) | Maintained only | Runs in the homelab k3s cluster; kept as a Docker fallback. |
| [Calibre-Web](docker-apps/media/calibre-web) | Maintained only | Runs in the homelab k3s cluster; kept as a Docker fallback. |
| [Jellyfin](docker-apps/media/jellyfin) | Used & maintained | Active in a dedicated LXC for direct iGPU access. |
| [Plex](docker-apps/media/plex) | Maintained only | Deferred and currently superseded by Jellyfin. |
| [Grafana](docker-apps/monitoring/grafana) | Used & maintained | Active on the main Docker host; watches every host. |
| [Netdata](docker-apps/monitoring/netdata) | Maintained only | Not used: replaced by node_exporter and Prometheus. |
| [Uptime Kuma](docker-apps/monitoring/uptime-kuma) | Maintained only | Not used: replaced by Gatus in the homelab k3s cluster. |
| [Pi-hole and dnsproxy](docker-apps/networking/pihole) | Used & maintained | Active on the Raspberry Pi Zero DNS host. |
| [Technitium](docker-apps/networking/technitium) | Maintained only | Deferred while Pi-hole remains the DNS service. |
| [Traefik](docker-apps/networking/traefik) | Used & maintained | Active on the Docker, Vault, and Hermes hosts. |
| [UpSnap](docker-apps/networking/upsnap) | Used & maintained | Active on the Raspberry Pi Zero DNS host. |
| [Vaultwarden](docker-apps/security/vaultwarden) | Used & maintained | Active on the dedicated `vault` VM. |
| [Immich](docker-apps/storage/immich) | Used & maintained | Active on the main Docker host. |
| [Nextcloud](docker-apps/storage/nextcloud) | Maintained only | Deferred while Proton Docs and Drive are used. |
| [Nextcloud AIO](docker-apps/storage/nextcloud-aio) | Maintained only | Deferred while Proton Docs and Drive are used. |
| [Paperless-ngx](docker-apps/storage/paperless-ngx) | Maintained only | Runs in the homelab k3s cluster; kept as a Docker fallback. |
| [Restic](docker-apps/storage/restic) | Maintained only | Not used. |

Update this table whenever a stack changes maintenance scope. Record deployment
and host changes in the homelab inventory as well.

## CI

All compose files in `docker-apps` are checked in the `compose-lint` workflow.
The workflow runs `docker compose config --quiet` for each `docker-compose.yaml`
on every pull request, every push to `master`, and manual runs. This validates
and resolves each Compose model, but does not pull images or start containers.
