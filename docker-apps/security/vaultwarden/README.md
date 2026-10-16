# Vaultwarden

Bitwarden-compatible password manager. On the homelab it runs on its own VM
(`vault`), for the homelab's secrets only; how that VM is built and backed up is
in the homelab repo (`docs/vaultwarden.md`).

## Configuration

- Image: `vaultwarden/server` (pinned, Renovate updates it), with the default
  SQLite backend. The tag is multi-arch (amd64, arm64, arm/v7, arm/v6).
- Storage: `${WORKDIR}/data` → `/data`: the database, attachments, Sends and
  signing keys. Keep it on local storage, not an NFS/SMB share (SQLite).
- URL: `https://vaultwarden.${DOMAIN}` through Traefik on the external `proxy`
  network. No host ports. HTTP and WebSockets both use container port 80.
- Registration and organisation invitations are off, and so is the admin panel
  (no admin token). No SMTP, mobile push or SSO, so email features do not work.

## First deployment

1. [Traefik](../../networking/traefik/README.md) runs on the same host with the
   `proxy` network and the `cloudflare` certificate resolver, and DNS for
   `vaultwarden.<domain>` points to it. Check that HTTPS is trusted before
   entering credentials. LAN or VPN access is enough.
2. Create `.env` next to `docker-compose.yaml` (ignored by Git):

   ```dotenv
   DOMAIN=example.com
   WORKDIR=/srv/vaultwarden
   VAULTWARDEN_SIGNUPS_ALLOWED=false
   ```

3. Create the data directory, writable by the container, then:

   ```bash
   docker compose config --quiet
   docker compose up -d
   ```

4. For the first account only, set `VAULTWARDEN_SIGNUPS_ALLOWED=true`, run
   `docker compose up -d` (a restart does not apply new environment values),
   create the account, then set it back to `false` and run `docker compose up -d`
   again. Check that a second account cannot register.
5. In the Bitwarden clients, choose a self-hosted server with the same URL, and
   turn on two-step login.

If the admin panel is ever needed, follow the
[admin page guide](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page)
with an Argon2-hashed token, never `DISABLE_ADMIN_TOKEN=true`. Settings saved
there go to `/data/config.json`, which overrides the environment, signups
included.

## Checks

- `https://vaultwarden.<domain>/alive` answers, and HTTP redirects to HTTPS.
- Log in on the web vault and a client, create an item and an attachment, and
  see them sync; recreate the container and check they are still there.
- Registration is closed and `/admin` is not available.

## Backups and updates

For a consistent backup, stop Vaultwarden and copy all of `${WORKDIR}/data` plus
the Compose file and `.env`. Do not copy a live `db.sqlite3` on its own: its WAL
can hold committed data. Online, `docker compose exec vaultwarden /vaultwarden backup`
makes a SQLite snapshot, but attachments, Sends, keys and `config.json` still
need copying. Test a restore into an empty directory on a separate instance,
never over the live vault.

Before an update, read the release notes (clients may need a newer
Vaultwarden), take and check a backup, then pull and `docker compose up -d`.
Reverting only the image may not undo database migrations: roll back by
restoring the pre-update backup with its matching image.

## References

- [Docker Compose example](https://github.com/dani-garcia/vaultwarden/wiki/Using-Docker-Compose),
  [configuration reference](https://github.com/dani-garcia/vaultwarden/blob/main/.env.template)
- [Disable registration](https://github.com/dani-garcia/vaultwarden/wiki/Disable-registration-of-new-users),
  [WebSocket notifications](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-WebSocket-notifications)
- [Backing up your vault](https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault),
  [Updating the image](https://github.com/dani-garcia/vaultwarden/wiki/Updating-the-vaultwarden-image)
