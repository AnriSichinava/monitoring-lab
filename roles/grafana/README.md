# grafana

Installs Grafana OSS on the mon-server, provisions the Zabbix datasource + 2 starter dashboards, and rotates the default admin password.

## What it does

| Step | Details |
|---|---|
| **APT repo** | Adds Grafana Labs official APT repo + signing key (keyring at `/etc/apt/keyrings/grafana.asc`) |
| **Install** | `apt install grafana` |
| **Plugin** | Installs `alexanderzobnin-zabbix-app` via `grafana-cli plugins install` (this is what lets Grafana query Zabbix's API) |
| **Config** | Deploys `/etc/grafana/grafana.ini` — port, domain, security settings |
| **Provisioning: datasource** | `/etc/grafana/provisioning/datasources/zabbix.yml` — declares the Zabbix datasource with API creds |
| **Provisioning: dashboards** | `/etc/grafana/provisioning/dashboards/monitoring-lab.yml` (provider config) + `/var/lib/grafana/dashboards/*.json` (dashboards themselves) |
| **Admin rotation** | Grafana ships with `admin/admin`; role hits `/api/user/password` once with basic auth to change it to `vault_grafana_admin_password`. Marker file `/etc/grafana/.password-rotated` prevents re-rotation |
| **Service** | `grafana-server` enabled + started; restart handler fires on config change |

## Prerequisites

**Before running this role:**

1. **Create a read-only Zabbix API user** for Grafana to authenticate as. In the Zabbix UI:
   - Users → Users → Create user
   - Username: `grafana-api`
   - Password: whatever you set in `vault_zabbix_api_password`
   - Role: `Guest role` (read-only)
   - User groups: `Guests`
   Never let Grafana use the `Admin` account — that's an account with write permissions.

2. **Populate vault secrets** in `inventories/production/group_vars/all/vault.yml`:
   ```yaml
   vault_grafana_admin_password: <strong-random>
   vault_zabbix_api_user: grafana-api
   vault_zabbix_api_password: <same-as-Zabbix-user-above>
   ```
   The role asserts these are not the placeholder values.

3. **Open port 3000/tcp** in your `host_vars/mon-server.yml` firewall rule list:
   ```yaml
   firewall_allowed_tcp_ports:
     - 3000    # Grafana UI — public
     - 8080
     - { port: 10050, source: "10.0.0.0/16" }
     - { port: 10051, source: "10.0.0.0/16" }
   ```

4. **Open port 3000/tcp in the Oracle Security List** (same VCN Ingress Rules page as before).

## Variables

See `defaults/main.yml` for the full list. Key ones:

| Variable | Default | Description |
|---|---|---|
| `grafana_listen_port` | `3000` | Grafana web listen port |
| `grafana_admin_username` | `admin` | Rotated password's target user |
| `grafana_admin_password` | `{{ vault_grafana_admin_password }}` | From vault |
| `grafana_zabbix_datasource_url` | `http://127.0.0.1:{{ zabbix_frontend_web_port }}/api_jsonrpc.php` | Zabbix API endpoint |
| `grafana_zabbix_api_user` / `grafana_zabbix_api_password` | vault | Zabbix API credentials |
| `grafana_plugins` | `[alexanderzobnin-zabbix-app]` | Plugins to install |

## Post-run access

Browser: `http://<mon-server-public-ip>:3000/`
- Username: `admin`
- Password: `{{ vault_grafana_admin_password }}`

You should see a Grafana home page. Under **Dashboards → Browse → monitoring-lab**, you'll find:
- **Infra Overview** — CPU / memory / disk / network across all hosts
- **Per-Host Deep Dive** — dropdown to select a host, then detailed panels

## Dependencies

- `common` — base OS hardening runs first
