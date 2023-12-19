Grafana Agent installation:
```
helm upgrade --install grafana-agent-collector grafana/grafana-agent --values values.yaml -n collector --create-namespace
```