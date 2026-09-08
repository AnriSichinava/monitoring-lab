# common

Base configuration for every host in the inventory. Runs before any component role.

## What it does

- Sets timezone and locale
- Creates the admin user with sudo + SSH key
- Installs baseline packages (curl, vim, htop, jq, etc.)
- Hardens SSH (disables password auth, disables root login)
- Configures unattended-upgrades for security patches
- Sets NTP for time sync

## Variables

See [`defaults/main.yml`](defaults/main.yml) and `inventories/production/group_vars/all.yml`.

## Requirements

- Ubuntu 22.04 / 24.04
- `ansible.posix` collection (in `requirements.yml`)

## Dependencies

None — this is the base role, everything else depends on it.
