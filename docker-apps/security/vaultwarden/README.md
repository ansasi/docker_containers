# Vaultwarden

Unofficial Bitwarden-compatible password manager. This is a new-install Compose
template, not a deployed service or a migration from Proton Pass. The deployment
host and storage location still need to be chosen. The external
[homelab inventory](https://github.com/ansasi/homelab#apps--status) remains the
deployment source of truth; it was not accessible while preparing this stack.

## Configuration

- Image: `vaultwarden/server:1.37.2`, with the default SQLite backend. The published
  tag includes Linux amd64, arm64, arm/v7, and arm/v6 images. Verify the target
  host before deploying; configuration validation does not test its architecture.
- Storage: `${WORKDIR}/data` maps to `/data`, including the database, attachments,
  Sends, and signing keys. Use a dedicated absolute path on local storage outside
  this checkout, not an NFS/SMB share for the SQLite database.
- URL: `https://vaultwarden.${DOMAIN}` via the existing external `proxy` network.
  No host ports are published. HTTP and WebSockets both use container port 80;
  legacy port 3012 and `WEBSOCKET_ENABLED` are not used.
- Public registration and organization invitations are disabled. The admin panel
  is disabled because no admin token is configured. SMTP, mobile push, and SSO
  are not configured; email-dependent features are unavailable.

## First Deployment

1. Confirm this is a fresh installation. If an existing Vaultwarden instance is
   found, inspect its live mounts and take a verified backup before planning a
   migration; do not point this template at its data as an experiment.
2. Ensure [Traefik](../../networking/traefik/README.md) is running on the same
   Docker host with its `proxy` network, `https` entrypoint, HTTP-to-HTTPS redirect,
   and `cloudflare` certificate resolver. This stack does not configure or restart
   Traefik. Point `vaultwarden.<your-base-domain>` DNS at that proxy and verify
   trusted HTTPS before entering credentials. LAN/VPN access is sufficient;
   public Internet exposure is not required.
3. Create a local `.env` beside `docker-compose.yaml` using these non-secret
   example values, adjusted for your host. `.env` files are ignored by Git.

```dotenv
DOMAIN=example.com
WORKDIR=/srv/vaultwarden
VAULTWARDEN_SIGNUPS_ALLOWED=false
```

4. Create the chosen data directory with permissions allowing the container to
   write. Run the following from this stack directory. Review logs locally;
   they can contain account information.

```bash
docker compose config --quiet
docker compose pull
docker compose up -d
docker compose ps
```

5. For the initial account only, restrict access to trusted LAN/VPN clients and
   temporarily set `VAULTWARDEN_SIGNUPS_ALLOWED=true` in `.env`. Recreate with
   `docker compose up -d`, open the HTTPS URL, and create the account. Immediately
   set it back to `false` and run `docker compose up -d` again. A plain restart
   does not apply changed environment variables. Verify a second, uninvited
   account cannot register before allowing wider access.
6. In Bitwarden clients, select the self-hosted server and enter the same HTTPS
   URL. Enable account two-step login and keep recovery information somewhere
   independent of this server.

The admin panel is intentionally omitted from this minimal stack. If it is later
needed, follow the official [admin panel guide](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page)
and use an Argon2-hashed token, never `DISABLE_ADMIN_TOKEN=true`. Settings saved
there create `/data/config.json`, which overrides corresponding environment
variables, including signup policy.

## Verification

- Check container health and `https://vaultwarden.<your-base-domain>/alive`.
- Log in through the web vault and a Bitwarden client, create a test item, and
  verify synchronization. Test live WebSocket sync between desktop/browser
  clients; mobile push is a separate, unconfigured feature.
- Upload and download a test attachment, then recreate the container and confirm
  the item and attachment persist.
- Verify registration is closed, `/admin` is unavailable, and HTTP redirects to
  trusted HTTPS rather than serving the vault in plaintext.
- Complete an isolated backup restore before storing important credentials.

## Backups And Updates

Before the first real use, arrange encrypted, off-host backups and test restores.
For a consistent full backup, schedule downtime, stop Vaultwarden, back up the
entire `${WORKDIR}/data` directory and the Compose/`.env` configuration, then
start it again. Do not copy only a live `db.sqlite3`: its WAL may contain committed
data. An online alternative is `docker compose exec vaultwarden /vaultwarden backup`
for a SQLite snapshot, but attachments, Sends, signing keys, and any `config.json`
still need a coordinated backup. Follow the upstream backup guide below.

Test restoration to a separate empty data directory and isolated instance, never
over the live vault. Restore the full stopped-instance backup with its matching
WAL if present. For an online SQLite snapshot, restore it as `db.sqlite3` without
a stale WAL. Start with the backed-up image version and verify login, test items,
and attachments. Keep the original data untouched until verification succeeds.

For updates, read the target release notes and client compatibility requirements,
take and verify a backup, update the pinned image, then run `docker compose pull`
and `docker compose up -d` during an approved maintenance window. Repeat the
verification above. Version 1.37.2 is required for Bitwarden clients 2026.8.0+;
future client updates may require newer Vaultwarden releases. Renovate can track
the pinned image under the existing configuration; merging a PR does not deploy it.

If an upgrade fails, stop the instance and preserve the failed data for diagnosis.
Restore the pre-upgrade backup to a clean directory and use its matching image
and configuration. Reverting only the image may not undo database migrations.
To abandon a fresh installation, `docker compose down` removes its container;
retain the data and backups rather than deleting them.

## Sources

Checked on 2026-09-09. Official Vaultwarden documentation takes precedence over
tutorials, particularly for current configuration and WebSocket behavior.

- [Official Docker Compose example](https://github.com/dani-garcia/vaultwarden/wiki/Using-Docker-Compose)
- [Version 1.37.2 release notes](https://github.com/dani-garcia/vaultwarden/releases/tag/1.37.2)
- [Version 1.37.2 configuration reference](https://github.com/dani-garcia/vaultwarden/blob/1.37.2/.env.template)
- [Published image tag and architectures](https://hub.docker.com/v2/repositories/vaultwarden/server/tags/1.37.2)
- [Image selection](https://github.com/dani-garcia/vaultwarden/wiki/Which-container-image-to-use)
- [Registration controls](https://github.com/dani-garcia/vaultwarden/wiki/Disable-registration-of-new-users)
- [WebSocket notifications](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-WebSocket-notifications)
- [Backups and restores](https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault)
- [Updating the image](https://github.com/dani-garcia/vaultwarden/wiki/Updating-the-vaultwarden-image)
- [Jim's Garage Vaultwarden Compose](https://github.com/JamesTurland/JimsGarage/blob/main/Vaultwarden/docker-compose.yaml):
  likely the requested "Tim's Garage" reference. Uses `/data`, the external
  `proxy` network, and Traefik on port 80. This stack replaces its `latest` tag
  and host-specific paths and explicitly sets the HTTPS domain and signup policy.
- [Techno Tim's Traefik 3 guide](https://technotim.com/posts/traefik-3-docker-certificates/)
  and [proxy code](https://github.com/timothystewart6/launchpad/blob/master/docker/traefik/docker.compose.yml):
  proxy references only. No dedicated Vaultwarden example was found in his current
  Launchpad repository or site sitemap. This stack reuses this repository's
  existing proxy and does not copy tutorial-wide TLS verification bypasses.
