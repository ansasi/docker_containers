# Monitoring: Grafana, Prometheus and Alertmanager

The central monitoring stack of the homelab. It runs **only on the main Docker
host (`docker1`)** and watches every other host over the LAN.

| Service | What it does |
|---|---|
| Grafana | Dashboards (`grafana.<DOMAIN>`) |
| Prometheus | Collects metrics and evaluates the alert rules (`prometheus.<DOMAIN>`) |
| Alertmanager | Sends alerts to ntfy.sh; silences for maintenance (`alertmanager.<DOMAIN>`) |
| cAdvisor | Container metrics of `docker1` |
| pve-exporter | Proxmox VE API: guests, storage |
| pbs-exporter | Proxmox Backup Server API: datastore usage, last backup per guest |

`node_exporter` (every host) and `smartctl_exporter` (Proxmox host) are **not**
part of this stack: they are installed by Ansible in the
[homelab repo](https://github.com/ansasi/homelab). Uptime Kuma stays a separate
stack and checks whether services are reachable; this stack checks whether the
infrastructure is healthy.

## Files

| File | Purpose |
|---|---|
| `config/prometheus.yml` | Scrape targets. Hosts use their Ansible inventory names as `instance`. |
| `config/rules/homelab.yml` | Alert rules |
| `config/alertmanager.yml` | Routing: `critical` → ntfy priority 5 (sound), `warning` and resolved → priority 2 (silent). ntfy formats the messages with inline templates. |
| `tests/homelab.test.yml` | Unit tests for the alert rules (`promtool test rules tests/homelab.test.yml`). CI runs them with the config checks ([monitoring-lint.yml](../../../.github/workflows/monitoring-lint.yml)). Kept outside `config/rules/`, which Prometheus loads entirely. |

## Alerts

| Group | Alert | Severity |
|---|---|---|
| Hosts | `HostDown` (5 min) | critical |
| Hosts | `HostDiskAlmostFull` (> 85%), `HostDiskFull` (> 95%) | warning, critical |
| Hosts | `HostDiskWillFillIn24h` | warning |
| Hosts | `NasShareAlmostFull` (QNAP shares, > 85%) | warning |
| Hosts | `HostMemoryHigh` (> 90% for 15 min), `HostOomKill` | warning |
| Raspberry Pi | `RaspberryPiHighTemperature` (> 75 °C), `RaspberryPiThrottled` | warning |
| Proxmox | `ProxmoxGuestDown` (guest with *Start at boot* is stopped) | critical |
| Proxmox | `ProxmoxStorageAlmostFull` (> 85%) | warning |
| Proxmox | `DiskSmartFailure`, `DiskNvmeCriticalWarning` | critical |
| Proxmox | `SmartctlNoDevices` (exporter sees no disks) | warning |
| Backups | `PbsBackupTooOld` (newest snapshot of an existing guest > 7 days) | critical |
| Backups | `PbsDatastoreAlmostFull` (> 85%), `PbsUnreachable` | warning |
| Containers | `ContainerRestartLoop`, `ContainerOomKilled` | warning |
| Monitoring | `ExporterDown`, `PrometheusRuleFailures`, `AlertmanagerNotificationsFailing` | warning |

The lab runs on demand, so every rule waits (`for:`) before firing, and
`PbsBackupTooOld` waits 2 hours so *Repeat missed* backup jobs can run after
boot. When a host is down, Alertmanager mutes its other alerts.

While the lab is off this stack is off too. The always-on Raspberry Pis are
covered by an external heartbeat (healthchecks.io), set up in the homelab repo.

## Setup

### 1. ntfy.sh

1. Install the ntfy app on the phone.
2. Pick a long random topic name, e.g. `openssl rand -hex 16`. Anyone who knows
   the name can read and send messages, so store the full URL
   (`https://ntfy.sh/<topic>`) in Proton Pass and treat it as a secret.
3. Subscribe to the topic in the app.
4. Test it: `curl -d "test from homelab" https://ntfy.sh/<topic>`.

Use the same URL as an ntfy notification in Uptime Kuma.

### 2. Proxmox VE API token (read-only)

On the Proxmox host ([pve-exporter docs](https://github.com/prometheus-pve/prometheus-pve-exporter#proxmox-ve-configuration)):

```bash
pveum user add prometheus@pve --comment "Prometheus pve-exporter"
pveum acl modify / --users prometheus@pve --roles PVEAuditor
pveum user token add prometheus@pve monitoring --privsep 0
```

Store the user (`prometheus@pve`), token name (`monitoring`) and token value in
Proton Pass.

### 3. Proxmox Backup Server API token (read-only)

On PBS, *Configuration → Access Control*:

1. *User Management → Add*: user `prometheus`, realm `pbs`.
2. *API Token → Add*: user `prometheus@pbs`, token name `monitoring`. Copy the
   secret.
3. *Permissions → Add → API Token Permission*: path `/`, token
   `prometheus@pbs!monitoring`, role `Audit`.

Store the user, token name and secret in Proton Pass.

### 4. Environment

| Variable | Example |
|---|---|
| `DOMAIN`, `WORKDIR`, `PUID`, `PGID` | as before |
| `NTFY_URL` | `https://ntfy.sh/<topic>` |
| `PVE_USER`, `PVE_TOKEN_NAME`, `PVE_TOKEN_VALUE` | `prometheus@pve`, `monitoring`, … |
| `PBS_USERNAME`, `PBS_API_TOKEN_NAME`, `PBS_API_TOKEN` | `prometheus@pbs`, `monitoring`, … |

Ansible passes these as task environment from Proton Pass.

### 5. Deployment order

1. Deploy `node_exporter` / `smartctl_exporter` with Ansible (homelab repo).
   Until then those targets show as down and raise `HostDown` / `ExporterDown`.
2. Deploy this stack on `docker1` and check *Status → Targets* in Prometheus.
3. Send a test alert:

   ```bash
   docker exec alertmanager amtool alert add TestAlert severity=warning instance=test \
     --annotation=summary="Test alert" --alertmanager.url=http://localhost:9093
   ```

## Maintenance

To power off a host on purpose while the lab is on, add a silence in the
Alertmanager UI (*New Silence*, matcher `instance="<host>"`).

## Known limitations

- Proxmox and PBS use their self-signed certificates, so both exporters skip TLS
  verification (`PVE_VERIFY_SSL=false`, `PBS_INSECURE=true`).
- pbs-exporter is tested upstream with PBS 3.x.
- Logs (Loki + Grafana Alloy) are not collected yet; see [TODO.md](../../../TODO.md).
