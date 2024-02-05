![Untitled](https://imgur.com/kyGXjcl.png)

# 前言

如同我們先前提到的，市面上存在諸如 Datadog、Sentry、Elastic 等老牌監控 SaaS 服務廠商，他們在前端領域已耕耘許久。然而，現有的前端監控解決方案大多仍屬於閉源，其使用的透明度並不足，往往被視為類似黑盒子的神秘 SaaS 服務。換句話說，前端監控領域仍被視作是監控廠商間的技術軍備競賽。

而 Grafana Faro 的出現，作為一款開源且高度整合社群主流可觀測性技術的工具，為我們理解前端可觀測性提供的全新的視角。當我們褪去 Faro 使用了眾多技術工具如 Grafana Stack、Prometheus、Opentelemetry 等服務的外衣後，深入了解一個可觀測性 Web SDK 涉及的各種資料面向及其整體架構設計。這使我們能夠更全面地理解前端監控的潛力，以及如何利用 Grafana Faro 在前端可觀測性領域實現創新。

# 從 Grafana Faro 洞悉前端可觀測數據

![Untitled](https://i.imgur.com/iY5lIO9.png)

Grafana Faro 在前端數據分析領域中的應用，可謂是數據分析經典流程的完整體現：

- 資料收集
- 資料儲存
- 資料見解

這一過程從使用 Faro Web SDK 開始，該 SDK 專門用於收集前端瀏覽器的「預處理」數據，預處理包括數據清洗、格式化、以及基礎計算，這些都是使數據變得可用和有意義的基礎步驟。而這些數據包含了使用者與網站或應用服務的互動訊息，例如性能指標、使用者行為、日誌以及錯誤報告等。

收集到的數據隨後會被 Grafana Agent 接收，並分送到不同的儲存系統中，主要是 Grafana Loki 和 Grafana Tempo。Grafana Loki 處理日誌數據，而 Grafana Tempo 則專注於追踪數據。這種分類儲存的方法確保了數據的高效管理和查詢。

最後，這些數據會被匯聚到 Grafana Dashboard 上，這裡的數據可視化功能扮演著關鍵角色。通過豐富的可視化工具，Grafana Dashboard 使得複雜的數據集變得直觀，幫助使用者快速識別趨勢和模式。從這些可視化中，我們可以提取出重要的見解，這些見解對於瞭解使用者行為、優化服務效能和改進使用者體驗至關重要。

### **何謂資料可視化**

資料可視化是將複雜的數據集以視覺形式呈現的過程，例如通過圖表、地圖等方式。這種方法有助於快速、直觀地理解和解釋數據。在傳統數據分析中，僅依靠數字和統計資料來獲取結論往往較為困難，因為人腦對於視覺信息的處理能力通常優於對文字的處理。因此，透過視覺化手段，如圖表和圖形，我們可以更容易地識別出數據模式、趨勢、統計規律和相關性。

一個優秀的資料可視化應包括四個要素：

- 簡潔性
- 信息豐富
- 高效率
- 美觀性

![Untitled](https://i.imgur.com/MWWPibe.gif)

例如皮尤研究中心所製作的「[Next America](https://www.pewresearch.org/social-trends/2014/04/10/next-america/)」對美國的人口統計資料進行可視化的成果。

該專案展示了美國日益增長的多樣性和人口老化趨勢等，並基於歷史數據分析未來幾十年的人口變化預測。專案中的一個亮點是利用動畫展示了年齡和性別別的人口細分金字塔，從而直觀地展示了從 1950 年代以來的人口統計變化。

# Grafana Faro 的可視化平台

在深入了解 Grafana Faro 在前端數據分析中的真正價值後，我們認識到，經過有效的資料收集和儲存，關鍵在於如何從這些數據中發掘出關聯性（Correlation）、行為（Behavior）、屬性（Attribute）、狀態（State）和意義（Meaning），以獲得深刻的見解（Insight）。Grafana Faro 通過對這些複雜數據的深入分析，協助我們識別使用者行為的模式，揭露應用的性能瓶頸，並最終提升用戶體驗。此外，Grafana 提供的強大且多樣化的 Dashboard 和 Panel 扮演了關鍵角色，它們不僅串聯了跨領域的數據，還使得數據模式、趨勢、統計規律和相關性的解釋變得更加人性化和易於理解。這一切共同促進了更加高效和精確的數據驅動決策過程。

當我們在使用 Grafana Faro 來分析和展示數據時，我們可以選擇通過自建 Grafana 平台（Grafana OSS）或使用 Grafana Cloud 這樣的 SaaS 服務。通過 Grafana 提供開箱及用的 Dashboard，我們可以逆向從中學習並提取出富有價值的數據見解，幫助我們理解數據的價值。雖然我不是數學家，但這真的很有趣。

## Grafana Cloud - Frontend Monitoring

截至 2022 年，開源數據可視化公司 Grafana Labs 在經歷了數輪融資後，估值已來到接近 60 億美元的身價，總融資額已超過 5 億美元。這些融資包括來自紅杉資本、光速創投、摩根大通等著名投資機構的支持。Grafana Labs 旗下的 Grafana Cloud 是一個受到投資人寄予厚望的商業 SaaS 平台。該平台基於 Grafana 團隊的開源專案開發，不僅為開源社群帶來回饋，也為 Grafana 團隊創造了可持續的收入來源，形成了一種雙贏局面。

![Untitled](https://i.imgur.com/VcuHori.png)

> ref: [equities.fyi](https://equities.fyi/company/grafana-labs)
>

Grafana 團隊在推出新服務時，通常會採用先在 Grafana Cloud 進行公開預覽的策略。這種做法使他們能夠快速獲得用戶的反饋和實際體驗心得，從而對產品進行及時的改進和優化。這種「先上雲端，後開源」的模式，有助於確保新功能在成熟且穩定之後，再被整合回開源專案中。

在 Grafana Cloud 的 Frontend 功能模組底層中，Faro Web SDK 被用作 RUM 資料收集工具，而且透過與 Grafana Scenes 的結合，實現了超越傳統 Grafana Dashboard 的資料呈現方式。

### Overview Page

在應用效能總覽頁面中，我們可以看到以 **Web Vitals** 為主的一系列圖表，並且也能看到每個頁面的載入時間跟錯誤總數。

![Untitled](https://i.imgur.com/Xg55fpk.png)

### Errors Page

此錯誤頁面可以直觀的了解應用服務是否產生大量錯誤，並且能夠根據錯誤進行過濾，藉由追蹤訊號、Session ID 以及各種資訊來提供上下文幫助除錯。

![Untitled](https://i.imgur.com/O1mFtsl.png)

![Untitled](https://i.imgur.com/AzootOl.png)

### Session Page

Fara Web SDK 能夠在前端幫助我們收集真實使用者的歷程已及互動相關事件，並且與其 Session ID 作為關聯，以便更深入的了解錯誤情況及原因。

![Untitled](https://i.imgur.com/O1mFtsl.png)

![Untitled](https://i.imgur.com/hsz630T.png)

> Grafana OSS 提供的 Dashboard 將會在實戰演練中展示。
>

# 實戰演練

現在我們將要實際搭建起整個 Grafana Faro 的資料架構，其中實現了包含 Grafana UI、Loki、Tempo 的深度整合，並搭配 Grafana Agent 與 Fara Web SDK 搭配無間的資料管道，最終使我們能夠有效的捕獲那些在前端看似遙不可及的珍貴數據，從而揭露出更參層次的洞察。

## Kube-Prometheus-Stack 安裝

我們將利用先前建立好的 Kube-Prometheus-Stack 來作為 Grafana 與 Prometheus 的可視化介面，其中詳細安裝過程可以回頭複習一下「Kube-Prometheus-Stack 實戰系列」。

```yaml
❯ helm upgrade --install prometheus-stack prometheus-community/kube-prometheus-stack --values=values.yaml -n prometheus --create-namespace
Release "prometheus-stack" does not exist. Installing it now.y/kube-prometheus-stack --values=values.yaml -n prometheus --create-namespace
NAME: prometheus-stack
LAST DEPLOYED: Thu Dec 21 21:03:15 2023
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

## **Grafana Tempo 安裝**

同樣的，我們也將利用 Grafana Tempo 當作分散式追蹤的後端儲存服務，詳細安裝過程可以複習「Grafana Tempo 實戰系列」。

```yaml
❯ helm upgrade --install tempo  grafana/tempo-distributed -n tracing -f values.yaml --create-namespace
Release "tempo" does not exist. Installing it now.ributed -n tracing -f values.yaml --create-namespace
NAME: tempo
LAST DEPLOYED: Thu Dec 21 21:12:26 2023
NAMESPACE: tracing
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
***********************************************************************
 Welcome to Grafana Tempo
 Chart version: 1.7.3
 Tempo version: 2.3.0
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

## **Grafana Loki 安裝**

同樣的，我們也將利用 Grafana Tempo 當作分散式追蹤的後端儲存服務，詳細安裝過程可以複習「Grafana Loki 實戰系列」。

```yaml
❯ helm upgrade --install loki grafana/loki-distributed -f values.yaml -f config.yaml -n logging --create-namespace
Release "loki" does not exist. Installing it now.buted -f values.yaml -f config.yaml -n logging --create-namespace
NAME: loki
LAST DEPLOYED: Thu Dec 21 21:09:48 2023
NAMESPACE: logging
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
***********************************************************************
 Welcome to Grafana Loki
 Chart version: 0.78.0
 Loki version: 2.9.2
***********************************************************************

Installed components:
* gateway
* ingester
* distributor
* querier
* query-frontend
* query-scheduler
* compactor
```

> 相關設定檔同步收錄在文章底部的 Github 連結。
>

## **Grafana Agent 安裝**

接下來我們就要將重點放在 Grafana Agent 身上，並且搶先使用 Grafana Agent 在 0.37 版本後才對 Flow mode 支援的 **faro.receiver** 功能元件。

```yaml
❯ helm upgrade --install grafana-agent-collector grafana/grafana-agent --values values.yaml -n collector --create-namespace
Release "grafana-agent-collector" does not exist. Installing it now.
NAME: grafana-agent-collector
LAST DEPLOYED: Thu Dec 21 21:14:17 2023
NAMESPACE: collector
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Welcome to Grafana Agent!
```

> 相關設定檔同步收錄在文章底部的 Github 連結。
>

## Grafana Agent Flow mode - Faro 參數設定檔詳解

```jsx
# values.yaml
// ...
agent:
  mode: 'flow'
  extraPorts:
     - name: "faro"
       port: 12347
       targetPort: 12347
       protocol: "TCP"
  configMap:
    create: true
    content: |
      logging {
      	level  = "info"
      	format = "logfmt"
      }
      
      faro.receiver "default" {   
        server {
          cors_allowed_origins = ["*"]
        }
        output {
          logs   = [loki.process.label.receiver]
          traces = [otelcol.exporter.otlp.traces.input]
        }
      }
      
      loki.process "label" {
        stage.logfmt {
          mapping = { 
            "app" = "app_name",
            "version" = "app_version",
            "environment" = "app_environment",
            "kind" = "",
          }
        }  

        stage.labels {
          values = {
            app  = "",         
            version = "",
            environment = "",
            kind = "",
          }
        }

        forward_to = [loki.write.default.receiver]
      }
      
      loki.write "default" {
        endpoint {
          url = "http://loki-gateway.logging/loki/api/v1/push"
        }
        external_labels = {}
      }
      
      otelcol.exporter.otlp "traces" {
        client {
          endpoint = "tempo-distributor.tracing:4317"
          tls {
            insecure             = true
            insecure_skip_verify = true
          }
        }
      }
```

### faro.receiver

- extraPorts：使用 faro.receiver 元件時，Grafana Agent 將會預設使用 port 12347 作為接收端口，需要特別在 Helm Value 中額外添加 extraPorts 相關設定。
- cors_allowed_origins：作為一個公開服務的端口，CORS 設定是關鍵性的安全配置（請勿在正式環境使用 "*" 或過於寬鬆的設定）。
- output：faro.receiver 元件最終會將接收到的資料分送至指定的 Log、Trace 端口，也就是這裡設定的 Loki 和 OTLP(Tempo) 區塊。

### loki.process

- Labels：除了我們需要指定 Loki 的寫入端點之外，我們可以為 Faro Web SDK 傳遞過來的資料作一層 Label Mapping 的預處理，提供我們在查詢或製作 Dashboard 的高度擴充性。

### otelcol.exporter.otlp

- OTLP Exporter：Grafana Agent 選擇 OpenTelemetry 開源統一的 OTLP 協議，避免了 Vendor Locking 的麻煩以及提供高度彈性。

![Untitled](https://i.imgur.com/G5N1BWR.png)

## 體驗前端導入 Faro Web SDK

首先讓我們將 Grafana 端點導出來：

```jsx
kubectl port-forward service/prometheus-stack-grafana 3000:80 -n prometheus
------
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```

而我們也需要注意 Grafana Agent 跟 Faro Web SDK 的網路是否暢通，如範例中則是採用暴露在 [localhost](http://localhost) 互相溝通的方式。

```jsx
kubectl port-forward service/grafana-agent-collector 12347:12347 -n collector
------
Forwarding from 127.0.0.1:12347 -> 12347
Forwarding from [::1]:12347 -> 12347
```

### 埋入 Faro Web SDK 程式碼

只需要簡單的在前端中導入 Faro Web SDK 短短幾行程式碼的設定，即可享受到完成 Grafana Faro 資料流的最後一塊拼圖。

```jsx
var faro = initializeFaro({
    url: 'http://localhost:12347/collect',
    app: {
        name: 'frontend-app',
        version: '1.0.0',
        environment: 'production'
    },
    instrumentations: [
        // Mandatory, overwriting the instrumentations array would cause the default instrumentations to be omitted
        ...getWebInstrumentations(),
        new TracingInstrumentation(),
    ],
});
```

如此一來，當我們的前端在進行各種操作時，就能依照 Faro Web SDK 的設定，傳遞 RUM 數據至我們暴露出的 Grafana Agent http://localhost:12347 端口了。

> Faro Web SDK 可以輕易的使用在任何前端服務，關於使用場景以及實際操作設定有機會將會額外分享。
更多使用範例可以參考 [Faro Web SDK](https://github.com/grafana/faro-web-sdk/blob/main/docs/sources/tutorials/quick-start-browser.md) 官方庫。
>

## Grafana OSS - Frontend Monitoring

在我們自建的 Grafana OSS 中導入官方 Grafana Faro Dashboard 後，可以發現相比於 Grafana Cloud，在 Grafana OSS 上主要提供基於 Grafana Dashboard 搭建的標準圖表，這些是我們熟悉的 Grafana 常規數據展現形式。這些圖表不僅提供了直觀的數據可視化，還讓我們能夠洞悉其背後的資料查詢邏輯。值得注意的是，這些基礎圖表不包括 Grafana Scenes 客製化的功能和元素，從而提供了一個更簡潔、專注於核心功能的使用體驗。

![Untitled](https://i.imgur.com/5x3zcAz.png)

> 相關設定檔同步收錄在文章底部的 Github 連結。
>

# 結論

自從 Grafana 團隊最初創造了 Grafana 這一工具以來，它們不僅以其為核心逐步構建起了一個完整的生態系統，而且對整個可觀測性領域產生了深遠的影響。通過深入了解 Grafana Faro，我們能體驗從前端的黑盒子到全面前端可觀測性的轉變，體會到細節的豐富和深度。不可否認，Grafana 正在下一盤很大的棋，而且好戲才剛開始。

在 Grafana Cloud 中，我們可以窺見 Grafana 團隊對可觀測性領域發展的獨到見解。目前 Grafana Faro 還處於預覽階段，其使用者界面還在不斷地進行大幅調整。其中 Grafana 團隊快速迭代產品的能力讓人感到讚嘆之餘，也讓我們注意到了他們越來越依賴其開源的客製化插件模組 Grafana Scenes 來增強創造力和優化使用者體驗。

那麼接下來，就讓我們一同探索如何從前端的視角出發，利用 Grafana Scenes 打造出一個獨特且超越 Grafana Cloud 的可視化界面吧。

---

相關程式碼同步收錄在：

https://github.com/MikeHsu0618/grafana-stack-in-kubernetes/tree/main/day34

References:

[The Next America](https://www.pewresearch.org/social-trends/2014/04/10/next-america/)

[Python大數據分析(ㄧ) - HackMD](https://hackmd.io/@aaronlife/python-bigdata-01#資料視覺化1)

[Big Data 的什麼不外包？](https://fredbigdata.blogspot.com/2013/05/big-data.html)

[資料未必氾濫，但洞察的確難求](https://fredbigdata.blogspot.com/2015/07/blog-post_27.html)