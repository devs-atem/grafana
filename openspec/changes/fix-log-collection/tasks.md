## 1. Fix container name and positions

- [x] 1.1 Update `config/promtail.yml`: fix container name relabel to strip leading `/`
- [x] 1.2 Add `promtail_positions` named volume to `docker-compose.yaml`
- [x] 1.3 Mount positions volume in promtail service at `/positions`
- [x] 1.4 Update positions filename in `config/promtail.yml` to `/positions/positions.yaml`

## 2. Add full Docker metadata collection

- [x] 2.1 Add `container_id` relabel from `__meta_docker_container_id` in `config/promtail.yml`
- [x] 2.2 Add `stream` relabel from `__meta_docker_container_log_stream`
- [x] 2.3 Add `network_mode` relabel from `__meta_docker_container_network_mode`
- [x] 2.4 Add `network_ip` relabel (`network_name` not available as Docker SD meta label — skipped)
- [x] 2.5 Skipped: `__meta_docker_container_hostname` not available in Promtail Docker SD
- [x] 2.6 Add `labelmap` for all Docker container labels (`__meta_docker_container_label_*`)
- [x] 2.7 Skipped: port labelmap not reliable in Promtail (targets are per-container, not per-port)
- [x] 2.8 Kept explicit `service`/`compose_project` relabels — labelmap produces different label names; clean names are more usable

## 3. Add JSON log parsing pipeline

- [x] 3.1 Add `pipeline_stages` to the docker scrape config in `config/promtail.yml`
- [x] 3.2 Add `json` stage to parse Docker log envelope fields
- [x] 3.3 Add `output` stage with `source: log` to extract actual message
- [x] 3.4 Add `timestamp` stage to parse Docker log timestamp

## 4. Provision Grafana log dashboard

- [x] 4.1 Create `config/grafana/provisioning/dashboards/` directory
- [x] 4.2 Create `config/grafana/provisioning/dashboards/dashboards.yml` config
- [x] 4.3 Create `config/grafana/provisioning/dashboards/docker-logs.json` dashboard
- [x] 4.4 Already covered by existing `./config/grafana/provisioning` mount (no change needed)

## 5. Verify

- [ ] 5.1 Run `docker-compose up -d` and verify all services start
- [ ] 5.2 Query Loki: `{job="docker"}` shows labels without leading `/`
- [ ] 5.3 Verify `container_id`, `stream`, `network_mode`, `hostname` labels present
- [ ] 5.4 Verify `label_*` labels present for compose containers
- [ ] 5.5 Verify log lines show actual messages, not JSON envelope
- [ ] 5.6 Open Grafana at :3000, check Docker Logs dashboard
- [ ] 5.7 Restart promtail and verify no duplicate logs
- [ ] 5.8 Run `docker-compose down && docker-compose up -d` and check positions preserved
