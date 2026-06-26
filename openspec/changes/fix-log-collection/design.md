## Context

Current stack: Promtail → Loki → Grafana. Promtail uses `docker_sd_configs` to discover containers via Docker socket and reads logs from `/var/lib/docker/containers/*/*.log`. Configuration is in `config/promtail.yml`. Loki stores logs with 30-day retention (TSDB + filesystem backend). No breaking changes — all modifications are additive or corrective.

## Goals / Non-Goals

**Goals:**
- Fix container name display (strip leading `/`)
- Collect all Docker SD metadata labels: container_id, log_stream, network_mode, network_name, network_ip, container labels, compose labels, ports, hostname
- Persist Promtail positions file across container restarts
- Parse Docker JSON log envelope to extract actual log message
- Provision a pre-built Loki Explore dashboard in Grafana

**Non-Goals:**
- Changing Loki retention or compactor config (already correct)
- Adding system-level log scraping (syslog, journald)
- Multi-node / clustered Loki
- Authentication for Loki or Prometheus

## Decisions

### D1: Use `labelmap` for all Docker container labels

Using `action: labelmap` with regex `__meta_docker_container_label_(.+)` captures ALL user-defined Docker labels (including compose labels like `com.docker.compose.service`) without needing to enumerate them. The existing explicit `service` and `compose_project` relabels become redundant and can be removed.

### D2: Positions file path

Move positions file from `/tmp/positions.yaml` to `/positions/positions.yaml` with a named volume `promtail_positions` mounted at `/positions`. This is cleaner than mounting a host path and works across restarts.

### D3: JSON parsing pipeline

Add a `pipeline_stages` block under the docker scrape config with a `json` stage to parse the Docker log envelope (`{"log":"...","stream":"stdout","time":"..."}`). Use `output` stage with `source: log` to replace the log line with just the message content. Keep the original JSON fields as additional labels/metadata.

### D4: Container name fix

Replace the original relabel with one that uses `regex: /?(.*)` and `replacement: $1` to strip the leading `/`. Alternatively, use `regex: /(.*)` since Docker always prepends `/`.

### D5: Dashboard provisioning

Create a Grafana dashboard JSON file provisioned to explore Loki logs with the `container` label as the primary filter. Use Grafana's Explore link (not a full dashboard) for simplicity, or a minimal dashboard with the "Logs" panel type.

## Risks / Trade-offs

- **Label cardinality**: Container labels vary per container → high label cardinality in Loki. Mitigation: Loki handles this fine at ~dozens of containers. Only a concern at 1000+ unique label combinations.
- **labelmap captures all**: If a container has sensitive data in labels (e.g., passwords as labels), they end up in Loki. Mitigation: this is a local-only stack (auth disabled, single host). Document this.
- **Positions volume**: Named Docker volume persists even after `docker-compose down`. On `docker-compose down -v`, volume is destroyed and positions reset. Minor inconvenience, standard Docker behavior.
