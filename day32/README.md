# 可觀測性宇宙的第三十一天 - Grafana Tempo 搭配 Odigos 實現 NoCode Observability

![https://ithelp.ithome.com.tw/upload/images/20231107/20149562GLzxU0eBWR.png](https://ithelp.ithome.com.tw/upload/images/20231107/20149562GLzxU0eBWR.png)

# 概述

隨著對可擴展應用程式的需求不斷增長，Kubernetes 成為管理容器化工作負載和服務的標準。它使得在分散式實例上部署和運行應用程式變得容易，但監控基礎架構可能具有挑戰性。而 Keyval 在 2023 年 11 月 6 日至 10 日在芝加哥舉行的 KubeCon 上亮相登場，並且正式釋出 v1.0.0 版本，NoCode Observability 的字眼才逐漸出現在世人的視線之中。尤其在 Tracing 方面，因為入侵性的 SDK 語法植入以及語言特性差異，即使 Opentelemetry 生態在實務上以趨近完善，但仍然沒有一個好方法可以作為無痛導入分散式追蹤的解決方案，直到開始有像是 Keyval Odigos 的服務，利用了 OpenTelemetry 搭配 eBPF，建構出也許是下個可觀測性遙測資料收集的解決方案 — NoCode Observability Plateform。

## Keyval 介紹

Keyval 在 2023 年的表現相當亮眼，不僅僅在著名的創業加速器 Y Combinator (YC) 中，於今年初 W23 冬季批次選中獲得種子投資，並且於今年北美的 KubeCon 展示其主力產品 Odigos 的強大之處。

Keyval 的 Odigos 利用 eBPF 技術創建一個開源解決方案，可以自動為任何應用程式產生分散式跟踪，而無需任何工程工作，Odigos 會自動偵測叢集中每個應用程式的程式語言，並相應地執行自動偵測。對於編譯語言（如 Go），使用 eBPF 來偵測應用程式。對於虛擬機器語言（如 Java），使用 OpenTelemetry。此外，Odigos 建立的管道遵循最佳實踐，例如：將 API 金鑰持久保存為 Kubernetes 機密、使用最少的收集器映像等等。

## Odigos 介紹

Odigos 是一個全面自動化的 NoCode Observability Plateform，旨在簡化應用程式在全球節點上運行時的監控和管理。它支援多種語言和框架，如 Java、 Python、 .NET、 Node.js 和 Go，易於與 Kubernetes 和其他工具集成。Odigos 透過自動偵測應用程式語言和使用開源標準如 OpenTelemetry 和 eBPF，減少了學習和維護的負擔。此外，它避免了專有格式和供應商鎖定的問題，提供了一個開放、兼容的解決方案，使得可觀測性變得簡單，適合各種規模的業務場景使用。

# 實戰演練

分散式追蹤可能會顛覆我們監控和調試複雜系統的方法。它不同於在單一應用程式中捕捉時間點的指標或日誌，分散式追蹤通過賦予請求唯一的 ID，來追蹤其在分散式系統中的流動。這讓開發者能夠理解每個請求的上下文及其分散式應用程式的運作方式。

然而，分散式追蹤不同於指標或日誌較為困難。分散式追蹤需要在多個應用程式中實施才能發揮價值。如果系統中的任何一個應用程式未能產生分散式追蹤資料，那麼請求的上下文傳播將被中斷，追蹤的價值也會大幅降低，因為無法顯示請求的完整路徑。這可能使得確定問題的根源變得更加困難，而這一點在指標和日誌中通常可以通過利用現有的基礎架構或日誌框架來自動實現。

不過我們現在有了 Keyval Odigos，實現了讓我們看見在十分鐘內迅速、無痛、完整地建立大規模分散式追蹤的可能性，現在就讓我們趕快來實際體驗看看。

## Kind 安裝（MacOS）

首先我們在本地操作時，需要建立本地 Kubernetes 叢集，並且 Odigos 目前沒有支援我們先前熟悉的 Kubernetes in Docker-desktop，官方推薦的則是 kind。

```jsx
brew install kind
```

![https://ithelp.ithome.com.tw/upload/images/20231107/20149562mWjSnHC5FC.png](https://ithelp.ithome.com.tw/upload/images/20231107/20149562mWjSnHC5FC.png)

接下來就讓我們建立起 Kind 本地叢集：

```jsx
$ kind create cluster
Creating cluster "kind" ...
⢀⡱ Ensuring node image (kindest/node:v1.27.3) 🖼 
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Thanks for using kind! 😊
```

## Kube-Prometheus-Stack 安裝

在接下來的實戰中，我們將利用先前建立好的 Kube-Prometheus-Stack 及其相關設定來當作 Grafana Tempo 的可視化介面，其中詳細安裝過程可以回頭複習一下「Kube-Prometheus-Stack 實戰系列」。

```jsx
$ helm upgrade --install prometheus-stack prometheus-community/kube-prometheus-stack --values=values.yaml -n prometheus --create-namespace
Release "prometheus-stack" does not exist. Installing it now.
NAME: prometheus-stack
LAST DEPLOYED: Tue Nov  7 00:19:09 2023
NAMESPACE: prometheus
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace prometheus get pods -l "release=prometheus-stack"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

> 相關設定檔同步收錄在文章底部的 Github 連結。
>

## Grafana Tempo 安裝

同樣的，我們也將利用 Grafana Tempo 當作分散式追蹤的後端儲存服務，詳細安裝過程可以複習「Grafana Tempo 實戰系列」。

```jsx
$ helm upgrade --install tempo  grafana/tempo-distributed -n tracing -f values.yaml --create-namespace
Release "tempo" does not exist. Installing it now.
NAME: tempo
LAST DEPLOYED: Tue Nov  7 00:22:23 2023
NAMESPACE: tracing
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
***********************************************************************
 Welcome to Grafana Tempo
 Chart version: 1.6.9
 Tempo version: 2.2.3
***********************************************************************

Installed components:
* ingester
* distributor
* querier
* query-frontend
* compactor
* memcached
* gateway
```

> 相關設定檔同步收錄在文章底部的 Github 連結。
>

## Odigos 安裝

接下來我們就可以透過本地 Odigos Cli 工具來執行 Odigos 相關操作：

```jsx
$ brew install keyval-dev/homebrew-odigos-cli/odigos
```

接下來，建立 Odigos 相關 Kubernetes Resources：

```jsx
$ odigos install
Installing Odigos version v1.0.0 in namespace odigos-system ...
Creating namespace odigos-system                  ✔
Creating CRDs                                     ✔
Creating Odigos OdigosDeployment                  ✔
Creating Odigos OdigosConfig                      ✔
Creating Odigos OwnTelemetry Pipeline             ✔
Creating Odigos DataCollection                    ✔
Creating Odigos Instrumentor                      ✔
Creating Odigos Scheduler                         ✔
Creating Odigos Odiglet                           ✔
Creating Odigos AutoScaler                        ✔
Waiting for Odigos pods to be ready ...           ✔

SUCCESS: Odigos installed.
```

## 建立 Demo 微服務：

這邊我們使用 Odigos 為我們提供的微服務範例，來作為我們 Traces 訊號的來源：

```jsx
$ kubectl apply -f https://raw.githubusercontent.com/keyval-dev/microservices-demo/master/release/kubernetes-manifests.yaml
deployment.apps/adservice created
service/adservice created
deployment.apps/cartservice created
service/cartservice created
deployment.apps/checkoutservice created
service/checkoutservice created
deployment.apps/currencyservice created
service/currencyservice created
deployment.apps/emailservice created
service/emailservice created
deployment.apps/frontend created
service/frontend created
service/frontend-external created
deployment.apps/paymentservice created
service/paymentservice created
deployment.apps/productcatalogservice created
service/productcatalogservice created
deployment.apps/recommendationservice created
service/recommendationservice created
deployment.apps/redis-cart created
service/redis-cart created
deployment.apps/shippingservice created
service/shippingservice created
deployment.apps/loadgenerator created
```

## 實戰演練 Odigos NoCode Observability Plateform

使用 Odigos 暴露出 Odigos UI 端口到本地：

```jsx
$ odigos ui
2023/11/07 00:27:50 Starting Odigos UI...
2023/11/07 00:27:50 Odigos UI is available at: http://localhost:3000
```

接下來我們就可以在 [localhost:3000](http://localhost:3000)  上看到 Odigos 精美簡潔的串接介面，就讓我們開始神奇的 NoCode 之旅。

### 選擇 Application Source

![https://ithelp.ithome.com.tw/upload/images/20231107/20149562gA0ZMb1MuC.png](https://ithelp.ithome.com.tw/upload/images/20231107/20149562gA0ZMb1MuC.png)

這裡我們將 Odigos 官方提供的微服務 Demo 全選起來，點擊 Next 進入下一頁選擇 Destination。

### 選擇 Destination Backend

![https://ithelp.ithome.com.tw/upload/images/20231107/20149562DXgNaVJcOr.png](https://ithelp.ithome.com.tw/upload/images/20231107/20149562DXgNaVJcOr.png)

在此頁面中，Odigos 到目前為止已經提供了許多 Managed 或 Self-Managed 的遙測訊號儲存後端平台，並且不局限於 Logging、Tracing、Monitoring。

以下則是到目前為止 Odigos 官方支持的完整列表：

![https://ithelp.ithome.com.tw/upload/images/20231107/20149562hXPMls4Het.png](https://ithelp.ithome.com.tw/upload/images/20231107/20149562hXPMls4Het.png)

### 選取 Grafana Tempo 作為 Destination

![https://ithelp.ithome.com.tw/upload/images/20231107/2014956287lfoonlgh.png](https://ithelp.ithome.com.tw/upload/images/20231107/2014956287lfoonlgh.png)

在這邊我們輸入我們先前安裝好的 Grafana Tempo otel 端點 http://tempo-distributor.tracing:4317 後，點擊確認。

### 查看 Observability Pipeline 全域圖

![https://ithelp.ithome.com.tw/upload/images/20231107/20149562g4fQBBh1Ll.png](https://ithelp.ithome.com.tw/upload/images/20231107/20149562g4fQBBh1Ll.png)

現在我們已經可以看到剛剛建立好的微服務 Demo 成功的透過 Odigos 啟動 data-collection 收集 Trace 送到我們指定的 Grafana Tempo 儲存後端。並且 Odigos 在 v0.1.52 版本中的 data-collection 成功捨棄令人詬病的 Sidecar 注入模式，轉而實現出使用守護進程 Daemonset 的工作負載，實現對 Logs、Traces、Metrics 的收集，相信這個模式很大機率將能成為未來可觀測性遙測資料收集的改進方向。

### 查看 Pipeline 中的 Application Source

![https://ithelp.ithome.com.tw/upload/images/20231107/20149562tC4rApHyCI.png](https://ithelp.ithome.com.tw/upload/images/20231107/20149562tC4rApHyCI.png)

可以看到 Odigos 成功判斷出每一個微服務的程式語言，並且在 Kubernetes 叢集中，建立起了對應的 Instrumented Application CRD 資源，如下的 Golang 範例：

```jsx
apiVersion: odigos.io/v1alpha1
kind: InstrumentedApplication
metadata:
  creationTimestamp: '2023-11-06T16:58:22Z'
  generation: 1
  name: deployment-frontend
  namespace: default
spec:
  languages:
    - containerName: server
      language: go
      processName: /frontend/server
```

## 在 Grafana Explore 中實際體驗 Grafana Tempo 搭配 Odigos

首先讓我們將 Grafana 端點導出來：

```jsx
kubectl port-forward service/prometheus-stack-grafana 3100:80 -n prometheus
------
Forwarding from 127.0.0.1:3100 -> 3100
Forwarding from [::1]:3100 -> 3100
```

接著進入我們本機中的 [localhost:3100](http://localhost:3000) 就能看到精美的 Grafana Login 頁面。

### 新增 Tempo 為新的 Data Source

現在我們就在 Data Sources 中將我們的 tempo-gateway 或 tempo-query-frontend 查詢組件的端點新增到 Grafana 上。

![https://ithelp.ithome.com.tw/upload/images/20231107/20149562VxBsMriZbE.png](https://ithelp.ithome.com.tw/upload/images/20231107/20149562VxBsMriZbE.png)

### 使用 Grafana Tempo 查看 Traces

![https://ithelp.ithome.com.tw/upload/images/20231107/20149562BDTzQo0niv.png](https://ithelp.ithome.com.tw/upload/images/20231107/20149562BDTzQo0niv.png)

到此，我們只花費了僅僅不到十分鐘，就已經成功的串接起五種不同程式語言的分散式追蹤資料，並且沒有修改任何一點 Source Code，這在發展十幾年的分散式追蹤領域是非常驚人的突破。

### 開啟 Grafana Tempo 查看 Service Graph / Node Graph

使先我們需意在 Grafana Data Source 頁面中，將 Tempo 的進階相關設定如下開啟：

![https://ithelp.ithome.com.tw/upload/images/20231107/20149562POE0MhFI4u.png](https://ithelp.ithome.com.tw/upload/images/20231107/20149562POE0MhFI4u.png)

### 查看 Service Graph

![https://ithelp.ithome.com.tw/upload/images/20231107/20149562X7R0qR9qaJ.png](https://ithelp.ithome.com.tw/upload/images/20231107/20149562X7R0qR9qaJ.png)

### 查看 Node Graph

![https://ithelp.ithome.com.tw/upload/images/20231107/201495629gfsYFTmbh.png](https://ithelp.ithome.com.tw/upload/images/20231107/201495629gfsYFTmbh.png)

到此我們就已經大功告成！

## 移除 Kind 本地叢集

```jsx
$ kind delete cluster
```

# 結論

Odigos 提出的 NoCode Observability Plateform 概念，無疑在可觀測性世界以及 Opentelemetry 中都引起不小的關注，其中最大的關鍵在於 ePBF 技術在近期擁有突破性的發展，才解決 Golang 這個熱門語言長期沒辦法擁有一個好的分散式追蹤訊號收集方案的大問題。而從 Odigos 的微服務可觀測性 Demo 中，我們真的可以看到，竟然有一項服務可以在短短十分鐘內，使用完全不入侵程式碼的方式收集到十來種微服務的 Traces，這是我在此之前不敢想像的畫面。我們可以預見未來，原本需要相對高門檻、多人合作實現的可觀測性領域，已經逐漸降低入門的最低要求，更重要的是它使我們能為組織釋放更多人力資源、專注於提高生產力，而不是顧忌各種語言的可觀測性需求與維護，增加工作人員的心理負擔！

---

相關程式碼同步收錄在：

[https://github.com/MikeHsu0618/grafana-stack-in-kubernetes/tree/main/day31](https://github.com/MikeHsu0618/grafana-stack-in-kubernetes/tree/main/day31)

Referrences：

[Instant Distributed Traces with Odigos](https://www.youtube.com/watch?v=nynyV7FC4VI)

[keyval: Open-source codeless monitoring pipeline | Y Combinator](https://www.ycombinator.com/companies/keyval)

[Meet the YC Winter 2023 Batch | Y Combinator](https://www.ycombinator.com/blog/meet-the-yc-winter-2023-batch)

[Odigos零侵入原理分析](https://www.jianshu.com/p/9fcf16308e6c)

[Odigos: 一款助你在 Kubernetes 上快速构建端到端无侵入的可观测解决方案-CSDN博客](https://blog.csdn.net/easylife206/article/details/130652425)