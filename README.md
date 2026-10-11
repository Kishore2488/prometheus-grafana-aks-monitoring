# Prometheus + Grafana Monitoring Stack

![Validate](https://github.com/Kishore2488/prometheus-grafana-aks-monitoring/actions/workflows/validate.yml/badge.svg)

A runnable monitoring stack — Prometheus, Grafana, and the alerting rules I'd ship alongside an AKS cluster. Spin it up locally with Docker Compose to see it working end to end.

## Run it

```bash
docker compose up -d
# Prometheus:  http://localhost:9090
# Grafana:     http://localhost:3000  (admin / admin)
```

Grafana comes pre-provisioned with the Prometheus datasource, so there's nothing to click through manually.

## What's in it

- `prometheus/prometheus.yml` — scrape config (self-monitoring, ready to add AKS/kube-state-metrics targets).
- `prometheus/alert-rules.yml` — alerting rules for high CPU, pod restart loops, and target-down detection.
- `grafana/provisioning/datasources/datasource.yml` — auto-provisions the Prometheus datasource so Grafana is usable immediately after `docker compose up`.

## CI

The alert rules and scrape config are validated with `promtool check rules` / `promtool check config` in GitHub Actions on every push — a syntax error in an alerting rule is exactly the kind of thing that should fail CI, not get discovered during an incident.

## Why these alerts

- **Pod CPU > 85% for 10m** — catches sustained pressure before it becomes an eviction.
- **Pod restart count increasing** — a crash-looping pod is a signal worth paging on, not just logging.
- **Scrape target down** — the one alert that can't fully cover Prometheus's own downtime, so in production it's paired with an external dead-man's-switch heartbeat check rather than relying on a local rule alone (noted in the rule's comment).
