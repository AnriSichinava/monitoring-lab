# telegraf

Installs Telegraf on monitoring targets and ships system metrics (CPU, memory, disk, diskio, net, processes) to VictoriaMetrics running on mon-server, using the InfluxDB v1 line-protocol output — VictoriaMetrics accepts this natively without needing a real InfluxDB.

## What it does

| Step | Details |
|---|---|
| **APT repo** | Adds InfluxData's official APT repo + signing key (keyring at `/etc/apt/keyrings/influxdata-archive_compat.asc`) |
| **Install** | `apt install telegraf` |
| **Config** | Deploys `/etc/telegraf/telegraf.conf` — collection interval, output URL, input plugins |
| **Service** | `telegraf` enabled + started; restart handler fires on config change |

## Why InfluxDB output, not Prometheus remote-write

Telegraf's `outputs.influxdb` plugin pushes on its own schedule (agent-push), which is simpler to reason about here than `outputs.prometheusremotewrite` plus a scrape config. VictoriaMetrics is wire-compatible with InfluxDB v1's `/write` endpoint, so no translation layer is needed — same idea as pointing a real InfluxDB client at VM.

## Variables

See `defaults/main.yml`. Key ones:

| Variable | Default | Description |
|---|---|---|
| `telegraf_collection_interval` | `15s` | How often each input plugin polls |
| `telegraf_victoriametrics_host` | mon-server's `private_ip` | Resolved automatically from inventory — VCN-internal, not public IP |
| `telegraf_victoriametrics_port` | `8428` | VictoriaMetrics HTTP listener |
| `telegraf_influxdb_database` | `telegraf` | Required by the output plugin; ignored by VictoriaMetrics itself |

## Dependencies

- `common` — base OS hardening runs first
- Requires `victoriametrics` already running on mon-server and port 8428 reachable from the target's private IP (see `roles/victoriametrics/README.md` for the firewall rule)
