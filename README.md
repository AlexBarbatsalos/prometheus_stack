# Prometheus Monitoring Stack

A ready-to-deploy monitoring stack for small businesses and single-server environments. Docker Compose-based, opinionated for simplicity, designed to be deployed per client.

## What's included

- **Prometheus** — metrics collection and alerting engine
- **Grafana** — dashboards, pre-configured with Prometheus datasource and Node Exporter dashboard
- **Alertmanager** — alert routing and notification (email)
- **Node Exporter** — host-level metrics (CPU, memory, disk, network)
- **cAdvisor** — per-container resource metrics
- **Blackbox Exporter** — external endpoint probing (HTTP, TLS)

## Quick start

```bash
git clone https://github.com/YOUR_USERNAME/prometheus-stack.git
cd prometheus-stack
cp .env.example .env
```

Edit `.env` with your Grafana admin password and desired retention period.

Edit `alertmanager/alertmanager.yml` with your SMTP credentials.

Edit `prometheus/targets/blackbox_http.yml` with the URLs you want to monitor.

```bash
docker compose up -d
```

Grafana is available at http://localhost:3000.

## Configuration

### Monitored URLs

Add or remove HTTP targets in `prometheus/targets/blackbox_http.yml`. Prometheus picks up changes automatically within 30 seconds — no restart needed.

```yaml
- targets: ['https://example.com']
  labels:
    name: 'my-website'
    environment: 'production'
```

### Custom exporters

If the client runs application-specific exporters (PostgreSQL, MySQL, Redis, etc.), add them to `prometheus/targets/custom_exporters.yml`.

### Alert rules

Alert rules live in `prometheus/rules/alerts.yml`. Included out of the box:

| Alert | Condition | Severity |
|---|---|---|
| TargetDown | Any scrape target unreachable for 2m | critical |
| WebsiteDown | HTTP probe failing for 2m | critical |
| TlsCertExpiringSoon | TLS certificate expires within 14 days | warning |
| SlowProbe | HTTP probe > 3s for 5m | warning |
| HighMemoryUsage | Memory usage > 90% for 5m | warning |
| DiskSpaceLow | Disk usage > 85% for 10m | warning |
| HighCpuUsage | CPU usage > 90% for 10m | warning |

### Alertmanager

Edit `alertmanager/alertmanager.yml` directly with SMTP credentials. The default routing sends all alerts via email.

### Grafana dashboards

Provisioned dashboards live in `grafana/provisioning/dashboards/json/`. To add a community dashboard:

1. Download the JSON from grafana.com
2. Replace datasource references: `sed -i 's/${DS_PROMETHEUS}/prometheus/g' filename.json`
3. Also check for lowercase variant: `sed -i 's/${ds_prometheus}/prometheus/g' filename.json`
4. Drop it into the `json/` directory
5. Restart Grafana: `docker compose restart grafana`

## Ports (development)

| Service | Port |
|---|---|
| Grafana | 3000 |
| Prometheus | 9090 |
| Alertmanager | 9093 |
| Node Exporter | 9100 |
| cAdvisor | 8080 |
| Blackbox Exporter | 9115 |

In production, only Grafana should be exposed — ideally behind a reverse proxy with TLS.

## Validating configuration

```bash
# Check Prometheus config and rules
docker compose exec prometheus promtool check config /etc/prometheus/prometheus.yml

# Reload Prometheus without restart
curl -X POST http://localhost:9090/-/reload
```

## Roadmap

- [ ] Caddy reverse proxy with automatic TLS
- [ ] docker-compose.dev.yml for dev/prod split
- [ ] Docker/cAdvisor dashboard
- [ ] Blackbox overview dashboard
