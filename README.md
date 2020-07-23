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
```
