# TODO — Apps to Add

Prioritized roadmap for the homelab.

For the full current-state inventory (what's active, interim, deferred, or
rejected across all hosts), see the
[homelab README's Apps & status table](https://github.com/ansasi/homelab#apps--status) —
this file only tracks apps that are still candidates to add.

Status legend: 🔥 Next up · 🧭 Needs research/decision · 🕒 Needs time investment · ⏳ Deferred

## Phase 1 — Next up

- [ ] **Gitea** 🔥 — Self-hosted Git + CI/CD. Needed because GitHub Actions can't reach
  the homelab, and the homelab doesn't run 24/7 (it's powered on/used on demand), so it
  can't act as an always-on git remote by itself either. Was postponed in favor of
  GitLab, but GitLab is too heavy to run on-demand. **Open decision:** how repos stay
  pushable while the homelab is off — e.g. keep GitHub/GitLab.com as the primary remote
  and mirror to Gitea when the homelab is up, vs. self-hosted runners that only register
  while online. Also want self-hosted repos backed up to GitHub or GitLab.
- [ ] **FreshRSS** 🔥 — No RSS reader today; wanted soon.
- [ ] **SearXNG** 🔥 — Private search backend for the upcoming Hermes bot integration.
- [ ] **CrowdSec** 🔥 — Important, but time-to-deploy is unknown — treat as a research
  spike before committing to a date.

## Phase 2 — Needs research or a decision

- [ ] **Beszel** 🧭 — Want to learn more before deciding vs. Grafana/Netdata.
- [ ] **Speedtest Tracker** 🧭
- [ ] **Ntfy** 🧭 — Evaluate alongside the observability apps above.
- [ ] **Tailscale** vs **NetBird** 🧭 — Leaning NetBird (prefer open source), not
  finalized.

## Phase 3 — Needs time investment (not urgent)

- [ ] **Memos**
- [ ] **Outline**
- [ ] **Stirling-PDF**
- [ ] **IT-Tools**

## Deferred

- [ ] **Authentik** / **Authelia** ⏳ — SSO isn't a priority right now; likely to be
  built on the Kubernetes cluster once it matures, not the Docker host.

## Not needed

- ~~**Vaultwarden**~~ — Using Proton Pass for secrets/vaults instead, which already
  isolates homelab credentials without adding infra to maintain.
