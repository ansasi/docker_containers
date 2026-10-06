# TODO — Apps to Add

For the full current-state inventory (what's active, interim, deferred, or
rejected across all hosts), see the
[homelab README's Apps & status table](https://github.com/ansasi/homelab#apps--status) —
this file only tracks apps that are still candidates to add.

Priority legend: 🔴 High · 🟡 Medium · 🟢 Low · ⚫ Resolved (not needed)

## Security & Authentication

Vaultwarden now has a [Compose stack and guide](docker-apps/security/vaultwarden/README.md).
This resolves the repository addition only: deployment and any move from Proton
Pass remain unconfirmed and are not performed by adding the stack.

- [ ] **Authentik** 🟢 — Identity provider / SSO (LDAP, SAML, OAuth2); pairs well with
  Traefik. Not a priority right now; the Kubernetes cluster is a learning lab, so the
  host is not decided yet. Pick one of Authentik/Authelia, not both.
- [ ] **Authelia** 🟢 — Lightweight auth proxy for 2FA in front of any reverse proxy.
  Same as above; alternative to Authentik, not both.
- [ ] **CrowdSec** 🟢 — Collaborative IPS / threat intelligence layer. Low priority
  while nothing is exposed to the internet (LAN/VPN access only); revisit if a
  service is published. A July 2026 draft (Traefik bouncer plugin) exists on the
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

## Development

- [ ] **Gitea** 🔴 — Lightweight self-hosted Git service. Needed because GitHub Actions
  can't reach the homelab, and the homelab doesn't run 24/7 (it's powered on/used on
  demand), so it can't act as an always-on git remote by itself either. Was postponed
  in favor of GitLab, but GitLab is too heavy to run on-demand. Also intended to close
  the gap left by decommissioning Watchtower: deploy the updates that Renovate (already
  active, see `.renovaterc.json5`) opens PRs for, instead of relying on Watchtower's
  blind auto-pull. **Open decision:** how repos stay pushable while the homelab is off —
  e.g. keep GitHub/GitLab.com as the primary remote and mirror to Gitea when the homelab
  is up, vs. self-hosted runners that only register while online. Also want self-hosted
  repos backed up to GitHub or GitLab.

## Communication

- [ ] **Ntfy (self-hosted)** 🟢 — Alerts use the hosted ntfy.sh for now, because a
  self-hosted server would be off whenever the lab is off. Revisit if the lab runs 24/7.

## Search & RSS

- [ ] **SearXNG** 🔴 — Privacy-respecting self-hosted meta search engine. Private search
  backend for the upcoming Hermes bot integration.
- [ ] **FreshRSS** 🔴 — Lightweight self-hosted RSS aggregator. No RSS reader today;
  wanted soon.
