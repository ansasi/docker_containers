# Monitoring: Grafana, Prometheus and Alertmanager

The central monitoring stack of the homelab. It runs **only on the main Docker
host (`docker1`)** and watches every other host over the LAN.

| Service | What it does |
|---|---|
| Grafana | Dashboards (`grafana.<DOMAIN>`) |
| Prometheus | Collects metrics and evaluates the alert rules (`prometheus.<DOMAIN>`) |
| Alertmanager | Sends alerts to ntfy.sh; silences for maintenance (`alertmanager.<DOMAIN>`) |
| pve-exporter | Proxmox VE API: guests, storage |
| pbs-exporter | Proxmox Backup Server API: datastore usage, last backup per guest |

`node_exporter` (every host) and `smartctl_exporter` (Proxmox host) are **not**
part of this stack: they are installed by Ansible in the
[homelab repo](https://github.com/ansasi/homelab). Whether services are reachable
is shown by Gatus, a status page in the homelab k3s cluster (homelab
`docs/gatus.md`); this stack checks whether the infrastructure is healthy.

## Files

| File | Purpose |
|---|---|
| `config/prometheus.yml` | Scrape targets. Hosts use their Ansible inventory names as `instance`. |
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
| `homelab-overview.json` | Own, see below | |
| `node-exporter-full.json` | [Node Exporter Full](https://grafana.com/grafana/dashboards/1860) (1860) | 45 |
| `smartctl.json` | [SMARTctl Exporter Dashboard](https://grafana.com/grafana/dashboards/22604) (22604) | 3 |

*Homelab overview* answers "is everything OK?" on one page: firing alerts,
every host (status, CPU, memory, fullest disk, temperature), every Proxmox
guest with the age of its newest backup, and how full the Proxmox storages, the
PBS datastore and the QNAP shares are. Its colours follow the alert thresholds.
Click a host to open it in Node Exporter Full.

The Hosts table and Host temperature graph prefer CPU hwmon sensors
(`k10temp` on the AMD Minisforum, `coretemp` on Intel, `cpu_thermal`), with
thermal zones as a fallback for the Pis. Disk/NVMe hwmon sensors are excluded.
The value is the highest reported CPU sensor, which can include AMD's control
temperature (Tctl). Both collectors are enabled by default in node_exporter;
the host kernel must expose the sensor. VMs normally have no physical sensor.
The QNAP has no hardware scrape target yet; see the
[temperature setup](https://github.com/ansasi/homelab/blob/master/docs/monitoring.md#host-temperatures)
for checks and the SNMP prerequisite.

There are no Proxmox VE or PBS dashboards: their web interfaces show guests,
storage and backups in more detail and with more history than Prometheus keeps
(7 days). pve-exporter and pbs-exporter are here for the alerts and the
overview.

To update a grafana.com dashboard, download its latest revision and point it at
the provisioned data source (provisioning does not fill in import variables):

```bash
curl -sL https://grafana.com/api/dashboards/1860/revisions/latest/download \
  | sed 's/${DS_PROMETHEUS}/prometheus/g' > config/grafana/dashboards/node-exporter-full.json
```

## Alerts

| Group | Alert | Severity |
|---|---|---|
| Hosts | `HostDown` (5 min) | critical |
| Hosts | `HostDiskAlmostFull` (> 85%), `HostDiskFull` (> 95%) | warning, critical |
| Hosts | `HostDiskWillFillIn24h` | warning |
| Hosts | `NasShareAlmostFull` (QNAP shares, > 85%) | warning |
| Hosts | `HostMemoryHigh` (> 90% for 15 min), `HostOomKill` | warning |
| Hosts | `HostFilesystemReadOnly` (root filesystem remounted read-only, e.g. a failing SD card) | critical |
| Raspberry Pi | `RaspberryPiHighTemperature` (> 75 °C), `RaspberryPiThrottled` (throttling without under-voltage, i.e. heat) | warning |
| Raspberry Pi | `RaspberryPiUnderVoltage` (not for rpi4, see below) | warning |
| Proxmox | `ProxmoxGuestDown` (guest with *Start at boot* is stopped) | critical |
| Proxmox | `ProxmoxStorageAlmostFull` (> 85%) | warning |
| Proxmox | `MinisforumCpuHighTemperature` (90–<98 °C for 10 min), `MinisforumCpuCriticalTemperature` (≥98 °C for 2 min) | warning, critical |
| Proxmox | `DiskSmartFailure`, `DiskNvmeCriticalWarning` | critical |
| Proxmox | `SmartctlNoDevices` (exporter sees no disks) | warning |
| Backups | `PbsBackupTooOld` (newest snapshot of an existing guest > 7 days) | critical |
| Backups | `PbsDatastoreAlmostFull` (> 85%), `PbsUnreachable` | warning |
| Monitoring | `ExporterDown`, `PrometheusRuleFailures`, `AlertmanagerNotificationsFailing` | warning |

The lab runs on demand, so every rule waits (`for:`) before firing, and
`PbsBackupTooOld` waits 2 hours so *Repeat missed* backup jobs can run after
boot. When a host is down, Alertmanager mutes its other alerts.

rpi4 runs on a 5.0 V power supply that cannot hold the Pi 4's minimum voltage
under load. That was accepted (2026-10-04), so it has no under-voltage alert;
`HostFilesystemReadOnly` reports the most likely damage (SD card errors).
Remove `instance!="rpi4"` from `RaspberryPiUnderVoltage` after replacing the
supply with a 5.1 V one.

While the lab is off this stack is off too, so nothing watches the always-on
Raspberry Pis during that time.

### Minisforum CPU temperature

`proxmox1` is the MS-A2 with a **Ryzen 9 7945HX**, according to the homelab's
[hardware notes](https://github.com/ansasi/homelab/blob/master/docs/hardware.md).
[AMD specifies 100 °C Tjmax](https://www.amd.com/en/products/processors/laptop/ryzen/7000-series/amd-ryzen-9-7945hx.html).
There is no single idle temperature: ambient temperature, BIOS/fan profile,
power settings and running VMs change it. Heavy CPU workloads can run in the
80s and have brief peaks in the low 90s. For context, Notebookcheck measured
about 84 °C sustained in an MS-A2 with the **9955HX**, not this lab's 7945HX
([review](https://www.notebookcheck.net/Minisforum-MS-A2-review-Compact-AMD-mini-PC-with-workstation-ambitions-and-GPU-upgrade-option.1062179.0.html)).
That is a chassis comparison, not a measured baseline for this host.

The alert levels are a homelab monitoring policy, not manufacturer limits:

| CPU reading | Notification |
|---|---|
| Below 90 °C | No high-temperature alert |
| At least 90 °C, below 98 °C, continuously for 10 min | Warning, silent ntfy |
| At least 98 °C, continuously for 2 min | Critical, ntfy with sound |

The warning band stops at 98 °C so both alerts do not notify for the same
condition. Short boost peaks do not satisfy the delays. Reaching 90 °C is a
reason to inspect load and cooling if sustained; it does not prove damage.
The critical level leaves 2 °C of headroom to AMD's limit. This is monitoring,
not a replacement for the CPU's own thermal protection; no BIOS, fan or power
settings are changed.

Only the highest `k10temp` CPU reading on `proxmox1` is used (including Tctl
and CCD sensors), not NVMe, GPU or ACPI thermal-zone readings. Missing CPU
metrics do not invent a temperature or trigger these alerts; verify
`node_hwmon_chip_names{instance="proxmox1",chip_name="k10temp"}` and
`node_hwmon_temp_celsius{instance="proxmox1"}` in Prometheus after rollout.
Tctl can be a control temperature; see the
[kernel driver documentation](https://docs.kernel.org/hwmon/k10temp.html).

The Host temperature graph uses 90/98 °C colours for `proxmox1` and keeps
70/75 °C for the Pis. Its shared 75 °C guide is removed. The Hosts table
shows temperature without a shared colour threshold, because one table
column cannot use different threshold values for each host. The Firing alerts
panel shows the delayed warning/critical state; graph colours are immediate.
The Raspberry Pi alert rules remain unchanged.

After merge, update the stack checkout on `docker1` and reload Prometheus's
rules through the normal controlled deployment. Grafana reloads the JSON
itself. Check both rules in Prometheus's Rules page and verify the CPU sensor
series before relying on notifications. Revert and reload the rules to undo;
no persistent application data changes.

## Setup

### 1. ntfy.sh

1. Install the ntfy app on the phone.
2. Pick a long random topic name, e.g. `openssl rand -hex 16`. Anyone who knows
   the name can read and send messages, so store the full URL
   (`https://ntfy.sh/<topic>`) in Proton Pass and treat it as a secret.
3. Subscribe to the topic in the app.
4. Test it: `curl -d "test from homelab" https://ntfy.sh/<topic>`.

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
- Logs are not collected (no Loki).
