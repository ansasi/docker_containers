# Traefik

## Introduction

Traefik is a popular reverse proxy that integrates well with Docker to manage and automate SSL certificates, routing, and load balancing.

## Installation Steps

1. Update the domain name in the `traefik.yaml` file to the desired domain.
2. Enable authentication for the Traefik dashboard by creating a password using `htpasswd` if wanted.
3. (Optional) Create a password using `htpasswd` for the Traefik dashboard.
   1. Install `apache2-utils` package.
   2. Run `echo $(htpasswd -nb "<user>" "<password>") | sed -e s/\\$/\\$\\$/g` to generate a password hash.
4. Create a API token in the Cloudflare dashboard.
   1. Go to the Cloudflare dashboard.
   2. Click on the profile icon in the top right corner.
   3. Click on `My Profile`.
   4. Click on `API Tokens`.
   5. Click on `Create Token`.
   6. Select `Edit DNS Zone`.
   7. Select the desired zone.
5. Add the Cloudflare API token to the environment variables in the `docker-compose.yaml` file.
6. Run `docker-compose up -d` to start the Traefik container.

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