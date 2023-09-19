Promtail 安裝：
```
helm upgrade --install promtail grafana/promtail -f promtail/values.yaml -n logging --create-namespace
```

重要參數：
```
# -- Overrides the chart's name
nameOverride: null
# -- Overrides the chart's computed fullname
fullnameOverride: null

daemonset:
  # -- Deploys Promtail as a DaemonSet
  enabled: true

deployment:
  # -- Deploys Promtail as a Deployment
  enabled: false
  
serviceMonitor:
  # -- If enabled, ServiceMonitor resources for Prometheus Operator are created
  enabled: true
  
prometheusRule:
  enabled: true
```