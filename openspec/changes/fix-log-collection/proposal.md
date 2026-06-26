## Why

The current log collection setup captures only 4 Docker labels (container, image, service, compose_project) and has a display bug where container names show with a leading `/`. The positions file is ephemeral, causing duplicate logs on restart. Docker JSON log envelopes are not parsed, leaving raw JSON in log lines. This change makes the logging stack complete, correct, and metadata-rich.

## What Changes

- Strip leading `/` from Docker container names so they display cleanly (e.g., `grafana` not `/grafana`)
- Collect all available Docker container metadata: container ID, log stream (stdout/stderr), network mode, network name, network IP, exposed ports, container labels, compose labels, hostname, command
- Persist the Promtail positions file via a Docker volume so restarts don't re-read all logs
- Add a JSON pipeline stage to Promtail to parse the Docker JSON log envelope and extract the actual log message
- Provision a pre-built "Docker Logs" Explore view in Grafana

## Capabilities

### New Capabilities

- `docker-log-collection`: Full Docker container metadata collection via Promtail relabeling, including labels, network info, ports, and log stream

### Modified Capabilities

None — no existing specs to modify. First change for this project.

## Impact

- `config/promtail.yml` — add relabel rules for metadata, pipeline stages for JSON parsing, positions volume configuration
- `docker-compose.yaml` — add named volume for Promtail positions, mount it to the positions path
- `config/grafana/provisioning/` — add dashboard/explore provisioning
- No breaking changes to existing data or APIs
