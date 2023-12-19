Grafana Agent installation:
```
helm upgrade --install faro-collector grafana/grafana-agent --values values.yaml -n collector --create-namespace
```