# Changelog

## v1.0.1 - 2026-10-27

### Security

- Remove usable default Rundeck and MariaDB passwords; require both values from
  Ansible Vault or another approved secret manager.
- Validate required secret inputs before role execution and suppress their task
  output.

### Changed

- Align Galaxy metadata with `tuxbull.rundeck` and the
  `Tuxbull-Open-Source/ansible.rundeck` source repository.
- Replace the obsolete test role name with the local `rundeck` role basename.
- Rewrite role usage documentation and add maintainer instructions for
  validation, release, and Galaxy import gates.

## v1.0.0 - 2026-06-26

Initial verified release of the Rundeck role.

### Added

- Install Rundeck with MariaDB backend and nginx HTTPS reverse proxy on Rocky Linux 9.
- Install Java before the Rundeck RPM transaction so the Rundeck RPM pre-install script can detect a valid Java runtime.
- Install and configure a local Ansible control-node runtime through `joe-speedboat/ansible.installer`.
- Configure the Ansible runtime for the `rundeck` service user with the `ansible` group.
- Ensure the `rundeck` user is a member of the `ansible` group so Rundeck jobs can execute the local Ansible runtime.
- Repair existing Ansible installer markers when they were created for a different install user.
- Manage firewalld HTTPS exposure for the nginx reverse proxy by default.
- Download the MariaDB JDBC driver from Maven Central with retry/timeout handling.

### Changed

- Use idempotent Ansible password hashing for the Rundeck admin user instead of a shell-based `md5sum` command.
- Document the Ansible installer and firewalld role variables in the README.

### Verified

- Fresh Rocky Linux 9.7 lab installation completed successfully.
- Rundeck, MariaDB, nginx, and firewalld were active after deployment.
- HTTPS access returned a redirect to the Rundeck login page.
- Admin login reached the Rundeck home page during validation.
- The updated `ansible.installer` runtime was rebuilt with `INSTALL_USER=rundeck` and `INSTALL_GROUP=ansible`.
- The `rundeck` user executed `ansible --version` successfully.
- The `rundeck` user executed `ansible localhost -i localhost, -c local -m ping` successfully.
- The restarted `rundeckd` Java process included the `ansible` supplementary group.

### Notes

- Idempotency reruns still report two changed tasks from the external `joe-speedboat.mariadb` dependency. Rundeck role tasks were verified separately.
