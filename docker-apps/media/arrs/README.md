# ARR stack

Media automation: downloads, library management and subtitles. Jellyfin is not
part of this stack (its own [Compose template](../jellyfin/docker-compose.yaml)).

## Services and connections

The Compose file is the source of truth for image versions. Active services are
qBittorrent, Seerr, Prowlarr, Radarr/Sonarr and their anime instances, two Bazarr
instances, FlareSolverr, and Gluetun. Radarr/Sonarr 4K are commented templates,
not enabled services. qBitmanage, SABnzbd, Lidarr, Readarr, Unpackerr, and Jellyfin
are not deployed by this stack.

The external Docker network is `proxy`, shared with Traefik. qBittorrent uses
`network_mode: service:gluetun`; its published port and Traefik labels live on
Gluetun. Prowlarr, FlareSolverr and the other apps connect directly to
`proxy`; Prowlarr's published port and Traefik labels live on Prowlarr.
qBittorrent is the only app on the VPN.

| Service | Address from another container on `proxy` | Published host port |
|---|---|---|
| qBittorrent | `http://gluetun:8085` | 8085 |
| Prowlarr | `http://prowlarr:9696` | 9696 |
| FlareSolverr | `http://flaresolverr:8191` | 8191 |
| Radarr | `http://radarr:7878` | 7878 |
| Radarr Anime | `http://radarr-anime:7878` | 7879 |
| Sonarr | `http://sonarr:8989` | 8989 |
| Sonarr Anime | `http://sonarr-anime:8989` | 8990 |
| Bazarr | `http://bazarr:6767` | None; Traefik |
| Bazarr Anime | `http://bazarr-anime:6767` | None; Traefik |
| Seerr | `http://seerr-jellyfin:5055` | None; Traefik |

Use container ports for app connections, including the anime instances.
Prowlarr's server URL in its Radarr/Sonarr application settings is
`http://prowlarr:9696`. Prowlarr reaches FlareSolverr at
`http://flaresolverr:8191` when a tagged indexer needs it.

Prowlarr stays off the VPN, as [TRaSH](https://trash-guides.info/Prowlarr/prowlarr-setup-proxy/)
and the [Servarr VPN guide](https://wiki.servarr.com/vpn) recommend: a shared VPN
exit can be blocked by indexers or flagged as account sharing. If one indexer
really needs a VPN, use a per-indexer tagged proxy in Prowlarr.

FlareSolverr is off the VPN too. Cloudflare blocks the Proton VPN exit
("Cloudflare has blocked this request"), while the same image solves the
challenge from the home address (tested on 2026-10-27 with 1337x). The VPN
hid nothing here anyway: Prowlarr contacts the indexer directly first and
only hands blocked requests to FlareSolverr.

## Storage contract

`NAS_STORAGE_LOCATION` is the host directory containing both `downloads` and
`library` (on the homelab: `/mnt/qnap/media/arrs`). Do not create an empty
replacement tree if the NAS is offline.

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

Each app keeps its config in `${WORKDIR}/<app>/config` (`seerr` for Seerr), on
local storage; Gluetun uses a named volume.

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
`gluetun:8085` with the qBittorrent credentials. Leave Remote Path Mappings empty
because the client reports the same paths the Arrs see. The disabled 4K
templates also use `/data`; give them their own roots and categories before
enabling them.

In qBittorrent, set the default save path to `/data/downloads/torrents`, use
Automatic Torrent Management, and give each category the corresponding full
`/data/downloads/torrents/<category>` save path. Never download into `library`.
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

Prowlarr needs four application connections, with the Radarr/Sonarr URLs from
the table above and their API keys. Full Sync overwrites the synchronised
indexer settings in each app. No media mount is needed.

## Jellyfin

The separate Docker template exposes the following library paths. Preserve
case: its existing TV path is `/media/TV`.

| Jellyfin library | Docker template folders |
|---|---|
| Movies | `/media/movies`, `/media/anime-movies` |
| Shows | `/media/TV`, `/media/anime-tv` |

A Jellyfin outside this template (the homelab's LXC) mounts the same NAS
folders with its own paths; editing this repository does not change them. Do not
add the torrent tree or the whole mixed library root as a library folder.

## Known issues

Compose validation cannot prove NAS availability, ACLs, hardlinks, app settings,
VPN behaviour or subtitle downloads. Quality profiles, custom formats and naming
live in the app databases.

- qBittorrent still uses `latest` (Renovate does not pick its version correctly).
  Choose a version after checking tracker compatibility and the running one.
- Gluetun retains its existing ping-based healthcheck. A successful ping is not
  proof of VPN isolation or successful app traffic; validate live connectivity.
- Publishing 6881 on the host is not Proton VPN port forwarding: the
  provider-forwarded port and qBittorrent's listening port are separate.
- FlareSolverr is retained to preserve existing integrations. TRaSH currently
  [flags it as non-functional](https://trash-guides.info/Prowlarr/prowlarr-setup-flaresolverr/);
  verify actual indexer needs/results rather than assuming bypass works. Test
  it directly from `docker1`:
  `curl -s -X POST localhost:8191/v1 -H 'Content-Type: application/json' -d '{"cmd":"request.get","url":"https://1337x.to/","maxTimeout":60000}' | jq -r .message`
  should print `Challenge solved!`.

Upstream image references: [qBittorrent](https://docs.linuxserver.io/images/docker-qbittorrent/),
[Radarr](https://docs.linuxserver.io/images/docker-radarr/),
[Sonarr](https://docs.linuxserver.io/images/docker-sonarr/),
[Bazarr](https://docs.linuxserver.io/images/docker-bazarr/), and
[Jellyfin](https://docs.linuxserver.io/images/docker-jellyfin/).
Gluetun's [shared-network instructions](https://github.com/qdm12/gluetun-wiki/blob/main/setup/connect-a-container-to-gluetun.md)
explain why qBittorrent is reached through Gluetun.
