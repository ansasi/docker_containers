# Restic

Scheduled [restic](https://restic.net/) backups of a host folder, with
[resticker](https://github.com/djmaze/resticker) (`mazzolino/restic`). One
Compose project per backup (`-p <name>`, `RESTIC_INSTANCE`); the variables are
listed at the top of `docker-compose.yaml`. Where each backup runs, what it
holds and its restore drill are in the homelab repository's app guides.

## Restore

From the backup container, with the repository and its password already set:

```bash
docker exec <instance>-backup restic snapshots
docker exec <instance>-backup restic restore latest --target /tmp/restore
```

`/data` is mounted read-only, so restore elsewhere (inside the container, or
on another machine with restic and the same variables) and move the files
into place with the app stopped.

[Official documentation](https://restic.readthedocs.io/en/latest/050_restore.html)
