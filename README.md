# Docker Containers

Docker Compose stacks for the homelab. Ansible in the
[homelab repository](https://github.com/ansasi/homelab) deploys them; where each
app runs is in its [README](https://github.com/ansasi/homelab#apps--status), and
plans are in its [ROADMAP.md](https://github.com/ansasi/homelab/blob/master/ROADMAP.md).

## Maintenance status

Not every folder under `docker-apps/` is deployed. This table sets how each stack
is maintained:

- **Used & maintained** — the application is in use, and its Compose stack
  should receive priority correctness, security, and upgrade reviews.
- **Maintained only** — the application is not currently used, but the Compose
  stack is kept viable for possible future deployment.
- **Neither** — the application was rejected or decommissioned. Its files are
  retained as reference and are not proactively upgraded or repaired.

| Stack | Repository status |
|---|---|
| [Hermes](docker-apps/ai/hermes) | Used & maintained |
| [Ollama](docker-apps/ai/ollama) | Maintained only |
| [Open WebUI](docker-apps/ai/openwebui) | Maintained only |
| [Paperclip](docker-apps/ai/paperclip) | Used & maintained |
| [Home Assistant](docker-apps/automation/homeassistant) | Maintained only |
| [ERPNext](docker-apps/business/erpnext) | Neither |
| [Twenty](docker-apps/business/twenty) | Maintained only |
| [Portainer](docker-apps/management/portainer) | Used & maintained |
| [ARRs](docker-apps/media/arrs) | Used & maintained |
| [Audiobookshelf](docker-apps/media/audiobookshelf) | Maintained only |
| [Calibre-Web](docker-apps/media/calibre-web) | Maintained only |
| [Jellyfin](docker-apps/media/jellyfin) | Used & maintained |
| [Plex](docker-apps/media/plex) | Maintained only |
| [Grafana](docker-apps/monitoring/grafana) | Used & maintained |
| [Netdata](docker-apps/monitoring/netdata) | Maintained only |
| [Uptime Kuma](docker-apps/monitoring/uptime-kuma) | Maintained only |
| [Pi-hole and dnsproxy](docker-apps/networking/pihole) | Used & maintained |
| [Technitium](docker-apps/networking/technitium) | Maintained only |
| [Traefik](docker-apps/networking/traefik) | Used & maintained |
| [UpSnap](docker-apps/networking/upsnap) | Used & maintained |
| [Vaultwarden](docker-apps/security/vaultwarden) | Used & maintained |
| [Immich](docker-apps/storage/immich) | Used & maintained |
| [Nextcloud](docker-apps/storage/nextcloud) | Maintained only |
| [Nextcloud AIO](docker-apps/storage/nextcloud-aio) | Maintained only |
| [Paperless-ngx](docker-apps/storage/paperless-ngx) | Maintained only |
| [Restic](docker-apps/storage/restic) | Maintained only |

This matrix sets maintenance scope only. The deployment inventory and all
homelab-specific configuration and operating notes belong in the
[homelab repository](https://github.com/ansasi/homelab#apps--status).
Update maintenance scope here when it changes; record what runs and where there.

## CI

All compose files in `docker-apps` are checked in the `compose-lint` workflow.
The workflow runs `docker compose config --quiet` for each `docker-compose.yaml`
on every pull request, every push to `master`, and manual runs. This validates
and resolves each Compose model, but does not pull images or start containers.
