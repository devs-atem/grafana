## ADDED Requirements

### Requirement: Container name SHALL be cleaned of leading slash
Docker API returns container names with a leading `/` (e.g., `/grafana`). The system SHALL strip this prefix so the `container` label value is the raw container name.

#### Scenario: Container started with docker-compose
- **WHEN** a container named `/promtail` is discovered via Docker SD
- **THEN** the `container` label SHALL be set to `promtail`

#### Scenario: Container started with plain docker run
- **WHEN** a container named `/my-app` is discovered via Docker SD
- **THEN** the `container` label SHALL be set to `my-app`

### Requirement: All Docker container metadata SHALL be collected as Loki labels
The system SHALL collect the following metadata from Docker SD and attach it as Loki labels for every log stream:

| Metadata | Label name | Source |
|----------|-----------|--------|
| Container ID | `container_id` | `__meta_docker_container_id` |
| Container name | `container` | `__meta_docker_container_name` (cleaned) |
| Container image | `image` | `__meta_docker_container_image` |
| Log stream | `stream` | `__meta_docker_container_log_stream` |
| Network mode | `network_mode` | `__meta_docker_container_network_mode` |
| Network name | `network_name` | `__meta_docker_container_network_name` |
| Network IP | `network_ip` | `__meta_docker_container_network_ip` |
| Hostname | `hostname` | `__meta_docker_container_hostname` |
| All container labels | `label_*` | `__meta_docker_container_label_*` (via labelmap) |
| All container ports | `port_*` | `__meta_docker_port_*` (via labelmap) |

#### Scenario: Container with labels discovered
- **WHEN** a container has Docker labels `com.docker.compose.service=web` and `environment=production`
- **THEN** the log stream SHALL include labels `label_com_docker_compose_service=web` and `label_environment=production`

#### Scenario: Container with exposed ports discovered
- **WHEN** a container exposes port 8080
- **THEN** the log stream SHALL include `port_8080` label with the mapped port value

#### Scenario: Container without compose labels discovered
- **WHEN** a container has no Docker labels
- **THEN** no `label_*` labels SHALL be present on the log stream
- **THEN** the log stream SHALL still be collected with all other metadata labels

### Requirement: Promtail positions file SHALL persist across container restarts
The Promtail positions file (which tracks read progress on log files) SHALL be mounted on a named Docker volume so it survives container recreation.

#### Scenario: Promtail container is recreated
- **WHEN** `docker-compose down && docker-compose up -d` is executed
- **THEN** Promtail SHALL read its previous log positions from the persisted volume
- **THEN** no duplicate log entries SHALL appear in Loki for already-read log content

#### Scenario: Promtail container is simply restarted
- **WHEN** `docker restart promtail` is executed
- **THEN** positions SHALL be preserved (container filesystem survives restart)
- **THEN** log tailing resumes from the last known position

### Requirement: Docker JSON log envelope SHALL be parsed
Docker `json-file` log driver wraps each log line in JSON (`{"log":"actual message","stream":"stdout","time":"2024-01-01T00:00:00Z"}`). The system SHALL parse this envelope so that only the actual log message is stored as the log line content.

#### Scenario: stdout log line from container
- **WHEN** a container writes `Hello World` to stdout
- **THEN** Loki SHALL store `Hello World` as the log line (not the raw JSON envelope)
- **THEN** the `stream` label SHALL be `stdout`

#### Scenario: stderr log line from container
- **WHEN** a container writes an error to stderr
- **THEN** Loki SHALL store the error message as the log line
- **THEN** the `stream` label SHALL be `stderr`

### Requirement: Grafana SHALL have a pre-provisioned Loki log explorer
A dashboard or explore link SHALL be provisioned in Grafana that opens the Loki datasource filtered by container name for log exploration.

#### Scenario: Grafana starts for the first time
- **WHEN** Grafana starts with the provisioning directory mounted
- **THEN** a "Docker Logs" dashboard SHALL be available
- **THEN** the dashboard SHALL query Loki with the `{job="docker"}` log stream selector

#### Scenario: User clicks dashboard link
- **WHEN** a user opens the "Docker Logs" dashboard
- **THEN** they SHALL see an Explore view pointed at Loki with container log data
