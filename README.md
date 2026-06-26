# Grafana Monitoring Stack

Grafana + Loki + Prometheus + Promtail + Node Exporter + cAdvisor.

## Quick Start

```bash
# 1. Configure ports (edit .env if defaults conflict)
cp .env.example .env   # or edit .env directly

# 2. Start everything
docker compose up -d

# 3. Verify
docker compose ps
```

## Services

| Service       | URL                        | Default Port |
| ------------- | -------------------------- | ------------ |
| Grafana       | http://localhost:3000       | 3000         |
| Prometheus    | http://localhost:9090       | 9090         |
| Loki          | http://localhost:3100       | 3100         |

- **Grafana login**: `admin` / `changeme` (set in `.env`)
- **Prometheus targets**: http://localhost:9090/targets
- **Loki ready**: http://localhost:3100/ready

## Configuration

All ports and credentials live in `.env`:

```env
LOKI_PORT=3100
PROMETHEUS_PORT=9090
GRAFANA_PORT=3000
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=changeme
```

Edit `.env` to change ports or credentials, then restart:

```bash
docker compose down && docker compose up -d
```

## Data Sources in Grafana

After first login, add data sources:

1. **Prometheus** — URL: `http://prometheus:9090`
2. **Loki** — URL: `http://loki:3100`

Or import the provisioned dashboards from `config/grafana/provisioning/`.

## Useful Commands

```bash
docker compose down          # Stop and remove containers
docker compose up -d         # Start in background
docker compose logs -f       # Tail all logs
docker compose restart       # Restart all services
docker compose ps            # List running containers
```
