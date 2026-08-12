# Anisble role for installing Grafana

example usage in playbook:

```
- name: SIOT server
  hosts: siot
  become: yes
  roles:
    - { role: users, tags: users }
    - { role: ssh-config, tags: ssh-config }
    - role: grafana
      tags: grafana
      vars:
        smtp_host: mail.myhost.com:587
        smtp_user: smtpuser
        smtp_pass: smtppass
        smtp_from_address: info@myhost
        smtp_from_name: Grafana
        domain: mydomain.com
        root_url: https://mydomain.com
```

The default admin user is:

- user: `admin`
- pass: `admin`

You will be prompted to change the admin password on first login.

## Data source provisioning

Data sources can be described in the playbook rather than clicked into the UI.
Each entry in `datasources` is written to Grafana's provisioning directory,
which Grafana reads at start-up:

```
    - role: grafana
      tags: grafana
      vars:
        datasources:
          - name: VictoriaMetrics
            type: prometheus
            url: http://localhost:8428
            is_default: true
```

`type` is any Grafana data source type; VictoriaMetrics uses `prometheus`, since
it serves the Prometheus query API and needs no plugin. `json_data` passes
type-specific settings through as a mapping. A provisioned data source cannot be
edited in the UI, which is what keeps the playbook the description of what
exists.

## Binding to localhost

Set `http_addr: 127.0.0.1` when Grafana is reached only through a reverse proxy
on the same host. The default is empty, which binds every interface.

### Changing a uid

Grafana updates a provisioned data source by uid, so giving a uid to one that
was already provisioned without one, or changing the uid of an existing one,
fails with `Datasource provisioning error: data source not found`. That failure
takes the whole provisioning module down and leaves Grafana restarting, so it is
worth recognising. Retire the old record by name in the same pass, which Grafana
applies before the additions:

```yaml
- role: grafana
  tags: grafana
  vars:
    delete_datasources:
      - name: VictoriaMetrics
    datasources:
      - name: VictoriaMetrics
        uid: victoriametrics
        type: prometheus
        url: http://localhost:8428
```

Once no such record exists, the `delete_datasources` entry has nothing to do and
can come out again.

## Dashboard provisioning

`dashboards_src` names a directory in the playbook repo holding dashboard JSON
files. The role installs them to `dashboards_dir`, removes any the repo no
longer has, and writes a provider to the provisioning directory, so Grafana
imports them on start and reapplies them on each poll:

```yaml
- role: grafana
  tags: grafana
  vars:
    dashboards_src: dashboards/myserver
```

The installed copy is authoritative, so a restart returns a dashboard to what
the repo says. `dashboards_allow_ui_updates` defaults to true, which leaves
panel edits available in the UI for working out what to commit next.
