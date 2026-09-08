# zabbix-agent

Installs and configures Zabbix Agent 2 (the Go-based rewrite) on Ubuntu hosts. Runs on every host in the monitoring stack — both the `mon-server` (self-monitor) and every `mon-target-*`.

## What it does

| Step | Details |
|---|---|
| **Repo** | Adds official Zabbix APT repo (arch-aware — `ubuntu` for x86_64, `ubuntu-arm64` for aarch64) + GPG signing key |
| **Install** | Installs `zabbix-agent2` from the Zabbix repo (not Ubuntu's older bundled version) |
| **Config** | Deploys `/etc/zabbix/zabbix_agent2.conf` from template with Server, ServerActive, Hostname, ListenPort, logging |
| **Include dir** | Ensures `/etc/zabbix/zabbix_agent2.d/` exists (owned by zabbix:zabbix) for user parameters and plugin drops |
| **Service** | Enables + starts `zabbix-agent2` systemd unit; restart handler fires on config change |

Depends on the `common` role (declared in `meta/main.yml`) — Ansible will run `common` first automatically when this role is included.

## Variables

| Variable | Default | Description |
|---|---|---|
| `zabbix_major_version` | `7.0` | Zabbix LTS series to install |
| `zabbix_release_deb_version` | `7.0-2` | Version tag on the Zabbix release .deb |
| `zabbix_server_ip` | first host in `monitoring_servers` group (inventory-derived) | Which Zabbix server this agent reports to |
| `zabbix_agent_hostname` | `{{ inventory_hostname }}` | How this agent identifies itself to Zabbix — must match host entry in Zabbix UI |
| `zabbix_agent_listen_port` | `10050` | TCP port for passive checks |
| `zabbix_agent_log_file` | `/var/log/zabbix/zabbix_agent2.log` | Log destination |
| `zabbix_agent_log_file_size` | `10` | MB before rotation |
| `zabbix_agent_include_dir` | `/etc/zabbix/zabbix_agent2.d` | Drop-in directory for extra configs |

## Tags

- `zabbix_agent` — everything
- `repo` — just apt repo setup
- `packages` — just install
- `config` — just config + service management

Example: `ansible-playbook playbooks/site.yml --tags config`

## Notes

- **Firewall:** if UFW is later enabled, the Zabbix server needs inbound access to port 10050 on the agent. Handled by whichever firewall role we add — not this role's job.
- **Server side:** until `zabbix-server` role is built and deployed, the agent will run and listen but no server will pull from it. That's expected — the agent process comes up fine and waits patiently.
- **Self-monitor:** on `mon-server`, the agent's `zabbix_server_ip` resolves to `mon-server`'s own public IP. Zabbix server will monitor itself via its own agent.

## Requirements

- Ubuntu 22.04 or 24.04 (asserted by the `common` role dependency)
- Internet access to `repo.zabbix.com` for the initial package + repo install

## Dependencies

- `common` (base OS hardening) — auto-runs before this role
