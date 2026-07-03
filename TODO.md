# TODO — Apps to Add

For the full current-state inventory (what's active, interim, deferred, or
rejected across all hosts), see the
[homelab README's Apps & status table](https://github.com/ansasi/homelab#apps--status) —
this file only tracks apps that are still candidates to add.

Priority legend: 🔴 High · 🟡 Medium · 🟢 Low · ⚫ Resolved (not needed)

## Security & Authentication

- [ ] **Vaultwarden** ⚫ — Bitwarden-compatible self-hosted password manager. Not needed —
  using Proton Pass for secrets/vaults instead, which already isolates homelab
  credentials without adding infra to maintain.
- [ ] **Authentik** 🟢 — Identity provider / SSO (LDAP, SAML, OAuth2); pairs well with
  Traefik. Not a priority right now; likely to be built on the Kubernetes cluster once
  it matures, not the Docker host. Pick one of Authentik/Authelia, not both.
- [ ] **Authelia** 🟢 — Lightweight auth proxy for 2FA in front of any reverse proxy.
  Same as above; alternative to Authentik, not both.

## Networking

- [ ] **Tailscale** 🟡 — Zero-config WireGuard mesh VPN. Alternative to NetBird; pick
  one, not both.
- [ ] **NetBird** 🟡 — WireGuard-based overlay network / mesh VPN alternative to
  Tailscale. Preferred (open source), but not finalized.

## Monitoring

- [ ] **Beszel** 🟡 — Lightweight hub-and-agent server + Docker monitoring (low resource
  usage). Want to learn more before deciding vs. Grafana/Netdata.
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

- [ ] **Ntfy** 🟡 — Simple pub/sub push notification server for scripts and alerts.
  Evaluate alongside the observability apps above (Beszel, Speedtest Tracker).

## Search & RSS

- [ ] **SearXNG** 🔴 — Privacy-respecting self-hosted meta search engine. Private search
  backend for the upcoming Hermes bot integration.
- [ ] **FreshRSS** 🔴 — Lightweight self-hosted RSS aggregator. No RSS reader today;
  wanted soon.
