# Homepage

[Homepage](https://gethomepage.dev/) dashboard for the homelab.

## Configuration

The Homepage config files (`settings.yaml`, `services.yaml`, `widgets.yaml`,
`bookmarks.yaml`, `custom.css`, ...) are **not** stored in this repo. They are
managed by Ansible in the homelab repo:

```
homelab/ansible/playbooks/files/homepage/config/
```

On deploy, the Homepage block in `playbooks/docker.yml` syncs those files to `${WORKDIR}/config`
on the host, and this compose file mounts that directory into the container
(`${WORKDIR}/config:/app/config`). Edit the dashboard there, not here.

## Widget credentials

Widgets that log in to a service get their credentials as environment
variables, referenced as `{{HOMEPAGE_VAR_...}}` in `services.yaml`. Ansible sets
them from Proton Pass; how to create the tokens is in the homelab repo
(`docs/homepage.md`).

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
