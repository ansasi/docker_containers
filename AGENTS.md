# Repository working agreement

These instructions apply to the entire repository. Read any more-specific
AGENTS.md in the affected directory before editing.

## Scope and sources of truth

- Read [README.md](README.md), [TODO.md](TODO.md), the affected stack's guide,
  [.renovaterc.json5](.renovaterc.json5), and relevant workflows before proposing changes.
- Use the README maintenance matrix for repository scope and the linked homelab
  inventory for deployment locations. Do not assume that a Compose folder is
  deployed, or that an application running in LXC uses this Compose file.
- Treat used applications as production, even when the homelab runs on demand.
- Maintain deferred stacks only within the requested scope. Do not proactively
  upgrade or repair stacks marked Neither.
- Do not infer that merging an upgrade PR means it has been deployed.

## Agree before changing

- Reviews and audits are read-only: report evidence, risks, and proposed changes,
  then ask for approval before implementation.
- For an approved change, briefly explain the intended edits and impact before
  editing. Stay within that scope; ask before adding unrelated improvements.
- Prefer the smallest necessary diff. Preserve existing comments, conventions,
  ports, mounts, environment contracts, and integrations unless changing them
  is explicitly part of the approved task.
- Do not add optional features or hardening merely because they are available.
- Inspect current develop and existing PRs before starting. Reuse relevant work
  instead of opening duplicate or conflicting upgrade PRs.
- Preserve user changes in a dirty worktree. Never overwrite unrelated edits.

## Official documentation first

- Before changing an application's image or configuration, consult its current
  official documentation, release notes, supported upgrade path, and official
  Compose example for the target version.
- Verify image tags and target-host architecture support; do not assume latest
  means compatible or that tags use semantic versioning.
- Cite the relevant upstream sources in the PR. Tutorials and videos are
  supplementary, not a replacement for official requirements.
- If documentation is unavailable or contradictory, state what is unverified
  and hold any change that cannot be justified safely.

## Production and migration safeguards

- Isolate breaking or potentially disruptive changes in separate draft PRs.
  This includes major upgrades, storage or database migrations, mount changes,
  port removal, network changes, new authentication, and Docker socket proxies.
- Do not combine an application migration with unrelated hardening or tooling.
- Before changing persistent paths, obtain read-only evidence of live mounts
  and data locations. Never guess where existing data is stored.
- Prefer additive, two-phase migrations when supported: retain existing paths
  or endpoints until applications have been reconfigured and verified.
- Document prerequisites, backups and restore verification, deployment order,
  smoke tests, and rollback. Database schema changes may require restoring a
  backup; reverting an image tag alone is not necessarily a safe rollback.
- Preserve DNS availability, reverse-proxy routing, and ARR download/library
  integrations. Do not assume direct host ports or legacy paths are unused.
- Never deploy, restart production services, delete data or volumes, rotate
  credentials, merge PRs, or alter branch protection without explicit authority.

## Testing and evidence

- Test every new or modified Compose file before presenting it as ready.
- Parse changed YAML with duplicate-key detection, preserving legitimate
  application-specific tags in companion files.
- Run `docker compose -f <file> config --quiet` for each affected Compose file,
  using non-secret lint placeholders for interpolation. Review relevant
  warnings and verify networks, mounts, ports, labels, dependencies, and env vars.
- Use [.github/workflows/compose-lint.yml](.github/workflows/compose-lint.yml)
  as the source of truth for repository-wide validation and CI placeholders.
- Where feasible, perform isolated startup/health tests using disposable data
  and networks, never production storage or credentials. Test target hardware
  or architecture when behavior depends on it.
- If Docker or target hardware is unavailable, state that limitation. CI can
  validate the Compose model; it does not prove successful startup, migrations,
  host permissions, storage access, or application behavior.
- Inspect CI on the final PR head. Investigate failures before handoff and
  distinguish pre-existing failures from regressions. Never claim unrun tests passed.
- For documentation-only changes, check links, paths, inventory consistency,
  and diff whitespace; do not start containers unnecessarily.
- For Renovate changes, run the official config validator and test package
  matching and tag/update behavior. Preserve registry-qualified names and the
  safeguards against legacy LinuxServer date tags and database major upgrades.
  Do not modify shared presets in another repository without approval.

## Git and PR workflow

- Use a descriptive branch and target develop unless an explicitly documented
  dependency requires a stacked PR. Do not push directly to develop.
- Keep each PR focused. For stacked PRs, state the prerequisite, merge order,
  and need to retarget/retest after the base PR lands.
- Include purpose, exact scope, upstream references where relevant, runtime
  impact, tests actually performed, limitations, and migration/rollback steps.
- Leave migration PRs draft until their documented prerequisites and live
  checks are verified. Do not equate green configuration CI with readiness.
- At handoff, provide the PR link, concise changes, CI result, remaining risks,
  and any user action required. Do not claim a branch, PR, merge, or test
  succeeded until confirmed.

## Secrets and documentation

- Never commit credentials, real .env files, tokens, or sensitive command output.
  Use clearly non-secret examples; avoid printing rendered configurations
  containing secrets.
- If an apparent secret is found, do not reproduce it. Notify the user and
  recommend revocation/rotation. Removing it from a file does not remove it
  from Git history; history rewriting and external rotation need approval.
- Update the affected documentation alongside approved behavior changes.
  Keep maintenance status and deployment status distinct and consistent.
- Documentation labels do not automatically disable Renovate. Changing bot
  behavior requires a separately scoped configuration change.
