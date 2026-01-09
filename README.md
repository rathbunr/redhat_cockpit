```markdown
# cockpit_wrapper

A lightweight Ansible role that installs and configures the Cockpit Web Console by composing the official Red Hat system role. The wrapper keeps playbooks simple and consistent across environments by exposing a minimal, well‑documented variable surface while delegating heavy lifting to `redhat.rhel_system_roles.cockpit`.

---

## Overview

This role:

- Installs Cockpit using the official system role on supported distributions.
- Optionally opens the Cockpit service port via the firewall role.
- Optionally adds SELinux port policy so Cockpit can listen on a nonstandard port.
- Intentionally does **not** manage certificates or the Cockpit certificate directory; certificate lifecycle is handled separately.
- Is small, DRY, and composable so it can be included in larger automation flows.

---

## Quick start

1. Install the required collection:

```bash
ansible-galaxy collection install redhat.rhel_system_roles
```

2. Add the role to a playbook:

```yaml
- name: Install and configure Cockpit
  hosts: rhel9
  become: true
  roles:
    - role: cockpit_wrapper
      vars:
        cockpit_packages: default
        cockpit_manage_firewall: true
        cockpit_manage_selinux: true
        cockpit_config:
          WebService:
            LoginTitle: "My Company Console"
            MaxStartups: 20
```

3. Run the playbook:

```bash
ansible-playbook -i inventory.ini playbooks/install-cockpit.yml
```

---

## Role variables

| Variable | Default | Description |
|---|---:|---|
| `cockpit_packages` | `default` | Predefined package set: `default`, `minimal`, `full`, or a list of packages. |
| `cockpit_enabled` | `true` | Enable Cockpit at boot. |
| `cockpit_started` | `true` | Ensure Cockpit service is running. |
| `cockpit_config` | `{}` | Dict written to `/etc/cockpit/cockpit.conf`. |
| `cockpit_port` | `9090` | TCP port Cockpit listens on. |
| `cockpit_manage_firewall` | `true` | Open Cockpit port via firewall role on RedHat systems. |
| `cockpit_manage_selinux` | `true` | Add SELinux port policy for the configured port on RedHat systems. |
| `use_system_role` | `true` | Whether to include `redhat.rhel_system_roles.cockpit`. |
| `cockpit_transactional_update_reboot_ok` | `true` | Allow reboot for transactional updates when required. |

---

## SELinux and Firewall

- **Firewall**: When `cockpit_manage_firewall` is enabled and the host is RedHat family, the role includes the official firewall system role to enable the `cockpit` service. The role **does not** remove ports; use the firewall role directly for removals.
- **SELinux**: When `cockpit_manage_selinux` is enabled and the host is RedHat family, the role adds a port label for the configured `cockpit_port` so Cockpit can bind to nonstandard ports. The role only **adds** policy; removal must be handled separately.

---

## Certificate integration

- **Out of scope**: This wrapper intentionally does not create or manage `/etc/cockpit/ws-certs.d`.
- **How to provide certificates**: Use a dedicated certificate role to generate or install certs and then set `cockpit_cert` and `cockpit_private_key` in your play or call the official cockpit role directly with those variables. Example:

```yaml
- include_role:
    name: redhat.rhel_system_roles.cockpit
  vars:
    cockpit_cert: /etc/pki/tls/certs/example.crt
    cockpit_private_key: /etc/pki/tls/private/example.key
```

---

## Testing and idempotence

- Run the role twice: the second run should report no changes if nothing has changed.
- Verify:
  - Service: `systemctl status cockpit`
  - Listening: `ss -ltnp | grep :{{ cockpit_port | default(9090) }}`
  - Config: inspect `/etc/cockpit/cockpit.conf` for expected settings
- CI suggestion: include a simple idempotence test that runs the role twice and asserts `changed == 0` on the second run.

---

## Security and best practices

- Use least privilege: run Ansible with a service account that has only the privileges required to install packages and manage services.
- Separate concerns: keep certificate issuance and private key handling in a dedicated role; do not store private keys in plain text in playbooks.
- Use Ansible Vault for secrets used by other roles (certificate credentials, CA service accounts).
- Audit: log and monitor changes to Cockpit configuration and service restarts.

---

## Contributing

- Follow existing repository conventions and keep changes small and focused.
- Add or update idempotence tests for any behavior change.
- Update this README when adding new variables or behavior.
- For pull requests, include a short description, rationale, and verification steps.

---

## Contact and support

- Open an issue in the repository with a clear description and reproduction steps.
- Propose enhancements as small, composable roles rather than expanding the wrapper’s scope.
```
