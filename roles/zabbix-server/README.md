# zabbix-server

Installs and configures Zabbix Server 7.0 LTS with PostgreSQL backend, PHP-FPM, and Nginx frontend on a single Ubuntu host.

## What it does

| Concern | Details |
|---|---|
| **Packages** | PostgreSQL, `zabbix-server-pgsql`, `zabbix-sql-scripts`, `zabbix-frontend-php`, `zabbix-nginx-conf`, `nginx`, `php-fpm`, PHP extensions (pgsql, gd, xml, bcmath, mbstring, ldap), `python3-psycopg2` |
| **PostgreSQL** | Creates `zabbix` DB + user via `community.postgresql` modules (never shells out to `psql` for schema-changing ops except the initial import) |
| **Schema import** | Runs Zabbix's shipped `server.sql.gz` once; idempotent via existence check on the `users` table |
| **Server config** | Templates `/etc/zabbix/zabbix_server.conf` — DB creds, ports, log paths, cache sizes, pollers/trappers |
| **Nginx vhost** | Templates `/etc/zabbix/nginx.conf` (Zabbix ships an included vhost) — listens on `zabbix_frontend_web_port` (default 8080), reverse proxies PHP to PHP-FPM socket (auto-detected) |
| **Frontend config** | Templates `/etc/zabbix/web/zabbix.conf.php` — DB creds, Zabbix server host/port, display name, PHP timezone |
| **Admin rotation** | Rotates default `Admin` / `zabbix` password to `vault_zabbix_admin_password` on first run; marker file `/etc/zabbix/.password-rotated` prevents re-rotation on subsequent runs |
| **Services** | Enables + starts `postgresql`, `zabbix-server`, `nginx`, and the correct `php-fpm` unit (auto-detected — `php8.1-fpm` on 22.04, `php8.3-fpm` on 24.04) |

## Prerequisites

**Before running this role, populate real secrets** in `inventories/production/group_vars/vault.yml`:

```yaml
vault_zabbix_db_password: <a-strong-random-string>
vault_zabbix_admin_password: <a-strong-random-string>
```

The role asserts these are not the placeholder `CHANGEME_*` strings and refuses to run if they are — this is a footgun guard.

**Recommended:** encrypt `vault.yml` with Ansible Vault:

```bash
# One-time setup — create a Vault password file (gitignored)
openssl rand -base64 32 > .vault-pass
chmod 600 .vault-pass

# Edit vault.yml with real values, then encrypt in place
ansible-vault encrypt inventories/production/group_vars/vault.yml

# Re-enable the vault_password_file line in ansible.cfg (currently commented)
```

After that, `ansible-vault edit inventories/production/group_vars/vault.yml` to change secrets. Never commit `.vault-pass` — it's gitignored.

**Also:** open Oracle Cloud security list to allow inbound TCP 8080 from your IP.

## Variables

See `defaults/main.yml` for the full list. Key ones:

| Variable | Default | Description |
|---|---|---|
| `zabbix_db_name` / `zabbix_db_user` | `zabbix` / `zabbix` | Database identity |
| `zabbix_db_password` | `{{ vault_zabbix_db_password }}` | From vault |
| `zabbix_server_listen_port` | `10051` | Agent → server traffic |
| `zabbix_frontend_web_port` | `8080` | Nginx listen port for web UI |
| `zabbix_frontend_display_name` | `monitoring-lab` | Shown in Zabbix UI header |
| `zabbix_frontend_php_timezone` | `{{ timezone }}` (falls back to `UTC`) | PHP `date.timezone` for the frontend |
| `zabbix_admin_password` | `{{ vault_zabbix_admin_password }}` | Rotated once via SQL update |

## Tags

- `zabbix_server` — everything
- `packages`, `postgres`, `db`, `schema`, `config`, `frontend`, `nginx`, `admin_password`, `security`, `services`

Example: `ansible-playbook playbooks/site.yml --tags admin_password` to re-rotate the admin password (delete `/etc/zabbix/.password-rotated` on the host first, or it'll skip).

## Post-run verification

Log into the Zabbix web UI at `http://<mon-server-public-ip>:8080/` with `Admin` / `<vault_zabbix_admin_password>`. You should see the Zabbix dashboard.

Under **Monitoring → Hosts**, add `mon-server` as a monitored host (or configure auto-registration for the agent we already installed) pointing at IP `127.0.0.1` port `10050`.

## Dependencies

- `common` (base OS hardening) — auto-runs before this role
