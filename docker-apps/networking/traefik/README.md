# Traefik

## Introduction

Traefik is a popular reverse proxy that integrates well with Docker to manage and automate SSL certificates, routing, and load balancing.

## Installation Steps

1. Set `DOMAIN` in the Compose `.env` file.
2. Create credentials for the Traefik dashboard.
   1. Install the `apache2-utils` package.
   2. Run `htpasswd -nB <user>` and enter the password when prompted.
   3. Add the complete output to `.env` using single quotes so Compose treats
      the `$` characters in the bcrypt hash literally:

      ```dotenv
      TRAEFIK_DASHBOARD_CREDENTIALS='<user>:$2y$...'
      ```
3. Create a API token in the Cloudflare dashboard.
   1. Go to the Cloudflare dashboard.
   2. Click on the profile icon in the top right corner.
   3. Click on `My Profile`.
   4. Click on `API Tokens`.
   5. Click on `Create Token`.
   6. Select `Edit DNS Zone`.
   7. Select the desired zone.
4. Add the Cloudflare API token to `.env` as `CF_DNS_API_TOKEN`.
5. Run `docker compose config` and then `docker compose up -d` to start Traefik.

*Note: `Nextcloud AIO` configuration is configured in `config.yml` file, as it does not yet allow Traefik labels*

## Usage Examples in other Docker Compose Files

### Portainer

```yaml
---

---
networks:
  frontend:
    external: true

volumes:
  portainer-data:
    driver: local

services:
  portainer:
    image: portainer/portainer-ce
    container_name: portainer
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer-data:/data
    networks:
      - frontend
    labels:
      - "traefik.enable=true"
      - traefik.docker.network=proxy
      - "traefik.http.routers.portainer.entrypoints=https"
      - "traefik.http.routers.portainer.rule=Host(`portainer.${DOMAIN}`)"
      - "traefik.http.routers.portainer.tls=true"
      - "traefik.http.routers.portainer.tls.certresolver=cloudflare"
      - "traefik.http.services.portainer.loadbalancer.server.port=9000"
    restart: unless-stopped
```
