# TODO — Apps to Add

For the full current-state inventory (what's active, interim, deferred, or
rejected across all hosts), see the
[homelab README's Apps & status table](https://github.com/ansasi/homelab#apps--status) —
this file only tracks apps that are still candidates to add.

Priority legend: 🔴 High · 🟡 Medium · 🟢 Low · ⚫ Resolved (not needed)

Where new apps go (owner decision, 2026-10): the planned apps are onboarded in
the k3s cluster (homelab repository, `k8s/`), not as new stacks here. The apps
running on Docker today stay as they are and move last, after the cluster has its
own monitoring (kube-prometheus-stack, below) and every cluster app that keeps
data has passed a restore drill (owner decision, 2026-10-14; homelab `README.md`,
*Restore test checklist*).

## Security & Authentication

Vaultwarden has a [Compose stack and guide](docker-apps/security/vaultwarden/README.md)
and runs on the dedicated Vault VM (homelab `ansible/playbooks/vault.yml`,
`docs/vaultwarden.md`). Moving the homelab secrets from Proton Pass follows the
migration order in the homelab README roadmap.

Single sign-on is **postponed** (owner decision, 2026-10-15): Authentik stays deployed
with no app connected, and the choice between Authentik and Authelia stays open.

- [ ] **Authentik** 🔴 — Identity provider / SSO (OIDC, SAML, LDAP, forward-auth). Deployed
  in the k3s cluster (homelab `k8s/apps/security/authentik`, docs/authentik.md) but
  unused: the most used in home-ops repositories and the most complete (UI for users and
  apps, users change their own passwords, built-in forward-auth); app settings are made
  in its UI unless written as blueprints. Paperless-ngx was connected and disconnected
  the same day (homelab PRs #88 and #91). When SSO resumes, put the Prometheus and
  Alertmanager routes behind it; they are LAN/VPN-only and have no login until then
  (anyone who reaches Alertmanager can silence alerts).
- [ ] **Authelia** 🔴 — Lighter identity provider (OIDC) and forward-auth. A trial in the
  k3s cluster was prepared and tested but not deployed (homelab PR #90, closed): its
  configuration, users (a SOPS file of password hashes) and apps all in Git. Without
  LLDAP, people cannot change their own password (each change is a PR); with LLDAP
  (users and groups created from Git) they can.
- [x] **Keycloak** ⚫ — Identity provider (OIDC, SAML). Not needed: the comparison is
  Authentik vs Authelia (2026-10-15); heavier, and needs oauth2-proxy for forward-auth.
- [ ] **CrowdSec** 🟢 — Collaborative IPS / threat intelligence layer. Later, only if a
  service is exposed to the internet (owner decision, 2026-10-15: none is planned).
  CrowdSec reads logs (Traefik, SSH), not network traffic, and ignores private LAN
  addresses by default, so with LAN/VPN-only access it adds little, and it would not
  notice a compromised device on the LAN. For that: IoT on the FritzBox guest network,
  Pi-hole's query log, or a network IDS (Suricata) with new hardware. A July 2026
  Docker draft (Traefik bouncer plugin) exists on the
  `claude/crowdsec-docker-setup-acfk4x` branch.

## Networking

- [ ] **Tailscale** 🟡 — Zero-config WireGuard mesh VPN. Alternative to NetBird (the
  planned choice); pick one, not both.
- [ ] **NetBird** 🔴 — WireGuard-based overlay network / mesh VPN alternative to
  Tailscale. Preferred (open source). Planned in the k3s cluster after Audiobookshelf,
  before the apps running on Docker move (owner decision, 2026-10).
- [ ] **Technitium DNS Server** 🟢 — Possible replacement for Pi-hole, later (owner
  decision, 2026-10: not yet). Until the DNS server is decided, no investment in
  Pi-hole high availability (a second instance with keepalived and sync, as Techno Tim
  runs it) or in ExternalDNS (automatic DNS records for the k3s apps).

Remote access uses the FritzBox's built-in WireGuard VPN for now. The WireGuard Easy
(wg-easy) stack was removed in October 2026 instead of migrating it to v15 (see git
history); Tailscale or NetBird is the planned replacement for external access.

## Management

- [ ] **Dockge** 🟢 — Lightweight manager for Compose stacks; a nice alternative to
  Portainer. Not used: Portainer covers it today, and an earlier evaluation found it
  not mature enough. Its old Compose stack was removed (see git history); revisit if
  Portainer becomes a burden.

## Monitoring

- [ ] **kube-prometheus-stack in the k3s cluster** 🔴 — **Next** (owner decision,
  2026-10-15, ahead of CrowdSec). Prometheus, Alertmanager and
  kube-state-metrics in the cluster, with the alert rules next to each app and alerts
  to ntfy.sh like the stack here. **Critical, required before any app running on
  Docker moves to the cluster** (owner decision, 2026-10-12): Paperless's first file
  backup stayed `Pending` and nothing reported it (homelab `docs/incidents.md`). First
  alerts: VolSync out of sync or absent, CloudNativePG backup age and WAL archiving,
  pods not ready for 15 minutes, Flux not Ready; and the Gatus status page's checks
  (`gatus_results_endpoint_success`, homelab `docs/gatus.md`). The Prometheus stack on
  `docker1` keeps watching the hosts (onedr0p and mirceanton run kube-prometheus-stack).
- [ ] **Logs: Loki + Grafana Alloy** 🟡 — Central log collection, searchable in the existing
  Grafana. Use Alloy as the collector (Promtail is end-of-life). Planned, not started.
  Alternative to evaluate when this starts: VictoriaLogs (lighter; onedr0p/home-ops).
- [ ] **Keep rpi4's metrics while the lab is off** 🟢 — Prometheus on `docker1` pulls,
  so the always-on Raspberry Pis have gaps in their graphs while `docker1` is off.
  Tried in October 2026 and reverted: vmagent on rpi4 pushing to Prometheus
  (`--web.enable-remote-write-receiver`, `out_of_order_time_window: 7d`, Traefik
  `ipAllowList` on `/api/v1/write`, a `HostMetricsMissing` alert for a host that
  stops pushing); see PRs #576 and #579 and the homelab vmagent PRs in git history.
  Too many moving parts for the benefit. Learned: the Raspberry Pi firmware turns
  off the memory cgroup on both Pis (no `MemoryMax`, Docker memory limits not
  enforced); leave dns1 (only DNS, 512 MB) out. Revisit together with
  Loki + Grafana Alloy: Alloy can also push metrics with a disk buffer, so one
  agent would cover both.
- [ ] **Beszel** 🟢 — Lightweight hub-and-agent server + Docker monitoring (low resource
  usage). Maybe later as a quick overview; metrics and alerting are consolidated on
  Prometheus + Grafana + Alertmanager.
- [ ] **Speedtest Tracker** 🟡 — Self-hosted internet speed test history dashboard.
  Needs evaluation.

## Productivity & Tools

- [ ] **Memos** 🟢 — Lightweight self-hosted micro-journal / note-taking (Twitter-like
  feed). Needs time investment.
- [ ] **Outline** 🟢 — Knowledge base and team wiki with real-time collaboration. Needs
  time investment.
- [ ] **Stirling-PDF** 🟢 — Swiss-army PDF processor (merge, split, OCR, compress).
  Needs time investment.
- [ ] **IT-Tools** 🟢 — Collection of developer / sysadmin utility tools in the browser.
  Needs time investment.

## Media

- [ ] **Subtitles with Whisper** 🟡 — Generate subtitles for all films and shows with
  Whisper (speech-to-text) on the workstation's NVIDIA RTX 4070 (12 GB), run on demand:
  no homelab host has a suitable GPU (Jellyfin's iGPU is for transcoding). Options to
  evaluate: Bazarr's [Whisper provider](https://wiki.bazarr.media/Additional-Configuration/Whisper-Provider)
  with `whisper-asr-webservice` (faster-whisper, GPU image) on the workstation, so the
  existing Bazarr (`media/arrs`) requests subtitles for what is missing; or Subgen
  (Whisper, triggered by Jellyfin). The workstation needs the NVIDIA Container Toolkit
  and access to the media library on the NAS (see the NAS items in the homelab
  `TODO.md`). Process the existing library once, then only new items. Notes from
  Bazarr's docs: Whisper subtitles get a fixed score (lower Bazarr's minimum score to
  accept them), and Whisper translates only into English.
- [ ] **Recyclarr** 🟢 — Keeps Sonarr's and Radarr's quality profiles in line with the
  TRaSH Guides, from a configuration file (onedr0p/home-ops). Add it when the ARR stack
  moves to the k3s cluster.

## Gaming

- [ ] **Minecraft server** 🟡 — Game server, in the k3s cluster during the Kubernetes
  learning, outside the order of the planned apps (owner request, 2026-10-14; homelab
  `k8s/README.md`, *Learning path*). The first app that is not a website: its game port
  (TCP 25565) needs its own MetalLB address or a Gateway API `TCPRoute` through Traefik.
  The world lives on a volume backed up by VolSync, with a restore drill. To decide when
  it starts: Java or Bedrock edition (Bedrock uses UDP 19132) and who plays. Reachable
  only while the lab is on; players from outside the LAN after NetBird. Candidate image:
  [itzg/minecraft-server](https://docker-minecraft-server.readthedocs.io/).

## Home Automation

- [ ] **Home Assistant** 🟢 — Home automation hub (onedr0p, mirceanton and Techno Tim
  run it). No smart-home devices yet, so not useful in the short term; revisit when
  there are some.

## Development

- [ ] **Gitea** 🟢 — Lightweight self-hosted Git service. **Later, maybe** (owner decision,
  2026-10). Its main purpose was deploying the updates Renovate opens PRs for (the gap
  left by Watchtower), and that does not need a self-hosted Git server: Flux deploys the
  k3s apps from GitHub, the Docker apps move to k3s over time, and Docker deploys stay a
  manual Ansible run until then. GitHub stays the main remote: the cluster rebuilds from
  Git, so its main Git server cannot run inside it, and the lab is off most of the time.
  If it comes back: GitHub-primary repositories as read-only pull mirrors, Gitea as the
  main remote only for new personal repositories (push-mirrored to GitHub as a backup),
  HTTPS first, SSH later. GitLab was ruled out: its baseline is 8 vCPU / 16 GB (the k3s
  workers have 4 vCPU / 8 GB) and pull mirroring needs a paid tier.

## Communication

- [ ] **Ntfy (self-hosted)** 🟢 — Alerts use the hosted ntfy.sh for now, because a
  self-hosted server would be off whenever the lab is off. Revisit if the lab runs 24/7.

## Search & RSS

- [ ] **SearXNG** 🔴 — Privacy-respecting self-hosted meta search engine. Private search
  backend for the upcoming Hermes bot integration.
- [ ] **FreshRSS** 🔴 — Lightweight self-hosted RSS aggregator. No RSS reader today;
  wanted soon.
