# TODO — Apps to Add

For the full current-state inventory (what's active, interim, deferred, or
rejected across all hosts), see the
[homelab README's Apps & status table](https://github.com/ansasi/homelab#apps--status) —
this file only tracks apps that are still candidates to add.

Priority legend: 🔴 High · 🟡 Medium · 🟢 Low · ⚫ Resolved (not needed)

Where new apps go (owner decision, 2026-10): the planned apps are onboarded in
the k3s cluster (homelab repository, `k8s/`), not as new stacks here. The apps
running on Docker today stay as they are and move last.

## Security & Authentication

Vaultwarden now has a [Compose stack and guide](docker-apps/security/vaultwarden/README.md).
This resolves the repository addition only: deployment and any move from Proton
Pass remain unconfirmed and are not performed by adding the stack.

- [ ] **Authentik** 🔴 — Identity provider / SSO (LDAP, SAML, OAuth2); pairs well with
  Traefik. Next, in the k3s cluster (owner decision, 2026-10). Pick one of
  Authentik/Authelia, not both. Once it exists, put the Prometheus and Alertmanager
  routes behind it; they are LAN/VPN-only and have no login until then (anyone who
  reaches Alertmanager can silence alerts).
- [ ] **Authelia** 🔴 — Lightweight auth proxy for 2FA in front of any reverse proxy.
  Same as above; alternative to Authentik, not both.
- [ ] **CrowdSec** 🔴 — Collaborative IPS / threat intelligence layer. Next, in the k3s
  cluster (owner decision, 2026-10), although nothing is exposed to the internet yet
  (LAN/VPN access only). A July 2026 Docker draft (Traefik bouncer plugin) exists on the
  `claude/crowdsec-docker-setup-acfk4x` branch.

## Networking

- [ ] **Tailscale** 🟡 — Zero-config WireGuard mesh VPN. Alternative to NetBird; pick
  one, not both.
- [ ] **NetBird** 🟡 — WireGuard-based overlay network / mesh VPN alternative to
  Tailscale. Preferred (open source), but not finalized.

Remote access uses the FritzBox's built-in WireGuard VPN for now. The WireGuard Easy
(wg-easy) stack was removed in October 2026 instead of migrating it to v15 (see git
history); Tailscale or NetBird is the planned replacement for external access.

## Management

- [ ] **Dockge** 🟢 — Lightweight manager for Compose stacks; a nice alternative to
  Portainer. Not used: Portainer covers it today, and an earlier evaluation found it
  not mature enough. Its old Compose stack was removed (see git history); revisit if
  Portainer becomes a burden.

## Monitoring

- [ ] **Logs: Loki + Grafana Alloy** 🟡 — Central log collection, searchable in the existing
  Grafana. Use Alloy as the collector (Promtail is end-of-life). Planned, not started.
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
