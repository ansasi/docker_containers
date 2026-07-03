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

3. Add to traefik's `.env` (`docker-apps/networking/traefik`):

   ```dotenv
   CROWDSEC_BOUNCER_API_KEY=<same key from step 1>
   TRAEFIK_LOGS_DIR=<same path as above>
   ```

   The key never lives in a git-tracked file: Traefik's dynamic config
   ([`traefik-config/config/crowdsec.yml`](../../networking/traefik/traefik-config/config/crowdsec.yml))
   reads it from the container environment via Go templating
   (`{{ env "CROWDSEC_BOUNCER_API_KEY" }}`), and CrowdSec auto-registers
   the bouncer from `BOUNCER_KEY_TRAEFIK` on startup.

4. Start CrowdSec:

   ```bash
   docker compose up -d
   ```

5. Restart Traefik so it downloads the plugin and picks up the new static
   config:

   ```bash
   docker compose -f ../../networking/traefik/docker-compose.yaml up -d --force-recreate
   ```

## Verification

```bash
# Bouncer auto-registered via BOUNCER_KEY_TRAEFIK
docker exec crowdsec cscli bouncers list

# Collections installed (crowdsecurity/traefik pulls in
# base-http-scenarios and http-cve) and traefik-logs parser receiving lines
docker exec crowdsec cscli collections list
docker exec crowdsec cscli metrics

# Traefik logs should show "Plugins loaded" with no bouncer errors
docker logs traefik --tail 50
```

To confirm enforcement, add a short-lived manual ban for your own IP and
check you get a 403 from any `Host()` behind Traefik until it expires
(the plugin runs in stream mode, so a fresh decision can take up to its
60s refresh interval to apply):

```bash
docker exec crowdsec cscli decisions add --ip <your-ip> --duration 5m --reason "test"
docker exec crowdsec cscli decisions list
# ... after the test:
docker exec crowdsec cscli decisions delete --ip <your-ip>
```

Note: Traefik does not access-log requests answered by the entrypoint
http→https redirect itself, so detection feeds off the https traffic —
which is where every real request ends up anyway. Enforcement applies on
both entrypoints.

## Notes / possible enhancements

- **AppSec/WAF:** CrowdSec's Traefik plugin can also run requests through
  the AppSec (WAF) engine in real time (`crowdsecurity/appsec-virtual-patching`,
  `crowdsecurity/appsec-generic-rules` collections, an extra listener on
  port 7422, and `crowdsecAppsecEnabled=true` on the middleware). Not
  enabled here to keep the initial setup light — worth revisiting once the
  base bouncer has been running for a while.
- **Central console:** optionally enroll into
  [app.crowdsec.net](https://app.crowdsec.net) for a dashboard and the
  community blocklist: `docker exec crowdsec cscli console enroll <enroll-key>`.
- **Log rotation:** Traefik's access log grows unbounded; consider
  logrotate on the host for `$TRAEFIK_LOGS_DIR/access.log` (CrowdSec's
  file acquisition follows rotation).
