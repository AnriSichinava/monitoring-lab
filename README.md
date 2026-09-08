# monitoring-lab

Ansible-deployed observability stack for Ubuntu Linux hosts. Point it at fresh Ubuntu 24.04 boxes, run one command, get a complete monitoring platform: metrics collection, long-term storage, dashboards, and alerting.

## Stack

| Component | Purpose |
|---|---|
| **Ansible** | Configuration management — playbooks + roles, one role per component |
| **Zabbix** | Monitoring server, agents on every host, custom templates for services |
| **VictoriaMetrics** | Long-term metrics store (Prometheus-compatible) |
| **Telegraf** | Metrics collector, runs on every host, ships to VictoriaMetrics |
| **Grafana** | Dashboards + visualization on top of Zabbix and VictoriaMetrics |
| **PostgreSQL** | Zabbix backend database |
| **Nginx** | Reverse proxy for Zabbix frontend |
| **Slack** | Alert delivery |
| **GitLab CI** | Lint → molecule tests → staged deploy |

## Architecture

```
                             ┌──────────────────────────────┐
                             │      Control node            │
                             │  (Ansible + git + SSH keys)  │
                             └───────────────┬──────────────┘
                                             │ SSH
                       ┌─────────────────────┼─────────────────────┐
                       ▼                     ▼                     ▼
              ┌────────────────┐   ┌────────────────┐   ┌────────────────┐
              │   mon-server   │   │  mon-target-1  │   │  mon-target-2  │
              │                │   │                │   │                │
              │  Zabbix svr    │   │  Zabbix agent  │   │  Zabbix agent  │
              │  PostgreSQL    │   │  Telegraf      │   │  Telegraf      │
              │  Grafana       │   │  Dummy web svc │   │  Dummy web svc │
              │  VictoriaMet.  │   │                │   │                │
              │  Nginx         │   │                │   │                │
              └───────┬────────┘   └────────────────┘   └────────────────┘
                      │
                      │ webhook
                      ▼
                  ┌───────┐
                  │ Slack │
                  └───────┘
```

## Repository layout

```
monitoring-lab/
├── ansible.cfg                 # Ansible defaults (SSH, inventory, roles path)
├── requirements.yml            # Ansible collections + roles from Galaxy
├── inventories/
│   └── production/
│       ├── hosts.yml           # Real host inventory (Oracle Cloud VMs)
│       ├── group_vars/
│       │   ├── all.yml         # Vars for every host
│       │   └── vault.yml       # Encrypted secrets (Ansible Vault)
│       └── host_vars/
├── playbooks/
│   ├── site.yml                # Master playbook — runs everything
│   ├── monitoring-server.yml   # mon-server only
│   └── targets.yml             # mon-target-* only
├── roles/
│   ├── common/                 # Base OS hardening, users, packages
│   ├── zabbix-server/          # Zabbix server + PostgreSQL + Nginx
│   ├── zabbix-agent/           # Agent install, registration
│   ├── victoriametrics/        # Single-binary install, systemd, retention
│   ├── telegraf/               # Collector config, output to VictoriaMetrics
│   └── grafana/                # Grafana + provisioned dashboards + datasources
├── molecule/                   # Role test scenarios
├── .gitlab-ci.yml              # Lint → molecule → deploy pipeline
└── docs/
    ├── architecture.md
    └── screenshots/
```

## Quickstart

Requires an Ansible control node (Linux or macOS) and 3 fresh Ubuntu 24.04/22.04 hosts reachable over SSH.

```bash
# 1. Clone the repo
git clone git@github.com:<your-user>/monitoring-lab.git
cd monitoring-lab

# 2. Install Ansible collections
ansible-galaxy install -r requirements.yml

# 3. Update inventory with your host IPs
$EDITOR inventories/production/hosts.yml

# 4. Set the Vault password (one time)
echo "your-vault-password" > .vault-pass && chmod 600 .vault-pass

# 5. Run the full site playbook
ansible-playbook -i inventories/production playbooks/site.yml
```

## Deliverables — build progress

- [ ] Repo scaffold + first commit
- [ ] `common` role (hardening, users, SSH, packages, timezone, unattended-upgrades)
- [ ] `zabbix-server` role (Zabbix + PostgreSQL + Nginx frontend, admin pw rotation)
- [ ] `zabbix-agent` role (install, config, registration)
- [ ] `victoriametrics` role (single-binary + systemd + retention)
- [ ] `telegraf` role (config from template, output to VictoriaMetrics)
- [ ] `grafana` role (install, provisioned datasources + 4 dashboards)
- [ ] 3 custom Zabbix templates (linux baseline, HTTP check, log-triggers)
- [ ] Slack alerts wired from Zabbix + Grafana
- [ ] Molecule tests for `common` and `zabbix-agent`
- [ ] GitLab CI pipeline (lint → molecule → staged deploy)
- [ ] Architecture diagram (Excalidraw) + dashboard screenshots

## What I learned

_This section will grow as the project ships. Placeholder for now._

## License

MIT — see [LICENSE](LICENSE).
