# ARR stack

Production stack on the main Docker host. Deployment locations are tracked in the
[homelab inventory](https://github.com/ansasi/homelab#apps--status).
Jellyfin runs separately in an LXC; its [Compose template](../jellyfin/docker-compose.yaml)
is maintained here but does not configure that LXC.

## Services and connections

The Compose file is the source of truth for image versions. Active services are
qBittorrent, Seerr, Prowlarr, Radarr/Sonarr and their anime instances, two Bazarr
instances, FlareSolverr, and Gluetun. Radarr/Sonarr 4K are commented templates,
not enabled services. qBitmanage, SABnzbd, Lidarr, Readarr, Unpackerr, and Jellyfin
are not deployed by this stack.

The external Docker network is `proxy`, shared with Traefik. qBittorrent,
Prowlarr, and FlareSolverr use `network_mode: service:gluetun`; their ports and
qBittorrent/Prowlarr Traefik labels therefore live on Gluetun. The other apps
connect directly to `proxy`.

| Service | Address from another container on `proxy` | Published host port |
|---|---|---|
| qBittorrent | `http://gluetun:8085` | 8085 |
| Prowlarr | `http://gluetun:9696` | 9696 |
| FlareSolverr | `http://gluetun:8191` | 8191 |
| Radarr | `http://radarr:7878` | 7878 |
| Radarr Anime | `http://radarr-anime:7878` | 7879 |
| Sonarr | `http://sonarr:8989` | 8989 |
| Sonarr Anime | `http://sonarr-anime:8989` | 8990 |
| Bazarr | `http://bazarr:6767` | None; Traefik |
| Bazarr Anime | `http://bazarr-anime:6767` | None; Traefik |
| Seerr | `http://seerr-jellyfin:5055` | None; Traefik |

Use container ports for app connections, including the anime instances.
Prowlarr has no independent network endpoint on `proxy`: use `gluetun:9696`
as its callback URL in Prowlarr's application settings. Within the shared VPN
namespace, Prowlarr can reach FlareSolverr at `http://localhost:8191`.
Existing published ports, proxy routes, VPN configuration and image versions
are preserved by the storage cleanup.

## Storage contract

`NAS_STORAGE_LOCATION` is the **existing** host directory containing both
`downloads` and `library`, not either subdirectory. In the migration discussed
for this installation it is `/mnt/qnap/media/arrs`; verify the live bind sources
before deployment. Do not create a replacement empty tree if the NAS is offline.

| Host path relative to `NAS_STORAGE_LOCATION` | Purpose |
|---|---|
| `downloads/torrents/radarr` | Standard movie torrents |
| `downloads/torrents/radarr-anime` | Anime movie torrents |
| `downloads/torrents/sonarr` | Standard TV torrents |
| `downloads/torrents/sonarr-anime` | Anime TV torrents |
| `library/movies` | Standard movie library |
| `library/anime-movies` | Anime movie library |
| `library/tv` | Standard TV library |
| `library/anime-tv` | Anime TV library |

| Container | Host bind source | Container destination |
|---|---|---|
| qBittorrent | `${NAS_STORAGE_LOCATION}/downloads/torrents` | `/data/downloads/torrents` |
| All Radarr/Sonarr instances | `${NAS_STORAGE_LOCATION}` | `/data` |
| Both Bazarr instances | `${NAS_STORAGE_LOCATION}/library` | `/data/library` |
| Prowlarr | No media bind | Config only |

Each app retains its existing `${WORKDIR}/<app>/config` directory (`seerr` for
Seerr); Gluetun retains its named volume. Keep application databases on suitable
local storage. Do not change `WORKDIR` during this cleanup.

This follows [TRaSH's Docker storage guidance](https://trash-guides.info/File-and-Folder-Structure/How-to-set-up/Docker/).
Our names `downloads/torrents` and `library` are intentional: consistency and a
common filesystem matter, not copying the guide's example directory names.
Radarr/Sonarr perform the import and must see source and destination through one
bind. qBittorrent does not need library access; Bazarr needs writable library
access for subtitles. Separate NAS exports, datasets, or nested filesystems can
still prevent hardlinks even under one `/data` bind.

### Permissions

Retain the working numeric `PUID`/`PGID` shared by the LinuxServer media apps and
`UMASK=002`. With suitable ownership/ACLs this allows group-writable directories
and files (normally 775/664). Verify the application identity with
`docker exec radarr id abc`; plain `docker exec radarr id` reports the exec user,
which can be root even though the application runs as `abc`.

On QNAP, inspect the mount type and ACLs before changing permissions. A failed
`chown` may reflect NFS root squashing or SMB ownership semantics. Do not disable
root squashing, erase ACLs, or recursively make media files executable. Effective
read/write access as the configured application user is the relevant check.

## Application settings (not configured by Compose)

| App | Library root | qBittorrent category |
|---|---|---|
| Radarr | `/data/library/movies` | `radarr` |
| Radarr Anime | `/data/library/anime-movies` | `radarr-anime` |
| Sonarr | `/data/library/tv` | `sonarr` |
| Sonarr Anime | `/data/library/anime-tv` | `sonarr-anime` |

Enable hardlinks and Completed Download Handling in all four apps. Use
`gluetun:8085` with qBittorrent credentials. Leave Remote Path Mappings empty
because the client reports the same paths the Arrs see. Review collections,
Seerr default roots, and any import lists for obsolete roots as well.
Disabled 4K templates also use `/data`; choose separate roots/categories and
verify their image versions and integration settings before enabling them.

In qBittorrent, set the default save path to `/data/downloads/torrents`, use
Automatic Torrent Management, and give each category the corresponding full
`/data/downloads/torrents/<category>` save path. Audit **existing torrents** as
well as defaults. Any incomplete, watched, or exported-torrent directory must
remain accessible after the narrower mount. Never download into `library`.
See [TRaSH qBittorrent setup](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/).

For seeding, leave global ratio/time/inactive-time limits disabled and manage
tracker-specific goals through the Starr indexer settings (or Prowlarr sync,
where supported). Verify the values actually received by each Arr and torrent.
Choose Pause/Stop on reaching a goal, not qBittorrent deletion. Enable Arr
completed-download removal only with correct tracker goals and successful
imports; leave the post-import category empty for this workflow. Where both
ratio and time apply, qBittorrent acts on the first limit reached: ensure the
chosen policy meets **all** private-tracker requirements.

Connect `bazarr` to `radarr:7878` and `sonarr:8989`, and `bazarr-anime` to
`radarr-anime:7878` and `sonarr-anime:8989`, using each instance's own API key.
Leave Bazarr Path Mappings empty and verify its paths start with `/data/library`.
This matches the [Bazarr setup guide](https://wiki.bazarr.media/Getting-Started/Setup-Guide/).
Check subtitle writing for one movie and episode per instance.

Prowlarr needs four application connections, using the Radarr/Sonarr URLs in
the table above and their respective API keys. Use Full Sync if Prowlarr is the
source of truth; understand that it overwrites synchronized indexer settings.
Test all four connections and the indexers. No media mount is needed.

## Jellyfin

The separate Docker template exposes the following library paths. Preserve
case: its existing TV path is `/media/TV`.

| Jellyfin library | Docker template folders |
|---|---|
| Movies | `/media/movies`, `/media/anime-movies` |
| Shows | `/media/TV`, `/media/anime-tv` |

For the active LXC, expose the same NAS anime directories through its own mount
configuration and add those **actual LXC paths** to the existing libraries.
Its current TV path may be `/media/tv`; do not rename it just to match Docker.
Editing this repository does not update LXC mounts. Scan only after files and
mounts are verified. Do not add the torrent tree or the whole mixed library root
as a Movies folder.

## Deployment gate and rollback

This is phase two of the migration: removing legacy `/movies`, `/tv`, and
`/downloads` aliases. The user reported the file moves complete and previously
verified standard Radarr hardlinks. That does not verify both Bazarr instances,
all remaining imports, or the current live configuration. Keep the PR draft
until the following checks are recorded.

1. Capture read-only live mount evidence, for example:
   ```bash
   docker inspect --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}' qbittorrent radarr radarr-anime sonarr sonarr-anime bazarr bazarr-anime
   findmnt -T /mnt/qnap/media/arrs/library/anime-movies
   ```
   Compare the bind sources with the existing deployment `.env`; do not publish
   that file or unfiltered container environment output. Confirm the NAS is
   mounted and every required source directory exists.
2. With transitional mounts still present, finish all settings above. Check
   every existing torrent path, all Arr roots/collections, both Bazarr libraries,
   Seerr, and any scripts. No dependency may remain on a removed alias.
3. Back up the deployed Compose, private `.env`, app configs/databases and
   qBittorrent resume state consistently (app backup or stopped-app copy).
   Back up the library according to the existing policy. Verify a restore into
   a separate location; retain the currently running image IDs locally so
   rollback does not depend on a moving tag.
4. Validate using the existing deployment environment:
   ```bash
   docker compose -f docker-compose.yaml config --quiet
   ```
   Required existing variables are `WORKDIR`, `NAS_STORAGE_LOCATION`, `PUID`,
   `PGID`, `DOMAIN`, `WIREGUARD_PRIVATE_KEY`, and `HOME_SUBNET`. `proxy` and the
   host's `/dev/net/tun` must exist. No sample credentials belong in Git.
5. Schedule a brief interruption, pause new grabs and subtitle jobs, and apply
   the reviewed change using the same project name, working directory and
   environment as the deployed stack. Reuse existing images; do not combine
   this cleanup with an image pull/upgrade. Recreate only changed services.
   Do not run `down -v` or delete any config/media directory. Jellyfin's Docker
   template is not a command to deploy a second Jellyfin alongside the LXC.
6. Inspect effective mounts and app health, test download-client connections,
   then perform one download/import per Arr. From the importing container,
   compare the real torrent and library files:
   ```bash
   docker exec radarr stat -c '%d %i %h %n' '/data/downloads/torrents/radarr/<torrent>/<file>' '/data/library/movies/<movie>/<file>'
   ```
   Replace placeholders with actual files; repeat in each other Arr. Matching
   device and inode with link count at least two proves a hardlink. Verify
   subtitle writes in both Bazarr instances, Prowlarr sync, Seerr roots, and
   Jellyfin playback. Resume automation after these checks pass.

If paths disappear or imports fail, pause affected jobs, restore the prior
Compose and recreate the affected containers with the same configs/images.
This restores aliases without moving media back. Keep application paths on
`/data/...`, which the transitional Compose already supports. Restore app
backups only if needed and while those apps are stopped; account for changes
made since backup. No database/image migration is part of this PR.

## Review boundaries and existing follow-ups

The storage model can be checked statically; Compose validation cannot prove
NAS availability, ACLs, actual hardlinks, application settings, VPN behavior,
or subtitle downloads. Quality profiles, custom formats, naming, and quality
sizes live in app databases and need their own TRaSH review.

- qBittorrent still uses `latest` (existing Renovate TODO). Do not pull it as
  part of this storage rollout; choose a version separately after checking
  tracker compatibility and the running version.
- Gluetun retains its existing ping-based healthcheck. A successful ping is not
  proof of VPN isolation or successful app traffic; validate live connectivity.
- Publishing 6881 on the host is not Proton VPN port forwarding. Verify the
  provider-forwarded port and qBittorrent's listening port separately; this PR
  does not change VPN routing or forwarding.
- FlareSolverr is retained to preserve existing integrations. TRaSH currently
  [flags it as non-functional](https://trash-guides.info/Prowlarr/prowlarr-setup-flaresolverr/);
  verify actual indexer needs/results rather than assuming bypass works.

Upstream image references: [qBittorrent](https://docs.linuxserver.io/images/docker-qbittorrent/),
[Radarr](https://docs.linuxserver.io/images/docker-radarr/),
[Sonarr](https://docs.linuxserver.io/images/docker-sonarr/),
[Bazarr](https://docs.linuxserver.io/images/docker-bazarr/), and
[Jellyfin](https://docs.linuxserver.io/images/docker-jellyfin/).
Gluetun's [shared-network instructions](https://github.com/qdm12/gluetun-wiki/blob/main/setup/connect-a-container-to-gluetun.md)
explain why the three VPN-routed services are reached through Gluetun.
