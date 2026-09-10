# firewall

Declaratively opens TCP ports on the iptables INPUT chain, idempotent, persisted across reboot.

## Why

Oracle Cloud Ubuntu images ship with a default iptables policy that rejects everything except port 22:

```
ACCEPT tcp -- 0.0.0.0/0  0.0.0.0/0  state NEW tcp dpt:22
REJECT all -- 0.0.0.0/0  0.0.0.0/0  reject-with icmp-host-prohibited
```

This is separate from Oracle's Security Lists — both firewalls must allow a port for traffic to reach the service. This role handles the OS-side layer.

## What it does

| Step | Details |
|---|---|
| **Install** | Ensures `iptables` + `iptables-persistent` + `netfilter-persistent` present |
| **Insert** | For each declared port, inserts an ACCEPT rule at position 1 (before the REJECT-all catchall). Uses `ansible.builtin.iptables` which is idempotent — won't create duplicate rules |
| **Persist** | Handler fires `netfilter-persistent save` when any rule was added/changed, so rules survive reboot |
| **Comment** | Each rule tagged with `monitoring-lab: <port>` for auditability |

## Variables

| Variable | Default | Description |
|---|---|---|
| `firewall_allowed_tcp_ports` | `[]` | List of TCP ports to open. Each entry is either an integer (opens `0.0.0.0/0`) or a dict `{ port: <n>, source: "<cidr>" }` for source restriction. |
| `firewall_persist_rules` | `true` | Whether to run `netfilter-persistent save` after changes |

## Usage

Declare ports in `group_vars/` or `host_vars/`:

```yaml
# host_vars/mon-server.yml
firewall_allowed_tcp_ports:
  - 8080                                          # Zabbix UI — public
  - { port: 10050, source: "10.0.0.0/16" }        # Zabbix agent — VCN internal
  - { port: 10051, source: "10.0.0.0/16" }        # Zabbix server — VCN internal
```

```yaml
# group_vars/monitoring_targets.yml
firewall_allowed_tcp_ports:
  - { port: 10050, source: "10.0.0.0/16" }        # Zabbix agent — server pulls from us
```

Then add the role to the playbook (before roles that need those ports listening publicly):

```yaml
roles:
  - common
  - firewall
  - zabbix-agent
```

## Tags

- `firewall` — everything
- `packages` — just the apt install
- `rules` — just the iptables insertion

## Dependencies

- `common` — auto-runs before this role
