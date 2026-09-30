# victoriametrics

Installs VictoriaMetrics (single-node) on the mon-server as a systemd-managed binary. Long-term metrics store fed by `telegraf` agents running on every host.

## What it does

| Step | Details |
|---|---|
| **Install** | Downloads the arch-matched release tarball from GitHub (no apt package exists), extracts `victoria-metrics-prod` to `/usr/local/bin`, skips re-download if the pinned version is already installed |
| **User** | Creates a dedicated system user/group; the binary never runs as root |
| **Config** | Deploys a systemd unit with `-storageDataPath`, `-retentionPeriod`, `-httpListenAddr` flags |
| **Service** | `victoriametrics.service` enabled + started; restart handler fires on binary or unit change |

## Prerequisites

**Open port 8428/tcp** in `host_vars/mon-server.yml`, restricted to the VCN (targets push metrics here over Telegraf, not the public internet):

```yaml
firewall_allowed_tcp_ports:
  - { port: 8428, source: "10.0.0.0/16" }
```

Also open it in the Oracle Security List's VCN ingress rules.

## Variables

See `defaults/main.yml`. Key ones:

| Variable | Default | Description |
|---|---|---|
| `victoriametrics_version` | `v1.153.0` | Pinned release tag |
| `victoriametrics_listen_port` | `8428` | HTTP API + Prometheus remote-write + InfluxDB line-protocol listener |
| `victoriametrics_retention_period` | `30d` | How long raw samples are kept |
| `victoriametrics_data_dir` | `/var/lib/victoria-metrics` | On-disk storage path |

## Post-run access

`curl http://<mon-server-private-ip>:8428/api/v1/query?query=up` from inside the VCN, or add it as a Prometheus-compatible datasource in Grafana pointed at `http://127.0.0.1:8428`.

## Dependencies

- `common` — base OS hardening runs first
- `firewall` — port 8428 must be declared in the host's `firewall_allowed_tcp_ports`
