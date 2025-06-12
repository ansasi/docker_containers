# Nextcloud AIO

Nextcloud AIO is the official installation method provided by Nextcloud team.

## Design

How it works is that it spins up a Nextcloud master container which you need to access and preconfigure so it can later spin up automatically the other Nextcloud containers (like the database, redis, etc).

## Proxy

The documentation regarding Traefik proxy is not good. In the official documentation they don't use labels and instead they create the `config.yml` file that is in `traefik-config/` folder and add it to the Traefik container.

In a [community discussion](https://help.nextcloud.com/t/nextcloud-aio-traefik/226510/4) they suggested to use these labels:

```yaml
- traefik.http.routers.nc-https.entrypoints=https
- traefik.http.routers.nc-https.rule=Host(`nc.${DOMAIN}`)
- traefik.http.routers.nc-https.tls=true
- traefik.http.routers.nc-https.tls.certresolver=cloudflare
- traefik.http.services.nc-https.loadbalancer.server.url=http://nextcloud-aio-apache:11000
- traefik.http.middlewares.nc-redirect.redirectscheme.scheme=https
- traefik.http.middlewares.nc-secure-headers.headers.hostsProxyHeaders=X-Forwarded-Host
- traefik.http.middlewares.nc-secure-headers.headers.referrerPolicy=same-origin
- traefik.http.middlewares.nc-chain.chain.middlewares=nc-redirect,nc-secure-headers
- traefik.http.routers.nc-https.middlewares=nc-chain
```

Although it looks good, at the time of writing this, it doesn't work. The url is not being used and Traefik automatically uses the port (even when it is not specified in the labels).

## Why are we not using the official image?

Currently the Nextcloud community approach with running myself multiple containers is the solution chosen. This is because the official solution has the next issues:

- It is less flexible.
- It uses many env vars that are not clearly documented.
- It does not work smoothly with Traefik.
