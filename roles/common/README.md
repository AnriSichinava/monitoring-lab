# common

Base OS configuration applied to every host in the inventory before any component role runs.

## What it does

| Concern | Details |
|---|---|
| **Baseline packages** | Installs a curated list of CLI tools (curl, vim, htop, jq, python3, ca-certificates, etc.) |
| **Timezone** | Sets system timezone via `systemd-timedated` (default `UTC`, overridden to `Asia/Jerusalem` in group_vars) |
| **Locale** | Generates + activates the system locale (default `en_US.UTF-8`) |
| **NTP** | Configures `systemd-timesyncd` with pool.ntp.org and ensures it's running |
| **SSH hardening** | Drop-in config at `/etc/ssh/sshd_config.d/99-monitoring-lab-hardening.conf` — password auth off, root login off, key-only |
| **Unattended upgrades** | Installs and enables `unattended-upgrades` for security patches; optional daily auto-reboot |

Every task uses fully-qualified module names, `changed_when`/proper modules for idempotency, and handlers for service restarts. Re-runs should report `changed=0`.

## Variables

All variables live in `defaults/main.yml` — override in `inventories/production/group_vars/all.yml` or per-host in `host_vars/`.

| Variable | Default | Description |
|---|---|---|
| `common_timezone` | `UTC` (fallback to `timezone` group var) | System timezone name |
| `common_locale` | `en_US.UTF-8` | System locale to generate + activate |
| `common_ssh_disable_password_auth` | `true` | Force key-only SSH auth |
| `common_ssh_disable_root_login` | `true` | Block direct root SSH |
| `common_ssh_port` | `22` | SSH listen port |
| `common_unattended_upgrades_enabled` | `true` | Install + enable unattended-upgrades timer |
| `common_unattended_upgrades_reboot` | `false` | Allow the auto-upgrade to reboot at 02:00 |
| `common_packages` | curated Ubuntu list | Packages to install on every host |
| `ntp_servers` | `pool.ntp.org` triplet | Upstream NTP servers for timesyncd |

## Tags

Run selectively with `--tags`:

- `packages` — apt install only
- `timezone` / `time` — timezone + NTP
- `locale`
- `ntp`
- `ssh` / `security` — SSH hardening
- `updates` — unattended-upgrades

Example: `ansible-playbook playbooks/site.yml --tags ssh`

## Requirements

- Ubuntu 22.04 or 24.04 (asserted at role start)
- `community.general` collection (from `requirements.yml`)
- `become: true` on the play (calls `sudo` for all system changes)

## Dependencies

None — this is the base role, every other role should list it in `meta/main.yml → dependencies`.
