# Watchtower

> **Decommissioned. Not deployed and not maintained** (maintenance status
> *Neither* in the [repository README](../../../README.md#maintenance-status)).
> This folder is kept for reference only.

[Watchtower](https://github.com/containrrr/watchtower) pulled new images and
restarted containers automatically. The homelab replaced that blind auto-pull
with Renovate PRs and reviewed deployments (see [TODO.md](../../../TODO.md)).

- **Upstream is archived:** the `containrrr/watchtower` repository became
  read-only on 17 December 2025 and no longer receives updates
  ([announcement](https://github.com/containrrr/watchtower/discussions/1993)).
- **No stack opts in any more:** this compose file uses
  `WATCHTOWER_LABEL_ENABLE=true`, which only updates containers labelled
  `com.centurylinklabs.watchtower.enable=true`. Those labels have been removed
  from every stack, so starting it again would update nothing.
- The Watchtower block in the homelab's `ansible/playbooks/docker.yml` is
  commented out.
