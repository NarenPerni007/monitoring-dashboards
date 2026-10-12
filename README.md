# monitoring-dashboards

Prometheus alerts and Grafana dashboards from clusters I've run. Nothing exotic — just the set I actually keep.

## Layout

```
prometheus/
  alerts.yml           # node + k8s app alerts (PrometheusRule)
  recording-rules.yml  # pre-aggregated cpu/mem rules the alerts and dashboards share
alertmanager/
  alertmanager.yml     # critical -> pagerduty + slack, warning -> slack, node-down inhibits pod noise
grafana/dashboards/
  k8s-cluster-overview.json  # gauges, pods by namespace, top cpu pods, disk, restarts
  node-overview.json         # cpu, mem, disk, load, network per node
docs/runbooks/
  pod-crashlooping.md
  disk-pressure.md
```

## Apply

```bash
# rules (needs prometheus-operator; plain prometheus: use the rule groups as-is in prometheus.yml)
kubectl apply -f prometheus/

# alertmanager — fill in the two placeholders first, then apply via your existing secret/configmap setup
# grafana — Dashboards > Import, paste the JSON, pick your Prometheus datasource
```

## Replace before using

- `alertmanager/alertmanager.yml`: `<slack-webhook-url>` and `<pagerduty-integration-key>`
- dashboard datasource is `${DS_PROMETHEUS}` — Grafana prompts on import

## Tuning notes

Thresholds I settled on after on-call:

- CPU 85% / 10m (warning) — 90% pages too often on bursty batch nodes, 85 catches real saturation.
- Memory 90% / 10m — memory doesn't lie; if it's pinned at 90 the box is about to OOM something.
- Disk warns at 15% free *and* trending to full in 4h (`predict_linear`). A static threshold alone pages on nodes that sit at 12% free for months.
- CrashLoop alert needs 3+ restarts in 15m sustained 10m — filters out the single bad deploy that fixes itself on retry.
- NodeDown inhibits the pod alerts so one dead node doesn't spam.
