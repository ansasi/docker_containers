# Media Server & Download Manager Stack Docker Compose

This repository provides a Docker Compose configuration for a comprehensive media server and download management stack. It includes services for media playback, request management, torrent downloads, indexer management, library management (movies, TV shows, music, books, subtitles), and automated post-processing.

> **Note:** Some services are disabled by default (commented out) such as ErsatzTV, SABnzbd, Radarr 4K, and Sonarr 4K. Uncomment and configure these sections if you wish to enable them.

## Table of Contents

- [Media Server \& Download Manager Stack Docker Compose](#media-server--download-manager-stack-docker-compose)
  - [Table of Contents](#table-of-contents)
  - [Services Overview](#services-overview)
    - [Jellyfin](#jellyfin)
    - [ErsatzTV (Disabled)](#ersatztv-disabled)
    - [Jellyseerr](#jellyseerr)
    - [qBittorrent](#qbittorrent)
    - [qBitmanage](#qbitmanage)
    - [SABnzbd (Disabled)](#sabnzbd-disabled)
    - [Prowlarr](#prowlarr)
    - [Radarr](#radarr)
    - [Radarr 4K (Disabled)](#radarr-4k-disabled)
    - [Sonarr](#sonarr)
    - [Sonarr 4K (Disabled)](#sonarr-4k-disabled)
    - [Bazarr](#bazarr)
    - [Lidarr](#lidarr)
    - [Readarr](#readarr)
    - [Unpackerr](#unpackerr)
  - [Networks](#networks)
  - [Installation](#installation)
    - [Prerequisites](#prerequisites)
    - [Steps](#steps)
  - [Usage](#usage)
  - [Troubleshooting](#troubleshooting)

---

## Services Overview

### Jellyfin

- **Image:** `jellyfin/jellyfin:latest`  
- **Container Name:** `jellyfin`  
- **Description:**  
  Jellyfin is a media library and player used for streaming your media content.
- **Devices:**  
  Provides GPU access for hardware acceleration (transcoding):
  - `/dev/dri/card1` — Full GPU 02 access  
  - `/dev/dri/renderD128` — GPU render-only access (GPU 01)  
  - *Additional device mappings are available (commented out) and can be enabled if needed.*
- **Environment Variables:**
  - `PGID=1000`
  - `PUID=1000`
  - `UMASK=002`
  - `TZ=Etc/UTC`
- **Volumes:**
  - Media Libraries:
    - TV: `/media/disk/exthdd01/extarrs/data/library/tv` → `/media/TV`
    - Movies: `/media/disk/exthdd01/extarrs/data/library/movies` → `/media/movies`
    - Music: `/media/disk/exthdd01/extarrs/data/library/music` → `/media/music`
    - Pictures: `/media/disk/exthdd01/extarrs/data/library/pictures` → `/media/pictures`
    - Books: `/media/disk/exthdd01/extarrs/data/library/books` → `/media/books`
  - Configuration & Cache:
    - `./appdata/jellyfin/config` → `/config`
    - `./appdata/jellyfin/cache` → `/cache`
  - Transcoding:
    - `/dev/shm` → `/transcode`
- **Ports:**  
  - TODO
- **Network:**  
  - Static IP on network `media`: `10.0.30.100`

---

### ErsatzTV (Disabled)

- **Image Options:**  
  - Base (software transcoding): `jasongdove/ersatztv:latest`  
  - Nvidia transcoding: `jasongdove/ersatztv:latest-nvidia`  
  - VAAPI transcoding: `jasongdove/ersatztv:latest-vaapi`
- **Container Name:** `ersatztv`  
- **Description:**  
  ErsatzTV is an IPTV server that lets you configure and stream custom live TV channels using your media library.
- **Ports:**  
  - Example: `8294:8409` (Container port 8409 to host port 8294)
- **Volumes:**  
  - Configuration: `./appdata/ersatztv/config` → `/root/.local/share/ersatztv`
- **Network:**  
  - Static IP on network `media`: `10.0.30.112`
- **Notes:**  
  - Media paths are not needed if you connect ErsatzTV to Jellyfin with an API key.
  - This service is commented out by default. Uncomment and adjust as needed.

---

### Jellyseerr

- **Image:** `fallenbagel/jellyseerr:latest`  
- **Container Name:** `jellyseerr`  
- **Description:**  
  Jellyseerr is a media request management and discovery tool that integrates with your media server.
- **Environment Variables:**
  - `PGID=1000`
  - `PUID=1000`
  - `UMASK=002`
  - `LOG_LEVEL=debug`
  - `TZ=Etc/UTC`
- **Volumes:**  
  - `./appdata/jellyseerr/config` → `/app/config`
- **Ports:**  
  - `8111:5055` (Maps container port 5055 to host port 8111)
- **Network:**  
  - Static IP on network `media`: `10.0.30.101`

---

### qBittorrent

- **Image:** `ghcr.io/hotio/qbittorrent:latest`  
- **Container Name:** `qbittorrent`  
- **Description:**  
  qBittorrent is a torrent download client used to download torrent files.
- **Environment Variables:**
  - `PUID=1000`
  - `PGID=1000`
  - `UMASK=002`
  - `TZ=Etc/UTC`
  - `WEBUI_PORTS=8080/tcp,8080/udp`
- **Volumes:**  
  - Configuration: `./appdata/qbittorrent/config` → `/config`
  - Torrent Data: `./data/torrents` → `/data/torrents`
- **Ports:**  
  - TODO
- **Network:**  
  - Static IP on network `media`: `10.0.30.102`

---

### qBitmanage

- **Image:** `ghcr.io/stuffanthings/qbit_manage:latest`  
- **Container Name:** `qbitmanage`  
- **Description:**  
  qBitmanage automates routine tasks for qBittorrent, such as cross-seeding and cleanup.
- **Environment Variables:**
  - `PUID=1000`
  - `PGID=1000`
  - `UMASK=002`
  - `TZ=Etc/UTC`
  - Additional options (e.g., `QBT_RUN`, `QBT_SCHEDULE`, `QBT_CONFIG`, etc.) to customize behavior
- **Volumes:**  
  - Torrent data: `/media/disk/exthdd01/extarrs/data/torrents` → `/data/torrents`
  - Configuration: `./appdata/qbitmanage/config` → `/config`
  - qBittorrent Backup: `./appdata/qbittorrent/data/BT_backup` → `/torrentdir`
- **Network:**  
  - Static IP on network `media`: `10.0.30.111`

---

### SABnzbd (Disabled)

- **Image:** `ghcr.io/hotio/sabnzbd:latest`  
- **Container Name:** `sabnzbd`  
- **Description:**  
  SABnzbd is a download client for NZB files from Usenet.  
  **Important:** This container is disabled by default due to the need for a premium Usenet provider.
- **Environment Variables:**
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
- **Volumes:**  
  - Time sync: `/etc/localtime:/etc/localtime:ro`
  - Configuration: `./appdata/sabnzbd/config` → `/config`
  - Usenet Data: `/media/disk/exthdd01/extarrs/data/usenet` → `/data/usenet`
- **Ports:**  
  - TODO
- **Network:**  
  - Static IP on network `media`: `10.0.30.103`
- **Notes:**  
  - Uncomment to enable if you are using a Usenet provider.

---

### Prowlarr

- **Image:** `ghcr.io/hotio/prowlarr:latest`  
- **Container Name:** `prowlarr`  
- **Description:**  
  Prowlarr manages indexers for “ARR” apps, helping to integrate search and download services.
- **Environment Variables:**
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
- **Volumes:**  
  - `./appdata/prowlarr/config` → `/config`
- **Ports:**  
  - `8113:9696` (Maps container port 9696 to host port 8113)
- **Network:**  
  - Static IP on network `media`: `10.0.30.104`

---

### Radarr

- **Image:** `ghcr.io/hotio/radarr:latest`  
- **Container Name:** `radarr`  
- **Description:**  
  Radarr is a movie library manager for automating movie downloads and organization.
- **Environment Variables:**
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
- **Volumes:**  
  - Configuration: `./appdata/radarr/config` → `/config`
  - Media Data: `/media/disk/exthdd01/extarrs/data` → `/data`
- **Ports:**  
  - `8114:7878` (Maps container port 7878 to host port 8114)
- **Network:**  
  - Static IP on network `media`: `10.0.30.105`

---

### Radarr 4K (Disabled)

- **Image:** `ghcr.io/hotio/radarr:latest`  
- **Container Name:** `radarr4k`  
- **Description:**  
  A separate instance of Radarr dedicated to managing 4K movie content.
- **Volumes:**  
  - Configuration: `./appdata/radarr4k/config` → `/config`
  - Media Data: `/media/disk/exthdd01/extarrs/data` → `/data`
- **Ports:**  
  - `8344:7878`
- **Network:**  
  - Static IP on network `media`: `10.0.30.115`
- **Notes:**  
  - Uncomment to enable if 4K management is required.

---

### Sonarr

- **Image:** `ghcr.io/hotio/sonarr:latest`  
- **Container Name:** `sonarr`  
- **Description:**  
  Sonarr is a TV show and series library manager that automates downloads of TV content.
- **Environment Variables:**
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
- **Volumes:**  
  - Configuration: `./appdata/sonarr/config` → `/config`
  - Media Data: `/media/disk/exthdd01/extarrs/data` → `/data`
- **Ports:**  
  - `8115:8989` (Maps container port 8989 to host port 8115)
- **Network:**  
  - Static IP on network `media`: `10.0.30.106`
- **Notes:**  
  - For 4K content, use the Sonarr 4K instance.

---

### Sonarr 4K (Disabled)

- **Image:** `ghcr.io/hotio/sonarr:latest`  
- **Container Name:** `sonarr4k`  
- **Description:**  
  A separate instance of Sonarr dedicated to managing 4K TV content.
- **Volumes:**  
  - Configuration: `./appdata/sonarr4k/config` → `/config`
  - Media Data: `/media/disk/exthdd01/extarrs/data` → `/data`
- **Ports:**  
  - `8345:8989`
- **Network:**  
  - Static IP on network `media`: `10.0.30.116`
- **Notes:**  
  - Uncomment to enable if needed.

---

### Bazarr

- **Image:** `ghcr.io/hotio/bazarr:latest`  
- **Container Name:** `bazarr`  
- **Description:**  
  Bazarr manages subtitles for your media libraries by integrating with various media managers.
- **Environment Variables:**
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
- **Volumes:**  
  - Time sync: `/etc/localtime:/etc/localtime:ro`
  - Configuration: `./appdata/bazarr/config` → `/config`
  - Media Data: `/media/disk/exthdd01/extarrs/data` → `/data`
- **Ports:**  
  - `8116:6767` (Maps container port 6767 to host port 8116)
- **Network:**  
  - Static IP on network `media`: `10.0.30.107`

---

### Lidarr

- **Image:** `ghcr.io/hotio/lidarr:latest`  
- **Container Name:** `lidarr`  
- **Description:**  
  Lidarr automates the download and organization of music, acting as a music library manager.
- **Environment Variables:**
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
- **Volumes:**  
  - Time sync: `/etc/localtime:/etc/localtime:ro`
  - Configuration: `./appdata/lidarr/config` → `/config`
  - Media Data: `/media/disk/exthdd01/extarrs/data` → `/data`
- **Ports:**  
  - `8117:8686` (Maps container port 8686 to host port 8117)
- **Network:**  
  - Static IP on network `media`: `10.0.30.108`

---

### Readarr

- **Image:** `ghcr.io/hotio/readarr:latest`  
- **Container Name:** `readarr`  
- **Description:**  
  Readarr manages books and ebooks (EPUBs), automating download and library organization.
- **Environment Variables:**
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
- **Volumes:**  
  - Time sync: `/etc/localtime:/etc/localtime:ro`
  - Configuration: `./appdata/readarr/config` → `/config`
  - Media Data: `/media/disk/exthdd01/extarrs/data` → `/data`
- **Ports:**  
  - `8118:8787` (Maps container port 8787 to host port 8118)
- **Network:**  
  - Static IP on network `media`: `10.0.30.109`

---

### Unpackerr

- **Image:** `ghcr.io/hotio/unpackerr:latest`  
- **Container Name:** `unpackerr`  
- **Description:**  
  Unpackerr automatically extracts downloaded files for Radarr, Sonarr, Lidarr, and Readarr, then deletes the extracted files after import.
- **Volumes:**  
  - Configuration: `./appdata/unpackerr/config` → `/config`
  - Torrent Data: `/media/disk/exthdd01/extarrs/data/torrents` → `/data/torrents`
- **Environment Variables:**
  - `PUID=1000`
  - `PGID=1000`
  - `TZ=Etc/UTC`
  - `UN_LOG_FILE=/data/torrents/unpackerr.log`
  - API URLs & Keys (replace placeholders with your actual API keys):
    - `UN_SONARR_0_URL=http://sonarr:8989/sonarr`
    - `UN_SONARR_0_API_KEY=IMPORT_SONARR_API_KEY_HERE`
    - `UN_RADARR_0_URL=http://radarr:7878/radarr`
    - `UN_RADARR_0_API_KEY=IMPORT_RADARR_API_KEY_HERE`
    - `UN_LIDARR_0_URL=http://lidarr:8686/lidarr`
    - `UN_LIDARR_0_API_KEY=IMPORT_LIDARR_API_KEY_HERE`
    - `UN_READARR_0_URL=http://readarr:8787/readarr`
    - `UN_READARR_0_API_KEY=IMPORT_READARR_API_KEY_HERE`
- **Security Options:**  
  - `no-new-privileges: true`
- **Network:**  
  - Static IP on network `media`: `10.0.30.110`

---

## Networks

All services connect to a custom Docker network named `media` with the following configuration:

- **Driver:** `bridge`
- **Attachable:** `true`
- **Internal:** `false`
- **IPAM Configuration:**
  - **Subnet:** `10.0.30.0/24`  
    *(Change this to your preferred range if needed.)*
  - **Gateway:** `10.0.30.1`

---
## Installation

### Prerequisites

- **Docker Engine:** [Install Docker](https://docs.docker.com/engine/install/)
- **Docker Compose:** [Install Docker Compose](https://docs.docker.com/compose/install/)

### Steps

1. **Clone the Repository:**

  ```bash
  git clone https://your-repository-url.git
  cd your-repository-directory
  ```

2. **Create Required Directories:**

  Make sure these directories exist for persistent storage:

  ```bash
  mkdir -p appdata/jellyfin/config appdata/jellyfin/cache
  mkdir -p appdata/jellyseerr/config
  mkdir -p appdata/qbittorrent/config
  mkdir -p data/torrents
  mkdir -p appdata/qbitmanage/config
  mkdir -p appdata/prowlarr/config
  mkdir -p appdata/radarr/config
  mkdir -p appdata/sonarr/config
  mkdir -p appdata/bazarr/config
  mkdir -p appdata/lidarr/config
  mkdir -p appdata/readarr/config
  mkdir -p appdata/unpackerr/config
  ```

3. **Review & Customize:**

  - **Media Paths:** Update paths (e.g., `/media/disk/exthdd01/extarrs/data/...`) to match your setup.
  - **Device Access:** Modify device mappings under Jellyfin if additional GPU or hardware acceleration is needed.
  - **API Keys:** Replace placeholder API keys in Unpackerr with your actual keys.
  - **Network Subnet:** Adjust the network subnet in the Compose file if necessary.
  - **Enable/Disable Services:** Uncomment any disabled services if you wish to use them.

## Usage

1. **Start the Stack:**

  From the directory containing the `docker-compose.yml` file, run:

  ```bash
  docker-compose up -d
  ```

2. **Access the Services:**

  Replace `your-host` with your server's IP address or domain:

  - Jellyfin: [http://your-host:8110](http://your-host:8110)
  - Jellyseerr: [http://your-host:8111](http://your-host:8111)
  - qBittorrent: [http://your-host:8290](http://your-host:8290)
  - Prowlarr: [http://your-host:8113](http://your-host:8113)
  - Radarr: [http://your-host:8114](http://your-host:8114)
  - Sonarr: [http://your-host:8115](http://your-host:8115)
  - Bazarr: [http://your-host:8116](http://your-host:8116)
  - Lidarr: [http://your-host:8117](http://your-host:8117)
  - Readarr: [http://your-host:8118](http://your-host:8118)

  For any disabled services, enable them and adjust port mappings as required.

3. **View Logs:**

  To view real-time logs for a service, run:

  ```bash
  docker-compose logs -f <service_name>
  ```

  Replace `<service_name>` with the appropriate container name (e.g., `jellyfin`, `qbittorrent`, `unpackerr`).

## Troubleshooting

- **Port Conflicts:**  
  Ensure that the host ports (e.g., 8110, 8290, etc.) are not already in use.
  
- **Persistent Data:**  
  Verify that the required directories exist and have the proper permissions.
  
- **Device Mappings:**  
  Adjust or uncomment device mappings under Jellyfin if GPU acceleration is not functioning as expected.
  
- **Network Configuration:**  
  Confirm that the custom network settings do not conflict with other Docker networks.
  
- **API Keys:**  
  Double-check that all API keys for Unpackerr and other integrations are correctly set.
