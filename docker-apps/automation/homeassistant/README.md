# Home Assistant

This is the container configuration for Home Assistant.

It uses Home Assistant container and Postgres container for the database.

It is important to create secrets.yaml inside the `config` folder with the following content:

```yaml
recorder_db_url: postgresql://homeassistant:password@db/homeassistant
```

In the case of this repo this file is created in the Ansible playbook.
