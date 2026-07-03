# CrowdSec

## Introduction

CrowdSec is a collaborative IPS: it parses Traefik's access log, detects
malicious behaviour (scanning, brute force, known bad IPs from the
community blocklist, ...) and hands out ban decisions. Traefik enforces
those decisions on every request through the
[`crowdsec-bouncer-traefik-plugin`](https://github.com/maxlerebourg/crowdsec-bouncer-traefik-plugin),
applied globally at the entrypoint level, so every service already
routed through Traefik is protected without touching its own
`docker-compose.yaml`.

*Note: this deviates from
[JimsGarage's CrowdSec guide](https://github.com/JamesTurland/JimsGarage/tree/main/Crowdsec),
which runs a separate `fbonalair/traefik-crowdsec-bouncer` container that
Traefik calls via a `forwardAuth` middleware. That project is no longer the
actively maintained path — CrowdSec's own docs now point at the Traefik
plugin above, which runs inside Traefik itself (no extra container, no
extra network hop).*

## Prerequisites

- The [Traefik](../../networking/traefik) stack from this repo, already
  running on the external `proxy` network.

## Installation Steps

1. Generate a random bouncer API key:

   ```bash
   openssl rand -hex 32
   ```

2. Add to this stack's `.env`:

   ```dotenv
   WORKDIR=/path/to/crowdsec/data
   CROWDSEC_BOUNCER_API_KEY=<key from step 1>
   # Must be the same host path as TRAEFIK_LOGS_DIR in traefik's .env
   TRAEFIK_LOGS_DIR=/path/to/traefik/logs
   ```

3. Make sure traefik's `.env` also sets `TRAEFIK_LOGS_DIR` to that same
   path, and that its `docker-compose.yaml`/`traefik-config` changes from
   this same change have been deployed (access log to file, plugin
   enabled, `crowdsec-bouncer@file` wired into both entrypoints).

4. Edit
   [`../../networking/traefik/traefik-config/config/crowdsec.yml`](../../networking/traefik/traefik-config/config/crowdsec.yml)
   and replace the `crowdsecLapiKey` placeholder with the same key from
   step 1.

5. Start CrowdSec:

   ```bash
   docker compose up -d
   ```

6. Restart Traefik so it downloads the plugin and picks up the new static
   config:

   ```bash
   docker compose -f ../../networking/traefik/docker-compose.yaml up -d --force-recreate
   ```

## Verification

```bash
# Bouncer auto-registered via BOUNCER_KEY_TRAEFIK
docker exec crowdsec cscli bouncers list

# Collection installed and traefik-logs parser receiving lines
docker exec crowdsec cscli collections list
docker exec crowdsec cscli metrics

# Traefik logs should show the plugin loaded with no errors
docker logs traefik --tail 50
```

To confirm enforcement, add a short-lived manual ban for your own IP and
check you get a 403 from any `Host()` behind Traefik until it expires:

```bash
docker exec crowdsec cscli decisions add --ip <your-ip> --duration 1m --reason "test"
docker exec crowdsec cscli decisions list
```

## Notes / possible enhancements

- **More coverage:** add `crowdsecurity/http-cve` to `COLLECTIONS` for
  generic CVE-probing detection on top of the base Traefik scenarios.
- **AppSec/WAF:** CrowdSec's Traefik plugin can also run requests through
  the AppSec (WAF) engine in real time (`crowdsecurity/appsec-virtual-patching`,
  `crowdsecurity/appsec-generic-rules` collections, an extra listener on
  port 7422, and `crowdsecAppsecEnabled=true` on the middleware). Not
  enabled here to keep the initial setup light — worth revisiting once the
  base bouncer has been running for a while.
- **Central console:** optionally enroll into
  [app.crowdsec.net](https://app.crowdsec.net) for a dashboard and the
  community blocklist: `docker exec crowdsec cscli console enroll <enroll-key>`.
