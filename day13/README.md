# 可觀測性宇宙的第十三天 - Kube-Prometheus-Stack 實戰（一）

# 概述

在本章節，我們將深入探索 Prometheus 全家桶。透過細緻拆解每一個元件，從最基礎的設定出發，逐步理解 Kube-Prometheus-Stack 的四千行左右 Helm Values 參數，以及超過五十個的 Kubernetes 資源設定檔，背後的學問深不見底。我們將運用工程師的逆向思考技巧，站在巨人的肩膀上，俯視整個監控世界。

過程中，我們也會遭遇到由於 Kubernetes 的版本差異或特定的本地環境因素所帶來的挑戰。但不再是一味地遵循簡單的安裝指南，而是希望當我們面對眼前各式各樣的問題時，都能夠有所對策，自信地迎接每一個挑戰。

# Kube-Prometheus-Stack 安裝

我們將使用 Artifact hub 中的 [kube-prometheus-stack v50.0.0](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack/50.0.0) 版本，來完成各種安裝及學習的操作。

## Helm 安裝 (MacOs)

首先讓我們依照前面的章節安裝 Helm

```jsx
brew install helm
```

接著查看一下安裝好的 Helm 版本：

```jsx
helm version
------
version.BuildInfo{Version:"v3.9.0", GitCommit:"7ceeda6c585217a19a1131663d8cd1f7d641b2a7", GitTreeState:"clean", GoVersion:"go1.18.2"}
```

基本上只要是 Helm v3 之後的版本，操作上都不會遇到太的的問題。

## 使用 Helm Values 安裝 Kube-Prometheus-Stack

現在我們將要使用 Helm Values 文件裡面的定義好的參數，來為我們啟動整套完整的 Prometheus 監控系統。

```jsx
# values.yaml
fullnameOverride: "prometheus-stack"

kubeScheduler:
  enabled: true

kubeControllerManager:
  enabled: true

kubeEtcd:
  enabled: true

kubeProxy:
  enabled: true

kubeApiServer:
  enabled: true

kube-state-metrics:
  prometheus:
    monitor:
      enabled: true
  selfMonitor:
    enabled: true

kubelet:
  enabled: true
  serviceMonitor:
    cAdvisor: true

nodeExporter:
  enabled: true
  hostRootfs: true

prometheus-node-exporter:
  hostRootFsMount:
    enabled: true

alertmanager:
  enabled: true

grafana:
  enabled: true

  adminUser: admin
  adminPassword: admin

  defaultDashboardsEnabled: true
  defaultDashboardsTimezone: Asia/Taipei

  grafana.ini:
    users:
      viewers_can_edit: true

  persistence:
    accessModes:
      - ReadWriteOnce
    enabled: true
    size: 10Gi
    type: pvc

prometheusOperator:
  enabled: true

crds:
  enabled: true

prometheus:
  enabled: true
  
  agentMode: false
  prometheusSpec:
    enableFeatures: []
    
    enableRemoteWriteReceiver: false
    remoteWriteDashboards: false
    
    retention: 10d
    
    remoteWrite: []
    
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: hostpath
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi
    
    additionalScrapeConfigs: []
```

Kube-Prometheus-Stack 社群巧妙地將繁瑣的監控設定精煉成約四千行的 Helm Values 參數檔，讓使用者在面對不同的實務需求時，能夠進行靈活而細緻的配置調整。而我在此分享的不超過一百行的參數設定，基於個人經驗，認為它足以應對大部分初學者接觸 Prometheus 的場景，且不會帶來過大的學習負擔。

接下來，我們會逐項深入每個主要部分，藉由這批經過精心挑選的設定參數，藉由逆向學習，進一步探索 Kubernetes 中的進階理念。

### Kube-Prometheus-Stack 安裝

```jsx
helm upgrade --install prometheus-stack prometheus-community/kube-prometheus-stack --values=values.yaml -n prometheus --create-namespace
------
Release "prometheus-stack" does not exist. Installing it now.y/kube-prometheus-stack --values=values.yaml -n prometheus --create-namespace
NAME: prometheus-stack
LAST DEPLOYED: Thu Sep  7 00:43:12 2023
NAMESPACE: prometheus
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace prometheus get pods -l "release=prometheus-stack"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

以上指令將會部署整套 release name 為 prometheus-stack 的 kube-prometheus-stack Helm Chart 資源在 prometheus 命名空間中。

接著就可以馬上來查看這些資源：

```jsx
kubectl get pods -n prometheus
-------
NAME                                                       READY   STATUS    RESTARTS   AGE
pod/alertmanager-prometheus-stack-alertmanager-0           2/2     Running   0          16m
pod/prometheus-prometheus-stack-prometheus-0               2/2     Running   0          16m
pod/prometheus-stack-grafana-78578cb6cf-8dfhz              3/3     Running   0          16m
pod/prometheus-stack-kube-state-metrics-7bff4cbdf7-glqfl   1/1     Running   0          16m
pod/prometheus-stack-operator-6886bcf47b-qz4t6             1/1     Running   0          16m
pod/prometheus-stack-prometheus-node-exporter-wdznq        1/1     Running   0          16m
```

果真 Kube-Prometheus-Stack 在短短幾分鐘內，就替我們建立了整套監控系統，就讓我們繼續體驗一下這套全家桶的厲害之處。

### Kube-Prometheus-Stack - Prometheus

讓我們將 Grafana 端點導出來：

```jsx
kubectl port-forward service/prometheus-stack-prometheus 9090:9090 -n prometheus
------
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
```

接著進入我們本機中的 localhost:9090 就能看到 Prometheus 的 Web UI 頁面。

![https://ithelp.ithome.com.tw/upload/images/20230928/20149562WyUOFPrgIw.png](https://ithelp.ithome.com.tw/upload/images/20230928/20149562WyUOFPrgIw.png)

讓我們繼續看看其他 Prometheus 架構下的功能頁面。

![https://ithelp.ithome.com.tw/upload/images/20230928/20149562ClTgbNFPbq.png](https://ithelp.ithome.com.tw/upload/images/20230928/20149562ClTgbNFPbq.png)

Kube-Prometheus-Stack 為我們準備了數百條 Prometheus Rules，細膩涵蓋了 Kubernetes 基本告警場景，並且預設就和 Alertmanager 完整串接。

![https://ithelp.ithome.com.tw/upload/images/20230928/20149562St2BHghVih.png](https://ithelp.ithome.com.tw/upload/images/20230928/20149562St2BHghVih.png)

預先建立了各種 ServiceMonitor 資源來對整個 Kubernetes 叢集抓取必要的資源指標，提供完整全面的監控數據來源。

### Kube-Prometheus-Stack - Grafana

讓我們將 Grafana 端點導出來：

```jsx
kubectl port-forward service/prometheus-stack-grafana 3000:80 -n prometheus
------
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```

接著進入我們本機中的 [localhost:3000](http://localhost:3000) 就能看到精美的 Grafana Login 頁面。

![https://ithelp.ithome.com.tw/upload/images/20230928/20149562rKtDqrbi7N.png](https://ithelp.ithome.com.tw/upload/images/20230928/20149562rKtDqrbi7N.png)

輸入我們自訂的帳號密碼後，就可以開啟你的 Grafana 之旅。

> 本章節為教學分享性質，只在本機環境上進行操作，請勿在正式環境使用高風險帳號密碼組合。
>

最重要的是，安裝好的 Grafana 除了預設就已經接上我們的 Prometheus 為資料源，並且已經提供了一套非常完整的 Kubernetes 基本監控圖表。

![https://ithelp.ithome.com.tw/upload/images/20230928/20149562C9VxCm21JO.png](https://ithelp.ithome.com.tw/upload/images/20230928/20149562C9VxCm21JO.png)

### Kube-Prometheus-Stack - AlertManager

讓我們將 Grafana 端點導出來：

```jsx
kubectl port-forward service/prometheus-stack-alertmanager 9093:9093 -n prometheus
------
Forwarding from 127.0.0.1:9093 -> 9093
Forwarding from [::1]:9093 -> 9093
```

接著進入我們本機中的 [localhost:9093](http://localhost:3000) 就能看到 AlertManager 頁面。

![https://ithelp.ithome.com.tw/upload/images/20230928/20149562pL5FQzDGm3.png](https://ithelp.ithome.com.tw/upload/images/20230928/20149562pL5FQzDGm3.png)

AlertManager 的功能頁面就如畫面一樣，非常的精簡，沒有太多精美功能，甚至沒辦法在上面對告警做太多複雜設定，一切只能靠我們在 Kubernetes 設定檔中調整，這也是 Prometheus 官方團隊一直以來的初衷，他必須夠簡單高效、方便部署，而不是成為維運人員另一個負擔。

# 結論

不管部署了 Kube-Prometheus-Stack 幾次，每次在我的心中總是會對其不驚讚嘆，這在短短幾分鐘建立起這等規模的服務，究竟如果使用者從零開始搭建，可能要花費無數的夜晚才可能有穩定成熟的基本功能。但我們換個角度想想，使用這些像是魔法般的工具，就像黑盒子一樣，無法掌握的話終究會被反噬。

因為實際上在使用 Prometheus 會遇到許多挑戰還有坑，實在適合另開一篇好好聊聊，在下個章節中，我們將會解析為什麼我們需要在 Kubenetes 叢集中安裝如此多的套件，是不是全部安裝就沒問題？而眼尖的同學可能也從上列展示的圖片中，看出有些地方好像穿插了一些錯誤訊息。別擔心，這些魔法將會在接下來的章節全部揭曉！