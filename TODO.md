# TODO — Apps to Add

For the full current-state inventory (what's active, interim, deferred, or
rejected across all hosts), see the
[homelab README's Apps & status table](https://github.com/ansasi/homelab#apps--status) —
this file only tracks apps that are still candidates to add.

Priority legend: 🔴 High · 🟡 Medium · 🟢 Low · ⚫ Resolved (not needed)

## Security & Authentication

- [ ] **CrowdSec** 🔴 — Definitely important, but time-to-deploy is unknown — treat as a
  research spike before committing to a date.
- [ ] **Authentik** 🟢 — Not a priority right now; likely to be built on the Kubernetes
  cluster once it matures, not the Docker host. Pick one of Authentik/Authelia, not both.
- [ ] **Authelia** 🟢 — Same as above; alternative to Authentik, not both.
- [x] ~~**Vaultwarden**~~ ⚫ — Not needed. Using Proton Pass for secrets/vaults instead,
  which already isolates homelab credentials without adding infra to maintain.

## Networking

- [ ] **NetBird** 🟡 — Preferred (open source), but not finalized against Tailscale.
- [ ] **Tailscale** 🟡 — Alternative to NetBird; pick one, not both.

## Monitoring

- [ ] **Beszel** 🟡 — Want to learn more before deciding vs. Grafana/Netdata.
- [ ] **Speedtest Tracker** 🟡 — Needs evaluation.

## Development

- [ ] **Gitea** 🔴 — Self-hosted Git + CI/CD. Needed because GitHub Actions can't reach
  the homelab, and the homelab doesn't run 24/7 (it's powered on/used on demand), so it
  can't act as an always-on git remote by itself either. Was postponed in favor of
  GitLab, but GitLab is too heavy to run on-demand. Also intended to close the gap left
  by decommissioning Watchtower: deploy the updates that Renovate (already active, see
  `.renovaterc.json5`) opens PRs for, instead of relying on Watchtower's blind auto-pull.
  **Open decision:** how repos stay pushable while the homelab is off — e.g. keep
  GitHub/GitLab.com as the primary remote and mirror to Gitea when the homelab is up, vs.
  self-hosted runners that only register while online. Also want self-hosted repos backed
  up to GitHub or GitLab.

## Communication

- [ ] **Ntfy** 🟡 — Evaluate alongside the observability apps above (Beszel, Speedtest
  Tracker).

## Search & RSS

- [ ] **SearXNG** 🔴 — Private search backend for the upcoming Hermes bot integration.
- [ ] **FreshRSS** 🔴 — No RSS reader today; wanted soon.

## Productivity & Tools

- [ ] **Memos** 🟢 — Needs time investment.
- [ ] **Outline** 🟢 — Needs time investment.
- [ ] **Stirling-PDF** 🟢 — Needs time investment.
- [ ] **IT-Tools** 🟢 — Needs time investment.
