# centos-sshd-hardening

This Ansible role hardens the SSH server on Enterprise Linux 9/10 and Fedora. It writes one drop-in file, `/etc/ssh/sshd_config.d/00-hardening.conf`, and restarts sshd. The role checks the rendered file with `sshd -t` before it lands, so a bad render cannot break the running service.

## License

MIT. See [LICENSE](LICENSE).

## Supported platforms

| Distribution | Versions |
|---|---|
| Rocky Linux | 9, 10 |
| AlmaLinux | 9, 10 |
| CentOS Stream | 9, 10 |
| Fedora | 43, 44, 45 |

Fedora 45 is supported and tested. The role's Galaxy metadata lists only 43 and 44, because the Galaxy metadata schema does not list Fedora 45 yet.

## Requirements

- Ansible 2.15 or newer on the control node.
- One supported distribution on the target.
- No role dependencies. This role installs nothing, so the target needs the `openssh-server` package already present.

## Role variables

`meta/argument_specs.yml` is the authoritative source for this table. The role validates every variable against it before any task runs.

| Variable | Type | Default | Description |
|---|---|---|---|
| `sshd_hardening_disable_root` | bool | `true` | Deny SSH login for the root account (PermitRootLogin no). |
| `sshd_hardening_adjust_idle` | bool | `true` | Apply the client idle timeout settings below. |
| `sshd_hardening_client_alive_interval` | int | `300` | Seconds of idle time before the server sends a keepalive probe. |
| `sshd_hardening_client_alive_count_max` | int | `2` | Failed probes before the server drops the session. |
| `sshd_hardening_allowed_users` | list of str | `[]` | Users allowed to connect over SSH. An empty list allows all users. |
| `sshd_hardening_port` | int | `22` | TCP port the SSH server listens on. Changing this can lock you out. |
| `sshd_hardening_disable_password` | bool | `true` | Accept only key-based authentication. Deploy keys first. |
| `sshd_hardening_disable_x11_forwarding` | bool | `true` | Disable X11 forwarding. |

## Usage

Install from Ansible Galaxy:

```bash
ansible-galaxy install mariuszczyz.centos_sshd_hardening
```

Ansible Galaxy derives the released version from the git tag (for example `v2.0.0`). The role metadata carries no `version` key because the Galaxy metadata schema rejects it.

In playbook.yml:

```yaml
- hosts: servers
  roles:
    - role: mariuszczyz.centos_sshd_hardening
      tags: ['sshd-hardening']
```

## Lockout warnings

- Keep a second SSH session open while you apply changes. If the new configuration blocks your connection, the open session lets you revert it.
- Run against one host first with `--limit hostname`.
- Changing `sshd_hardening_port` moves the service. Point your SSH client and firewall at the new port before you disconnect.
- With `sshd_hardening_disable_password: true`, the role fails when neither root nor an allowed user has an `authorized_keys` file. Deploy SSH keys before you run the role.

Old 1.x releases wrote lines directly into `/etc/ssh/sshd_config`. This version does not remove those lines. The drop-in wins because OpenSSH uses the first value it reads and Enterprise Linux and Fedora include `sshd_config.d/*.conf` before the rest of the file. Remove stale lines from `/etc/ssh/sshd_config` by hand if you want one clean source.

## Testing

Run the Molecule default scenario with the podman driver:

```bash
molecule test
```

The scenario builds a systemd-capable image from each base image, creates privileged containers, converges the role, checks idempotence, and verifies the effective sshd configuration:

- `docker.io/rockylinux/rockylinux:9`
- `docker.io/library/almalinux:9`
- `quay.io/centos/centos:stream9`
- `docker.io/library/fedora:44`, `:45`

EL 10 base images (Rocky 10, AlmaLinux 10, CentOS Stream 10) need a CPU with x86-64-v3 support, so the default scenario omits them. Add them back on newer hardware or CI runners.

## Breaking changes from 1.x

- All variables changed names. Old 1.x playbooks fail argument validation.

| Old variable | New variable |
|---|---|
| `DISABLE_ROOT` | `sshd_hardening_disable_root` |
| `ADJUST_IDLE` | `sshd_hardening_adjust_idle` |
| `ALIVE_INTERVAL` | `sshd_hardening_client_alive_interval` |
| `ALIVE_COUNT_MAX` | `sshd_hardening_client_alive_count_max` |
| `SSH_USERS` | `sshd_hardening_allowed_users` |
| `SSH_PORT` | `sshd_hardening_port` |
| `DISABLE_PASSWORD` | `sshd_hardening_disable_password` |
| `DISABLE_X11_FORWARDING` | `sshd_hardening_disable_x11_forwarding` |

- The role now writes `/etc/ssh/sshd_config.d/00-hardening.conf` instead of editing `/etc/ssh/sshd_config` with lineinfile.
- `SSH_USERS` was one space-separated string. `sshd_hardening_allowed_users` is a list.
- The idle timeout task in 1.x never ran, because `ADJUST_IDLE: 300` failed its `== true` check. `sshd_hardening_adjust_idle` is a real boolean and the settings now apply.
- `ClientAliveCountMax` is now controlled by `sshd_hardening_client_alive_count_max` (default 2, the value 1.x hardcoded). The old `ALIVE_COUNT_MAX` default of 5 was never used.
- The `mariuszczyz.centos-common` dependency is gone. The role now fails early when no authorized key would survive the password-authentication shutdown.
- License changed from BSD to MIT.

## Author Information

Mariusz Czyz
