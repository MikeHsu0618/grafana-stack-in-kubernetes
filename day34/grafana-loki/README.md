```
helm upgrade --install loki grafana/loki-distributed -f values.yaml -f config.yaml -n logging --create-namespace

helm upgrade --install loki grafana/loki-distributed -f values.yaml -f config.yaml -n logging --version 0.74.2
```