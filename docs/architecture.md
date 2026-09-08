# Architecture

_Placeholder — full architecture write-up + Excalidraw diagram to land alongside first working release._

## Control plane

- **Control node** — any Linux or macOS host with Ansible 2.14+ installed. Runs playbooks over SSH.
- **Inventory** — declarative YAML at `inventories/production/hosts.yml`, split into `monitoring_servers` and `monitoring_targets` groups.
- **Secrets** — Ansible Vault–encrypted file at `inventories/production/group_vars/vault.yml`. Password loaded from `.vault-pass` (never committed).

## Data plane

- **mon-server (1×)** — Zabbix server + PostgreSQL backend + Nginx frontend + Grafana + VictoriaMetrics. Central point for all monitoring data + dashboards + alerts.
- **mon-target-* (N×)** — Zabbix agent + Telegraf + application under test (dummy web service). Each target sends metrics to both Zabbix (server-pull) and VictoriaMetrics via Telegraf (agent-push).

## Data flow

```
Target host ──── Zabbix agent (server pulls) ────► Zabbix server ── stores in ──► PostgreSQL
     │
     └────────── Telegraf (agent pushes) ──────────► VictoriaMetrics
                                                            │
Grafana ─── queries both Zabbix + VictoriaMetrics ─────────┘
    │
    └── panels + alerts ──► Slack webhook
```

## Alerting

Two independent alert paths — deliberate, to demonstrate multi-tool alerting:

1. **Zabbix triggers** → Slack webhook (via Zabbix media type)
2. **Grafana alert rules** → Slack webhook (via Grafana contact point)

Both land in the same `#monitoring-lab-alerts` Slack channel.

## Diagrams

_Excalidraw source will live at `docs/diagrams/` and be exported to `docs/diagrams/*.png` for the README._
