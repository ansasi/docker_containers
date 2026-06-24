# Homepage

[Homepage](https://gethomepage.dev/) dashboard for the homelab.

## Configuration

The Homepage config files (`settings.yaml`, `services.yaml`, `widgets.yaml`,
`bookmarks.yaml`, `custom.css`, ...) are **not** stored in this repo. They are
managed by Ansible in the homelab repo:

```
homelab/ansible/roles/homepage/files/config/
```

On deploy, the `homepage` Ansible role syncs those files to `${WORKDIR}/config`
on the host, and this compose file mounts that directory into the container
(`${WORKDIR}/config:/app/config`). Edit the dashboard there, not here.

## Running manually

`WORKDIR` must point at a directory that already contains a populated `config/`
folder, e.g.:

```sh
WORKDIR=/home/<user>/docker_volumes/management/homepage \
DOMAIN=docker.datia.me \
docker compose up -d
```
