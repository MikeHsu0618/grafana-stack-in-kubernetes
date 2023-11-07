# 可觀測性宇宙的第十五天 - Kube-Prometheus-Stack 實戰（三）

# 概述

在前兩章節的 Kube-Prometheus-Stack 實作經驗之後，相信大多數的讀者都成功搭建了屬於自己的 Prometheus 監控系統。然而，在真實的工作場景中，由於每個人、或每家公司所處的環境都有其獨特性，加上軟體不一定能在各種環境中呈現完美一致的運作。因此，即使嚴格按照教學步驟執行，結果可能與預期有所偏差，這些出入或問題，有些人俗稱為「坑」。

工程師真正的價值在於如何克服和填平這些「坑」。本章節中，我們將深入探討在 Docker Desktop with Kubernetes 環境中建立 Kube-Prometheus-Stack 時可能遇到的問題，以及針對這些問題的解決策略。

## Kube-Prometheus-Stack 上常見問題與解法

在安裝 Kube-Prometheus-Stack 時，常因許多運行環境因素差異，而導致 Prometheus 系統不能正常運作，以下就來看看這些問題，以及他們的解法。

## Node Exporter is repeatedly crashing

```jsx
Error: failed to start container "node-exporter": Error response from daemon: path /sys is mounted on /sys but it is not a shared or slave mount
```

![https://ithelp.ithome.com.tw/upload/images/20230930/20149562x1zdoEv1rQ.png](https://ithelp.ithome.com.tw/upload/images/20230930/20149562x1zdoEv1rQ.png)

如果發生 Node Exporter 發生不斷重啟的現象，這些多半是因為當前運行環境並不允許讀取根目錄，而這時將 hostRootFsMount 設定為 false ，可以解決大部分重啟的問題，如下：

```jsx
# values.yaml
nodeExporter:
  enabled: true
  hostRootfs: false

prometheus-node-exporter:
  hostRootFsMount:
    enabled: false
```

這個問題在過去的 Docker Desktop with Kubernetes 中時常被發現，但目前最新版本已沒有存在這個問題。

ref:

- https://github.com/prometheus-community/helm-charts/issues/467
- https://stackoverflow.com/questions/70556984/kubernetes-node-exporter-container-is-not-working-it-shows-this-error-message

## Unable to Scrape Metrics from Core Components

Kube-Prometheus-Stack 提供我們抓取在 Kubernetes 叢集中命名空間 kube-system 核心元件指標的選項，因為核心元件的安裝及管理方式不同，無法直接套用到現成的設定。

![https://ithelp.ithome.com.tw/upload/images/20230930/201495623ve2FqxSoH.png](https://ithelp.ithome.com.tw/upload/images/20230930/201495623ve2FqxSoH.png)

其中主要原因大致如下：

- Kubernetes 叢集主節點及核心元件，受到雲平台 Kubernetes 代管服務限制，所以無法存取，如 Google GKE、 AWS Eks、Azure Aks 等等。
- 使用的 Kubernetes 叢集啟動工具，如我們使用的 Docker Desktop，預設只給 127.0.0.1 訪問而非 0.0.0.0。

所以當我們點開如 Kubernetes / Scheduler 預設圖表時，並無法得到所需指標，導致 No data 的問題出現。

![https://ithelp.ithome.com.tw/upload/images/20230930/20149562dNlw5DkLG6.png](https://ithelp.ithome.com.tw/upload/images/20230930/20149562dNlw5DkLG6.png)

基本上，除了雲端平台提供的 Kubernetes 代管服務是無法自己控制的，在其他自建的 Kubernetes 叢集中，多半可以刪考下列各種前人的經驗調整預設核心元件的 —bind-address 或 port 來解決：

- https://dewble.tistory.com/entry/prometheus-alert-firingetcdkubeletkube-proxy-in-kubeadm-Kubernetes
- https://github.com/prometheus-operator/kube-prometheus/issues/718
- https://groups.google.com/g/prometheus-users/c/_aI-HySJ-xM/m/kqrL1FYVCQAJ

或者也可以關閉對核心元件指標的抓取，以及預設圖表的載入。

```jsx
# valus.yaml
kubeScheduler:
  enabled: false

kubeControllerManager:
  enabled: false

kubeEtcd:
  enabled: false

kubeProxy:
  enabled: false
```

## Kubelet 內部 cAdvisor 無法取得特定 Label

由於 Kubernetes 在 v1.24 後，剔除對 docker shim 的依賴，導致 Kubelet 內部原本用來收集各種容器指標的 cAdvisor 服務，沒辦法取得有關 image、container 等指標，直到目前這個問題依然持續發生著：

- https://github.com/google/cadvisor/issues/3162
- https://github.com/prometheus-community/helm-charts/issues/3058
- https://github.com/google/cadvisor/issues/3336

這造成我們 Kube-Prometheus-Stack 中許多關於服務資源使用量的圖表無法取得數據，大部分的圖表都沒辦法正常顯示。

![https://ithelp.ithome.com.tw/upload/images/20230930/20149562oGZr1gUWgt.png](https://ithelp.ithome.com.tw/upload/images/20230930/20149562oGZr1gUWgt.png)

目前社群上的解決方法為自己替 Kubelet 在 Kubernetes 叢集建立 cAdvisor 來產生正常的容器資源指標，以解決 Kubelet 在 v1.24 升級後普遍遇到的的問題。

首先讓我們先將 Kubelet 的 cAdvisor 功能關閉：

```jsx
kubelet:
  enabled: true
  serviceMonitor:
    cAdvisor: false
```

接下來，我們需要自訂安裝 cAdvisor 服務並建立 ServiceMonitor 資源，使 Prometheus 拉取容器指標：

```jsx
# cAdvisor.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: cadvisor
  name: cadvisor
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app: cadvisor
  name: cadvisor
rules:
  - apiGroups:
      - policy
    resourceNames:
      - cadvisor
    resources:
      - podsecuritypolicies
    verbs:
      - use
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    app: cadvisor
  name: cadvisor
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cadvisor
subjects:
  - kind: ServiceAccount
    name: cadvisor
    namespace: kube-system
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  annotations:
    seccomp.security.alpha.kubernetes.io/pod: docker/default
  labels:
    app: cadvisor
  name: cadvisor
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: cadvisor
      name: cadvisor
  template:
    metadata:
      labels:
        app: cadvisor
        name: cadvisor
    spec:
      automountServiceAccountToken: false
      containers:
        - args:
            - --housekeeping_interval=10s
            - --max_housekeeping_interval=15s
            - --event_storage_event_limit=default=0
            - --event_storage_age_limit=default=0
            - --enable_metrics=app,cpu,disk,diskIO,memory,network,process
            - --docker_only
            - --store_container_labels=false
            - --whitelisted_container_labels=io.kubernetes.container.name,io.kubernetes.pod.name,io.kubernetes.pod.namespace
          image: gcr.io/cadvisor/cadvisor:v0.45.0
          name: cadvisor
          ports:
            - containerPort: 8080
              name: http
              protocol: TCP
          resources:
            limits:
              cpu: 800m
              memory: 2000Mi
            requests:
              cpu: 400m
              memory: 400Mi
          volumeMounts:
            - mountPath: /rootfs
              name: rootfs
              readOnly: true
            - mountPath: /var/run
              name: var-run
              readOnly: true
            - mountPath: /sys
              name: sys
              readOnly: true
            - mountPath: /var/lib/docker
              name: docker
              readOnly: true
            - mountPath: /dev/disk
              name: disk
              readOnly: true
      priorityClassName: system-node-critical
      serviceAccountName: cadvisor
      terminationGracePeriodSeconds: 30
      tolerations:
        - key: node-role.kubernetes.io/controlplane
          value: "true"
          effect: NoSchedule
        - key: node-role.kubernetes.io/etcd
          value: "true"
          effect: NoExecute
      volumes:
        - hostPath:
            path: /
          name: rootfs
        - hostPath:
            path: /var/run
          name: var-run
        - hostPath:
            path: /sys
          name: sys
        - hostPath:
            path: /var/lib/docker
          name: docker
        - hostPath:
            path: /dev/disk
          name: disk
---
apiVersion: v1
kind: Service
metadata:
  name: cadvisor
  labels:
    app: cadvisor
  namespace: kube-system
spec:
  selector:
    app: cadvisor
  ports:
    - name: cadvisor
      port: 8080
      protocol: TCP
      targetPort: 8080
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app: cadvisor
    release: prometheus-stack
  name: cadvisor
  namespace: kube-system
spec:
  endpoints:
    - metricRelabelings:
        - sourceLabels:
            - container_label_io_kubernetes_pod_name
          targetLabel: pod
        - sourceLabels:
            - container_label_io_kubernetes_container_name
          targetLabel: container
        - sourceLabels:
            - container_label_io_kubernetes_pod_namespace
          targetLabel: namespace
        - action: labeldrop
          regex: container_label_io_kubernetes_pod_name
        - action: labeldrop
          regex: container_label_io_kubernetes_container_name
        - action: labeldrop
          regex: container_label_io_kubernetes_pod_namespace
      port: cadvisor
      relabelings:
        - sourceLabels:
            - __meta_kubernetes_pod_node_name
          targetLabel: node
        - sourceLabels:
            - __metrics_path__
          targetLabel: metrics_path
          replacement: /metrics/cadvisor
        - sourceLabels:
            - job
          targetLabel: job
          replacement: kubelet
  namespaceSelector:
    matchNames:
      - kube-system
  selector:
    matchLabels:
      app: cadvisor
```

ref：https://github.com/rancher/rancher/issues/38934#issuecomment-1294585708

執行上列設定後，cAdvisor 就能替我們開始收集指標：

```jsx
kubectl apply -f cAdvisor.yaml
------
serviceaccount/cadvisor created
clusterrole.rbac.authorization.k8s.io/cadvisor created
clusterrolebinding.rbac.authorization.k8s.io/cadvisor created
daemonset.apps/cadvisor created
service/cadvisor created
servicemonitor.monitoring.coreos.com/cadvisor created
```

如此一來圖表可以正常顯示：

![https://ithelp.ithome.com.tw/upload/images/20230930/20149562E6ldxTNlNr.png](https://ithelp.ithome.com.tw/upload/images/20230930/20149562E6ldxTNlNr.png)

而消失的 image、container 指標也出現了：

![https://ithelp.ithome.com.tw/upload/images/20230930/20149562yrfUj9qLd5.png](https://ithelp.ithome.com.tw/upload/images/20230930/20149562yrfUj9qLd5.png)

# 結論

堅持到現在的各位，相信都深切體會到監控領域的複雜度。換句話說，當我們想要全面監控某個服務時，除了需要深入了解所使用的監控工具，更關鍵的是對監控的目標有足夠的認知。只有如此，我們才能精確地維護和提升服務的品質。透過不同的問題解決經驗，我們對這領域才會有更深入的認識。

在接下來的章節，我們將繼續探索 Grafana 在可觀測性宇宙中的角色和重要性，並從中吸取實戰的教訓，努力構建出屬於自己的可觀測性宇宙。