# Monitoring stack

Reusable Compose services: Grafana, Prometheus, Alertmanager and the
Proxmox VE / PBS exporters. The homelab's inventory, targets, alerts,
dashboards, credentials setup and operating instructions live in
[ansasi/homelab: monitoring](https://github.com/ansasi/homelab/blob/master/docs/monitoring.md).

## Configuration

As with Homepage, configuration is managed outside this repository. Ansible
copies `homelab/ansible/playbooks/files/grafana/config/` to `${WORKDIR}/config`
before starting this stack. A manual deployment must supply its own populated
config tree first:

| Host path, relative to `WORKDIR` | Container path |
|---|---|
| `config/prometheus.yml` | `/etc/config/prometheus.yml` |
| `config/rules/` | `/etc/config/rules/` |
| `config/alertmanager.yml` | `/etc/alertmanager/alertmanager.yml` |
| `config/grafana/provisioning/datasources/` | `/etc/grafana/provisioning/datasources/` |
| `config/grafana/provisioning/dashboards/` | `/etc/grafana/provisioning/dashboards/` |
| `config/grafana/dashboards/` | `/etc/grafana/dashboards/` |

These config mounts are read-only. Other Grafana provisioning directories
keep their image defaults. Config validation and rule tests are owned by the
homelab repository alongside those files.

## Environment

| Variables | Purpose |
|---|---|
| `WORKDIR`, `DOMAIN`, `PUID`, `PGID` | Config/data root, reverse-proxy hostname suffix and Grafana user |
| `NTFY_URL` | Topic URL, delivered as the `ntfy_url` Compose secret |
| `PVE_USER`, `PVE_TOKEN_NAME`, `PVE_TOKEN_VALUE` | Read-only Proxmox API credentials |
| `PBS_ENDPOINT` | PBS API URL, e.g. `https://pbs.example.com:8007` |
| `PBS_USERNAME`, `PBS_API_TOKEN_NAME`, `PBS_API_TOKEN` | Read-only PBS API credentials |

Supply these through the deployment environment; do not commit a real `.env`.
The external `proxy` network and reverse proxy must already exist.

Grafana keeps data in `${WORKDIR}/data`; Prometheus and Alertmanager use their
named volumes. Keep the Compose project identity and these data paths when
moving configuration. The homelab guide contains the ordered first-switch
and rollback procedure. This repository does not deploy changes by merging.
