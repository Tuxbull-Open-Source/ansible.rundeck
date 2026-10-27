# Ansible Rundeck role

Install and configure Rundeck with a local MariaDB backend and an Nginx HTTPS
reverse proxy on a single Rocky Linux 9 host. The role can also install a local
Ansible runtime for the `rundeck` service account and expose HTTPS through
firewalld.

## Requirements

- Ansible 2.11 or later.
- Rocky Linux 9 target with network access to the configured Rundeck, MariaDB
  JDBC, and optional plugin sources.
- Role dependency from `roles/requirements.yml`:

  ```bash
  ansible-galaxy role install -r roles/requirements.yml
  ```

- The `ansible.posix` collection when `rundeck_manage_firewalld` is enabled.
- Secret values supplied through Ansible Vault or another approved secret
  manager; do not put them in inventories or version control.

## Role variables

All defaults are in [`defaults/main.yml`](defaults/main.yml). The following
values must be set for every deployment:

| Variable | Purpose |
| --- | --- |
| `rundeck_domain` | Public DNS name used by Rundeck and the Nginx certificate. |
| `rundeck_admin_pass` | Required Rundeck administrator password. Supply securely. |
| `mariadb_user_password` | Required password for the Rundeck MariaDB user. Supply securely. |

Useful controls:

| Variable | Default | Purpose |
| --- | --- | --- |
| `rundeck_protocol` | `https` | Public Rundeck scheme. |
| `rundeck_generate_ssh` | `true` | Generate an SSH key for the Rundeck service account. |
| `rundeck_install_ansible` | `true` | Install the local Ansible runtime for Rundeck jobs. |
| `rundeck_manage_firewalld` | `true` | Start firewalld and expose the HTTPS service. |
| `rundeck_plugins` | `[]` | Plugin JAR URLs to install. |
| `nginx_ssl_cert_days` | `3650` | Validity period for the generated self-signed certificate. |

The role creates a self-signed certificate for the configured domain. Replace
it with a certificate issued by your approved PKI before exposing the service
to users who require trusted TLS.

## Example playbook

Keep real values in an encrypted variable file, not this repository:

```yaml
---
- name: Install Rundeck
  hosts: rundeck
  become: true
  vars_files:
    - vault.yml
  roles:
    - role: tuxbull.rundeck
      vars:
        rundeck_domain: rundeck.example.invalid
        rundeck_admin_pass: "{{ vault_rundeck_admin_pass }}"
        mariadb_user_password: "{{ vault_rundeck_mariadb_password }}"
```

Install the role from Galaxy after it has been published:

```bash
ansible-galaxy role install tuxbull.rundeck
```

For development from this repository, use a local role path named `rundeck`.

## Operations

The role manages `rundeckd`, `mariadb`, and `nginx`. After a deployment,
verify that all expected services are enabled and active, validate Nginx,
confirm HTTPS reaches the Rundeck login endpoint, and run a second convergence
before considering the deployment idempotent.

Changing the administrator or database password updates Rundeck configuration
and restarts Rundeck. Back up application and database data before maintenance
or upgrades. The role does not implement an uninstall, database migration
rollback, automated certificate renewal, or secret rotation workflow.

## Validation

Run YAML parsing, Ansible syntax validation, `git diff --check`, and the
approved live-test workflow described in [`AGENTS.md`](AGENTS.md). Syntax
checks alone do not prove installation, TLS behavior, database connectivity, or
idempotency.

## License

[LGPL-3.0](https://opensource.org/licenses/LGPL-3.0)

Copyright (c) Chris Ruettimann
