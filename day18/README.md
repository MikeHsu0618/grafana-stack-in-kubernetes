# 可觀測性宇宙的第十八天 - Grafana Cloud - Kubernetes Monitoring 實戰介紹

# 概述

還記得在之前的介紹中，我們提到 Grafana 團隊，推出了自家的 SaaS 產品「Grafana Cloud」，Grafana Cloud 是 Grafana 團隊提供的一個完整的全託管服務，它整合了 Grafana、Prometheus、Loki 和其他工具，我們只要按照步驟安裝必須的 Agent 採集我們本地資料，就能夠輕鬆地監控和觀察他們的基礎設施、應用程序以及其他系統，非常適合早期團隊以及小型用戶群。

而在這邊想提到的是，在 Kubernetes 中學習 Grafana Stack 以及維護其整個基礎設施架構，本身就是一件非常不容易的任務，但不代表這樣我們就必須擁有一支專業的維運團隊，才能擁有良好的監控服務品質，而 Grafana Cloud 不只為我們提供 Grafana 相關服務的全託管服務，他們還為了 Kubernetes 叢集打造出被名為「Kubernetes Monitoring」的應用區塊，來降低我們對 Kubernetes 可觀測性的門檻以及成本，同時我們也可以透過觀察，了解官方對於可觀測性的最佳實踐，現在就讓我們一探 Grafana 精心替我們準備的好菜吧。

![https://ithelp.ithome.com.tw/upload/images/20231003/20149562GVAvQwzbg7.png](https://ithelp.ithome.com.tw/upload/images/20231003/20149562GVAvQwzbg7.png)

# Kubernetes Monitoring in Grafana Cloud

很顯然，我們會基於非常多種原因以及很多動機需要監控 Kubernetesh 叢集，但手動完成這件事需要對 Kubernetes 生態以及監控領域一定水準的了解，獲得這些好處可能成本會相當沈重。隨著時間，他還會增加了出錯的可能性，從而在不知不覺中使問題無法掌控。

這就是為何 Grafana Cloud 在 2022 年正式引入 Kubernetes Monitoring 功能區塊，它可以為您處理許多手動困難，以便您可以專注於充分利用 Kubernetes 監控。無論你是 SRE、應用程序開發人員還是 DevOps 管理員，Grafana Cloud 中的 Kubernetes 監控都可以讓您更輕鬆地排查事件、推出更改和維護服務器。

Grafana Cloud 用戶只需將 Grafana Agent 安裝到自己的 Kubernetes 集群上，幾分鐘之內，各種指標就會發送到 Prometheus 和 Grafana。從那裡，Grafana Cloud 用戶可以開箱即用地訪問其 Kubernetes 基礎設施的指標、日誌和 Kubernetes 事件以及預構建的儀表板和警報。

而適合使用 Grafana Cloud 的原因除了其完整便捷的使用門檻外，最大的因素在於 Grafana Cloud 提供一定額度的免費方案，足以讓早期團隊或個人開發者使用。

就來看看他們承諾的永久免費計畫及付費方案：

![https://ithelp.ithome.com.tw/upload/images/20231003/201495622Ckwh4JyR0.png](https://ithelp.ithome.com.tw/upload/images/20231003/201495622Ckwh4JyR0.png)

> 有興趣的同學也可以到 Grafana 官方 playground 來體驗 Kubernetes Monitoring 帶來的體驗：https://play.grafana.org/a/grafana-k8s-app/home。
>

# 使用 Kubernetes Monitoring 的理由

如果 Grafana Cloud 的 Kubernetes Monitoring 只是幫我們建出跟開源版本一模一樣的監控系統，我想 Grafana 團隊不會有今天這樣的成就，也確實在 Kubernetes Monitoring 介面中，為我們帶來了前所未有的方便，就讓我們一起來看看他的厲害之處有哪些吧。

## Cluster Navigation

![https://ithelp.ithome.com.tw/upload/images/20231003/20149562A9ZN1u84OV.png](https://ithelp.ithome.com.tw/upload/images/20231003/20149562A9ZN1u84OV.png)

Cluster Navigation 是一個用於可視化各個級別的 Kubernetes 對象的全新界面，使用者可以非常直覺的管理多個叢集，並且顆粒等級由叢集到最細的容器，可以沿著麵包屑層層延伸，可以快速跳轉到特定命名空間以查看正在運行的 Kubernetes Pod 數量、實時觀察部署情況或縮小 Pod 處於不良狀態的任何服務範圍，並且全面整合了各個服務使用狀況及日誌，使得管理及維護人員的時間成本大幅降低。

![https://ithelp.ithome.com.tw/upload/images/20231003/201495629AzERhEHy4.png](https://ithelp.ithome.com.tw/upload/images/20231003/201495629AzERhEHy4.png)

## ****Cost Analysis****

我們可以點選 Cost 區塊來幫助我們了解 Kubernetes 基礎設施消耗的資源成本，並確定潛在節省的領域。

此圖表中有兩個選項卡顯示基礎設施成本：

- Overview：按不同雲服務提供商劃分的成本。
- Savings：按每種資源細分的成本，以及與未使用資源相關的成本。

Grafana Cloud 中的 Cost 功能來自於開源專案 OpenCost ，一個專注於 Kubernetes 雲平台成本可視化的專案，也是許多雲端平台文件中，主要推薦的成本輔助工具。

![https://ithelp.ithome.com.tw/upload/images/20231003/20149562ZiPbcVmatC.png](https://ithelp.ithome.com.tw/upload/images/20231003/20149562ZiPbcVmatC.png)

## ****Prebuilt Grafana dashboards for Kubernetes Monitoring****

除了 Cluster Navigation 之外，安裝 Grafana Agent 還會為我們提供多個預構建的 Kubernetes 儀表板和警報，以監控集群、命名空間、工作負載和 Pod 級別的 CPU 使用情況。因此，Grafana Cloud 用戶可以立即開始在儀表板中跟踪使用的資源、運行的 Pod 和存儲操作。

![https://ithelp.ithome.com.tw/upload/images/20231003/20149562mEV50nEUm7.png](https://ithelp.ithome.com.tw/upload/images/20231003/20149562mEV50nEUm7.png)

## ****Machine learning tools****

CPU 和內存預測可以幫助您確保資源在資源使用高峰期間可用，並幫助您減少由於過度配置而導致的未使用資源量。

Grafana 的機器學習功能為 Grafana Cloud 用戶帶來了預測他們系統的當前和未來狀態的能力。要進行預測，首先需要確定源查詢，即要進行建模的時間序列，並配置機器學習模型。此後，系統將開始在 Grafana Cloud 後端進行模型的訓練。當模型訓練完成後，您可以透過發送查詢來預測該時間序列在未來某段時間內的可能值。此外，該模型還會提供預測值的區間。

值得注意的是，這個模型是自適應的。隨著新的數據和模式的出現，它將持續進行學習，確保其預測保持最新且與您的數據同步。

![https://ithelp.ithome.com.tw/upload/images/20231003/20149562vZ4p4eK7O5.png](https://ithelp.ithome.com.tw/upload/images/20231003/20149562vZ4p4eK7O5.png)

# Kubernetes Monitoring 實戰

現在，就讓我們來體驗看看 Grafana Cloud 號稱數分鐘內就可以搭建起的完整 Kubernetes 監控系統吧

## 註冊帳號並登入 Grafana Cloud

Grafana Cloud 提供完全不需要信用卡的免費會員使用，可以在輸入基本資料後，轉眼間就正式進入個人的 Grafana Cloud 首頁，迎接你的將會是目前方案的使用額度狀況，讓每個人都能第一時間注意到自己的使用量，以免很多人都經歷過的天價賬單悲劇不斷重演。

![https://ithelp.ithome.com.tw/upload/images/20231003/20149562ByhHQlRSRi.png](https://ithelp.ithome.com.tw/upload/images/20231003/20149562ByhHQlRSRi.png)

## 安裝 Kubernetes Monitorinig 專用 Grafana Agent

![https://ithelp.ithome.com.tw/upload/images/20231003/20149562Dqymi3dwdI.png](https://ithelp.ithome.com.tw/upload/images/20231003/20149562Dqymi3dwdI.png)

沒錯，我們一登入進來馬上進可以進入安裝環節，而仔細一看官方提供給我們的安裝五步驟，已經可以說是保母級別的安裝教學了，只要確定你有基本的安裝工具並且命名叢集好後，產生一個 Token 後，就可以直接複製第四步驟的內容貼進指令列中執行。

### 安裝流程

大致流程如下：

1. 準備好 kubectl、helm 安裝必備指令。
2. 選用啟用功能並指定相關資源名稱。
3. 產生或使用已存在的 Access Policy Token。
4. 將下列 Grafana Agent 客製 Helm 安裝指令內容複製並部署到自己的叢集上。
5. 最後，就可以在自己的 Kubernetes Monitoring 區塊上看見自己叢集的相關資訊了。

在第四步中，我們可以得到如下的部署指令及參數，接著我們只要放進指令列執行安裝即可：

```jsx
helm repo add grafana https://grafana.github.io/helm-charts &&
  helm repo update &&
  helm upgrade --install grafana-k8s-monitoring grafana/k8s-monitoring \
    --namespace "k8s-monitoring" --create-namespace --values - <<EOF
cluster:
  name: "docker-desktop"

externalServices:
  prometheus:
    host: "https://prometheus-****.grafana.net"
    basicAuth:
      username: "1128739"
      password: "****"

  loki:
    host: "https://logs-prod-011.grafana.net"
    basicAuth:
      username: "663469"
      password: "****"

opencost:
  opencost:
    exporter:
      defaultClusterId: "docker-desktop"
    prometheus:
      external:
        url: "https://prometheus-****.grafana.net/api/prom"
EOF
```

如果出現以下類似錯誤，代表所安裝的 Prometheus CRDs 有設定上的衝突，建議可以先把 Prometheus CRDs 先移除。

```jsx
Error: rendered manifests contain a resource that already exists. Unable to continue with install: CustomResourceDefinition "alertmanagers.monitoring.coreos.com" in namespace "" exists and cannot be imported into the current release: invalid ownership metadata; label validation error: missing key "app.kubernetes.io/managed-by": must be set to "Helm"; annotation validation error: missing key "meta.helm.sh/release-name": must be set to "grafana-k8s-monitoring"; annotation validation error: missing key "meta.helm.sh/release-namespace": must be set to "k8s-monitoring"
```

順利的話，就可以看到如下的安裝成功畫面。

```jsx
Update Complete. ⎈Happy Helming!⎈
Release "grafana-k8s-monitoring" does not exist. Installing it now.
NAME: grafana-k8s-monitoring
LAST DEPLOYED: Tue Sep 12 01:34:20 2023
NAMESPACE: k8s-monitoring
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

最後，我們就可以順利查看 Grafana Cloud Kubernetes Monitoring 上的自己本地叢集資料。

![https://ithelp.ithome.com.tw/upload/images/20231003/20149562gdUEFEPCIO.png](https://ithelp.ithome.com.tw/upload/images/20231003/20149562gdUEFEPCIO.png)

果真是分鐘級的開箱及用啊！

# 結論

隨著對 Grafana 逐漸了解，越能體會套 Grafana Cloud 對自家產品的高度整合，絕對不是開源專案版本可以輕鬆跟上的，其強大和靈活的監控可視化能力，不僅簡化了設定過程，還提供大量了預設介面，使得每個團隊都能在短時間內快速上手，開始實現監控的最大價值，另一個特別讓我印象深刻的功能是 Grafana Cloud 的機器學習預測。這項功能為我提供了未來數據的洞察，幫助我們更好地理解和預測系統行為，這在預防可能的問題或故障時非常有價值。此外，Grafana Cloud 的外部整合能力也相當出色。不僅支持多種數據源，還能輕鬆與其他工具和平台進行整合，使得數據的匯聚和可視化過程更加無縫。

總結來說，Grafana Cloud 提供了一個高效、全面且易於使用的可觀測性解決方案，不論我們規模龐大服務複雜的大型企業，或是步調快速的新創團隊，都能在 Grafana Cloud 身上找到需要的理由。

---

References

[Monitor the past, present, and future of your Kubernetes resource utilization | Grafana Labs](https://grafana.com/blog/2023/07/12/monitor-the-past-present-and-future-of-your-kubernetes-resource-utilization/)

[Rein in spending with Kubernetes cost monitoring in Grafana Cloud | Grafana Labs](https://grafana.com/blog/2023/06/26/rein-in-spending-with-kubernetes-cost-monitoring-in-grafana-cloud/)

[How to monitor Kubernetes nodes in Grafana Cloud](https://grafana.com/blog/2022/10/25/how-to-monitor-the-health-and-resource-usage-of-kubernetes-nodes-in-grafana-cloud/)

[5 key benefits of Kubernetes monitoring](https://grafana.com/blog/2022/10/13/5-key-benefits-of-kubernetes-monitoring/)

[Kubernetes Monitoring in Grafana Cloud: Prebuilt Grafana dashboards, preconfigured Kubernetes metrics and alerts, and more](https://grafana.com/blog/2022/07/13/introducing-kubernetes-monitoring-in-grafana-cloud/)

[Introduction to Grafana Kubernetes Monitoring |  Grafana Cloud documentation](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/kubernetes-monitoring/about-k8s-monitoring/)