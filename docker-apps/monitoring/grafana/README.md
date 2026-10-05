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
| `config/prometheus.yml` | Scrape targets. Hosts use their Ansible inventory names as `instance`. rpi4 is not scraped here: it pushes (see [rpi4 pushes its metrics](#rpi4-pushes-its-metrics)). |
| `config/rules/homelab.yml` | Alert rules |
| `config/alertmanager.yml` | Routing: `critical` → ntfy priority 5 (sound), `warning` and resolved → priority 2 (silent). ntfy formats the messages with inline templates. |
| `config/grafana/provisioning/` | Grafana data sources (Prometheus, Alertmanager) and the dashboard folder, see [Grafana](#grafana) |
| `config/grafana/dashboards/` | Dashboard JSON files |
| `tests/homelab.test.yml` | Unit tests for the alert rules (`promtool test rules tests/homelab.test.yml`). CI runs them with the config checks ([monitoring-lint.yml](../../../.github/workflows/monitoring-lint.yml)). Kept outside `config/rules/`, which Prometheus loads entirely. |

## Grafana

Data sources and dashboards come from git (Grafana
[provisioning](https://grafana.com/docs/grafana/latest/administration/provisioning/)),
so a new Grafana volume or a rebuilt `docker1` gets them back.

- **Data sources:** `Prometheus` (UID `prometheus`, default) and `Alertmanager`
  (UID `alertmanager`; active alerts and silences under *Alerting*). Read at
  startup only. A data source created by hand with one of these names is
  replaced, because Grafana does not start when a provisioned name already
  exists with another UID.
- **Dashboards:** folder *Homelab*. Grafana re-reads the JSON files every 30
  seconds. They cannot be saved from the UI: export the JSON (*Share → Export*),
  replace the file here and commit. *Save as* a copy outside the folder is fine
  for experiments.

| File | Source | Revision |
|---|---|---|
| `node-exporter-full.json` | [Node Exporter Full](https://grafana.com/grafana/dashboards/1860) (1860) | 45 |
| `proxmox-via-prometheus.json` | [Proxmox via Prometheus](https://grafana.com/grafana/dashboards/10347) (10347), recommended by prometheus-pve-exporter | 5 |
| `smartctl.json` | [SMARTctl Exporter Dashboard](https://grafana.com/grafana/dashboards/22604) (22604) | 3 |
| `cadvisor.json` | [Cadvisor exporter](https://grafana.com/grafana/dashboards/14282) (14282) | 1 |
| `proxmox-backup-server.json` | [pbs-exporter](https://github.com/natrontech/pbs-exporter/tree/main/grafana-dashboard) `grafana-dashboard/pbs-exporter.json` | `5fd0d95` |

To update a grafana.com dashboard, download its latest revision and point it at
the provisioned data source (provisioning does not fill in import variables):

```bash
curl -sL https://grafana.com/api/dashboards/1860/revisions/latest/download \
  | sed 's/${DS_PROMETHEUS}/prometheus/g' > config/grafana/dashboards/node-exporter-full.json
```

The cAdvisor dashboard's info table is partly empty because cAdvisor runs with
`--store_container_labels=false`; the graphs are not affected.

## Alerts

| Group | Alert | Severity |
|---|---|---|
| Hosts | `HostDown` (5 min), `HostMetricsMissing` (rpi4 pushes nothing for 15 min) | critical |
| Hosts | `HostDiskAlmostFull` (> 85%), `HostDiskFull` (> 95%) | warning, critical |
| Hosts | `HostDiskWillFillIn24h` | warning |
| Hosts | `NasShareAlmostFull` (QNAP shares, > 85%) | warning |
| Hosts | `HostMemoryHigh` (> 90% for 15 min), `HostOomKill` | warning |
| Hosts | `HostFilesystemReadOnly` (root filesystem remounted read-only, e.g. a failing SD card) | critical |
| Raspberry Pi | `RaspberryPiHighTemperature` (> 75 °C), `RaspberryPiThrottled` (throttling without under-voltage, i.e. heat) | warning |
| Raspberry Pi | `RaspberryPiUnderVoltage` (not for rpi4, see below) | warning |
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

rpi4 runs on a 5.0 V power supply that cannot hold the Pi 4's minimum voltage
under load. That was accepted (2026-10-04), so it has no under-voltage alert;
`HostFilesystemReadOnly` reports the most likely damage (SD card errors).
Remove `instance!="rpi4"` from `RaspberryPiUnderVoltage` after replacing the
supply with a 5.1 V one.

### rpi4 pushes its metrics

rpi4 (Raspberry Pi 4) is always on, this stack is not. So it runs **vmagent**
(installed by Ansible, homelab repo `ansible/roles/vmagent`): it scrapes the
Pi's own node_exporter every 15 s with the same labels Prometheus would use
(`job="node_exporter"`, `instance="rpi4"`) and pushes the data to
`https://prometheus.<domain>/api/v1/write`. While this stack is off, vmagent
keeps the data on the Pi (up to 500 MB, about a week) and sends it when
Prometheus is back, so the graphs have no gaps.

dns1 (Pi Zero 2 W) is also always on but is **scraped as before**, with gaps
while this stack is off. It is the LAN's only DNS server with little memory,
and keeping its history is not worth running anything extra on it.

- Prometheus runs with `--web.enable-remote-write-receiver`, and
  `out_of_order_time_window: 7d` in `prometheus.yml` so it accepts the late
  samples (without it, Prometheus rejects them as too old).
- Only rpi4 may push: see [Who can push: the allow-list](#who-can-push-the-allow-list).
- `HostMetricsMissing` fires when rpi4 sends nothing for 15 minutes: if it
  dies, its `up` series disappears instead of becoming 0, so `HostDown` alone
  would stay silent.

Alerts still only run while this stack is on: nothing alerts about the Pis
while the lab is off (an external heartbeat is planned, see
`docs/monitoring.md` in the homelab repo). rpi4's history, though, is complete.

#### Who can push: the allow-list

Prometheus has no login: anyone on the LAN can read it, and with the receiver
on, anyone could also write to it. Written data is trusted like scraped data,
so a buggy or compromised device could push made-up values, hiding a real
problem or raising false alerts. So Traefik only accepts writes from rpi4.

| Address | Host |
|---|---|
| `192.168.178.15` | `rpi4` (Pi 4) |

How it works (labels on the `prometheus` service in `docker-compose.yaml`):

- Router `prometheus-write` matches `Host(prometheus.<DOMAIN>) && Path(/api/v1/write)`.
  Traefik gives a longer rule a higher priority, so writes take this router
  instead of `prometheus-secure`.
- Its middleware `prometheus-write-allowlist`
  ([`ipAllowList`](https://doc.traefik.io/traefik/middlewares/http/ipallowlist/))
  passes only the address above. Everyone else gets `403 Forbidden` and the
  request never reaches Prometheus.
- Everything else (UI, queries, Grafana, the alert links) goes through
  `prometheus-secure` as before, with no restriction.

What it does **not** do:

- It checks addresses, not identities. A device that takes this address
  passes, and everything on rpi4 can write, including the Hermes agents.
- It is not a replacement for authentication. Once an SSO proxy is in place
  (Authelia/Authentik, see [TODO.md](../../../TODO.md)), protect Prometheus with
  it and give vmagent credentials.

Keep it in sync:

- rpi4's address must not change. If it does, update `sourcerange` here
  and the homelab inventory (`ansible/inventory/hosts.ini`) together, then
  deploy this stack. Until then rpi4 gets `403` and keeps its data on disk
  (up to about a week).
- To add a host that pushes, add its address to `sourcerange`
  (comma-separated).

Check and troubleshoot:

```bash
# From any other machine: must print 403
curl -s -o /dev/null -w '%{http_code}\n' -X POST https://prometheus.<DOMAIN>/api/v1/write

# On rpi4: successful pushes count as status_code="2XX", refused ones "4XX"
curl -s 127.0.0.1:8429/metrics | grep vmagent_remotewrite_requests_total
# The log names the exact status code of a refused push
journalctl -u vmagent -n 20
```

- rpi4 gets `403` while other machines also get `403`: Traefik does not see
  rpi4's real address (e.g. a Docker or proxy address instead), or the
  address changed. Check the client address in the Traefik logs before
  widening the list.

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
3. *Permissions → Add → User Permission*: path `/`, user `prometheus@pbs`,
   role `Audit`.
4. *Permissions → Add → API Token Permission*: path `/`, token
   `prometheus@pbs!monitoring`, role `Audit`. Both are needed: a PBS token
   never gets more than its user
   ([PBS docs](https://pbs.proxmox.com/docs/user-management.html#api-tokens)).

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
