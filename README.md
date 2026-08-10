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
