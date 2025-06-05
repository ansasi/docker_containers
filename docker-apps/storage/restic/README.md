# Restic

Backup system.

## How to restore from a backup

1. SSH into the container.
2. List the snapshots:

    ```bash
    restic snapshots
    ```

3. Restore from an specific snapshot ID:

```bash
restic restore 79766175 --target /tmp-for-restore
```

[Official documentation](https://restic.readthedocs.io/en/latest/050_restore.html)