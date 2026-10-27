# Repository instructions

## Scope

This repository contains the reusable `tuxbull.rundeck` Ansible role. Keep it
focused on installing and configuring Rundeck, its MariaDB backend, an optional
local Ansible runtime, and the Nginx HTTPS reverse proxy on supported systems.
Do not commit credentials, private keys, generated certificates, inventories,
lab logs, provider state, migration notes, or internal infrastructure details.

## Supported platform and dependencies

The maintained platform is Rocky Linux 9. The role requires the role dependency
declared in `roles/requirements.yml`; install it before validation:

```bash
ansible-galaxy role install -r roles/requirements.yml
```

The controller needs the `ansible.posix` collection for firewalld tasks. Do not
modify downloaded dependencies in place.

## Configuration and security

`rundeck_admin_pass` and `mariadb_user_password` are required secret inputs.
Supply them from Ansible Vault or an approved secret manager; do not define
usable default passwords, commit secrets, or print them in diagnostics. The
role generates a Rundeck SSH key by default. Set `rundeck_generate_ssh: false`
when key generation is not appropriate for the target.

## Validation

Run from the repository root:

```bash
python3 - <<'PY'
from pathlib import Path
import yaml
files = list(Path('.').rglob('*.yml')) + list(Path('.').rglob('*.yaml'))
for path in files:
    yaml.safe_load(path.read_text())
print(f'parsed {len(files)} YAML files')
PY
ANSIBLE_ROLES_PATH="$(dirname "$PWD")" \
  ansible-playbook -i 'localhost,' -c local tests/install_rundeck.yml --syntax-check
git diff --check
```

Syntax validation does not install or configure Rundeck. Live testing needs an
explicitly approved Rocky Linux 9 test target, a non-secret temporary inventory
outside this repository, verification of Rundeck/MariaDB/Nginx/firewalld and
HTTPS behavior, an idempotency run, and cleanup of every disposable resource.

## Documentation and publication

`README.md` documents role usage and supported configuration. `CHANGELOG.md`
contains change and release history. Keep releases and version history out of
the README.

Publication identities are separate:

- GitHub source: `Tuxbull-Open-Source/ansible.rundeck`
- Galaxy namespace: `tuxbull`
- Galaxy role: `tuxbull.rundeck`

Do not self-merge a pull request. After a maintainer merges a release PR,
verify the exact default-branch commit and changelog entry before creating an
immutable three-component SemVer tag and matching published GitHub Release.
Only then import the reviewed source into Galaxy. Poll the import task, verify
the role's source owner/repository and version record, and install both the
unpinned and version-pinned Galaxy FQN in fresh temporary directories before
reporting publication complete.
