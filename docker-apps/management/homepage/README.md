# Homepage

[Homepage](https://gethomepage.dev/) dashboard for the homelab.

The homelab runs Homepage in its k3s cluster
(`homelab/k8s/apps/observability/homepage`), so this stack is not deployed; it
is kept working for a Docker host.

## Configuration

The Homepage config files (`settings.yaml`, `services.yaml`, `widgets.yaml`,
`bookmarks.yaml`, `custom.css`, ...) are **not** stored in this repo. The
homelab's are in:

```
homelab/k8s/apps/observability/homepage/app/config/
```

This compose file mounts `${WORKDIR}/config` into the container
(`${WORKDIR}/config:/app/config`). Those files are written for the cluster:
outside it, set `kubernetes.yaml` to `mode: disabled`, replace the Kubernetes
widget in `widgets.yaml` (for example with a `resources` widget), and list the
Docker host in `docker.yaml` (`socket: /var/run/docker.sock`, mounted by this
file).

## Widget credentials

Widgets that log in to a service get their credentials as environment
variables, referenced as `{{HOMEPAGE_VAR_...}}` in `services.yaml`; set them
in the environment (for example a `.env` file). How to create the tokens is in
the homelab repo (`docs/homepage.md`).

| Variable | Widget |
|---|---|
| `HOMEPAGE_VAR_PIHOLE_KEY` | Pi-hole |
| `HOMEPAGE_VAR_PVE_TOKEN_ID`, `HOMEPAGE_VAR_PVE_TOKEN_SECRET` | Proxmox VE: read-only API token (`PVEAuditor`) |
| `HOMEPAGE_VAR_PBS_TOKEN_ID`, `HOMEPAGE_VAR_PBS_TOKEN_SECRET` | Proxmox Backup Server: read-only API token (`Audit`) |

A variable that is not set leaves its widget showing an error; the rest of the
page works.

## Running manually

`WORKDIR` must point at a directory that already contains a populated `config/`
folder, e.g.:

```sh
WORKDIR=/home/<user>/docker_volumes/management/homepage \
DOMAIN=docker.example.com \
docker compose up -d
```
