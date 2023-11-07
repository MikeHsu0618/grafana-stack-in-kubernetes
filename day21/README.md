# 可觀測性宇宙的第二十一天 - Grafana Agent - Flow mode 實戰

# 概述

在上一個章節中，我們深入探討了別具一格的 Grafana Agent Operator。它允許我們在Kubernetes叢集中直接重用現有的 Prometheus Operator CRD 及其他相關資源。儘管這在 Kubernetes 中是一個理想的解決方案，但顯然 Grafana 團隊的願景遠不止於此。他們的目標不僅是專注於 Kubernetes 或傳統的設定方式。他們致力解決人們在各種不同環境中設置監控收集器時所遇到的問題，並真正地降低人們部署全面監控的障礙。不論您使用的是Linux、Windows、容器還是實體主機，只需採用統一的服務和設定方式，即可實現跨平台、跨設備的監控數據收集。基於此，Grafana團隊為我們帶來了全新的「Grafana Agent Flow」。

![https://ithelp.ithome.com.tw/upload/images/20231006/20149562YHQQJDkm6O.png](https://ithelp.ithome.com.tw/upload/images/20231006/20149562YHQQJDkm6O.png)

# Grafana Agent Flow 是什麼

「Grafana Agent  Flow mode」是 Grafana Agent 於 2022 年九月正式推出的的最新模式，也被稱為「Grafana Agent Flow」。這個模式是基於組件的重新構想版本，主要著重於易用性、可除錯性，以及滿足專業用戶需求的能力。

組件化的設計理念帶來了三大核心優勢：

1. **可重用性**：允許組件的輸出被用作多個其他組件的輸入，從而實現資源的最大化利用。
2. **可組合性**：組件可以被串連在一起，形成一個流程管道，這增強了彈性與客製化的可能性。
3. **單一任務導向**：每個組件的範疇被限制於一個特定的狹窄任務，這確保了其功能明確且副作用減少。

主要特色包括：

- 使用類似於Terraform的語言來寫出宣告式的設定。
- 透過宣告組件來配置管道的特定部分。
- 利用表達式來綁定組件，從而建構出一個可程式化的管道。
- 為了方便除錯，它還包含了一個專用的用戶界面，幫助用戶追踪管道的狀態。

總之，Grafana Agent Flow 為我們呈現了一套既強大又具有高度彈性的工具。對於熟稔 Terraform 組件和管道架構的使用者來說，是一個無縫接軌的體驗。Grafana 團隊從 Terraform 在多雲環境中一致性部署設定的成功策略中獲取靈感，開發出了 **`.river`** 設定檔。這不只是一個易於配置和管理的檔案格式，它更具備了深度客製化的潛力，完美滿足專業用戶的多元需求。而在 Flow mode 中，我們還可利用其內建的 Web UI，確保每一段管道都按照預期進行操作。

# Grafana Agent Flow 重要概念

Grafana Agent Flow 採用了一種名為 River 的自定義設定檔語言，其專為動態配置和連接組件而設計。River 的主要目標是通過使設定更易讀和易寫來減少設定文件中的錯誤。使用者可以輕鬆地從文檔中複製和粘貼，迅速入門。

此外，River 設定文件不僅聲明 Grafana Agent Flow 啟動哪些組件，還明確指示如何將它們綁定到一個管道中。值得注意的是，儘管 River 語言與 Terraform 所用的 HCL 語言在外觀上相似，但其語法和功能卻有所不同。River 的結構主要基於區塊、屬性和表達式，它具有聲明性質，因此組件、區塊和屬性的順序不固定，而是由組件間的關係決定。

## ****Components**** 組件

```jsx
prometheus.scrape "default" {
  targets = [{
    "__address__" = "demo.robustperception.io:9090",
  }]
  forward_to = [prometheus.remote_write.default.receiver]
}

prometheus.remote_write "default" {
  endpoint {
    url = "http://localhost:9009/api/prom/push"
  }
}
```

組件是 Grafana Agent Flow 的區塊，每個組件負責處理單個任務，例如檢索參數或收集 Prometheus 指標。

組件由兩部分組成：

- Arguments：配置組件的設置，如 prometheus.scrape。
- Exports：組件向其他組件公開的命名值，如 "default"。

## ****Pipelines**** 管道

當一個組件的參數引用另一組件的導出字段時，就建立了一個依賴關係：一個組件的輸入（參數）現在依賴於另一組件的輸出（導出）。每當引用的組件的導出被更新時，該組件的輸入都將被重新評估。

這些組件之間的引用所形成的數據流動就像一條管道。

例如，一個典型的管道可能如下：

1. 一個 local.file 組件監視含有API key 的磁盤上的文件。
2. 一個 prometheus.remote_write 組件被配置為接收指標，並使用來自 local.file 的API鑰匙進行身份驗證，將它們轉發到外部數據庫。
3. 一個 discovery.kubernetes 組件發現並導出可以收集指標的 Kubernetes Pods。
4. 一個 prometheus.scrape 組件引用前一組件的導出，並將收集到的指標發送到 prometheus.remote_write 組件。

![https://ithelp.ithome.com.tw/upload/images/20231006/201495624SR8Lm2eHI.png](https://ithelp.ithome.com.tw/upload/images/20231006/201495624SR8Lm2eHI.png)

## **Component Controller 組件控制器**

Grafana Agent Flow 的核心部分是其組件控制器，它在運行時負責組件的一系列管理工作。

主要職責包括：

1. **讀取和驗證設定文件**：確保所有設定符合系統要求。
2. **管理組件的生命週期**：從啟動到終止，監控組件的所有階段。
3. **評估配置組件的參數**：確保每個組件具有正確的設定值。
4. **報告已定義組件的健康狀態**：對外提供組件運行的健康狀況信息。

另外，組件之間形成的有向無環圖 (DAG) 賦予了組件間的關係，這不僅確保了組件參照順序的正確性，還防止了自我參照或循環參照的問題。在評估過程中，每個組件都是基於其所有依賴完成評估後再進行的。組件的健康狀態也是一個重要指標，它可能是：未知、健康、不健康或已退出。若組件評估失敗，將會被標記為不健康，但還是會繼續運行，以避免可能的失敗傳播。

藉由組建控制器，我們已經可以清楚的了解 Grafana Agent 內部的真實運作情況，這也使其能夠收集內部資訊以及組件的健康狀態，最終使用 UI 的方式呈現給我們，大大提升我們除錯的效率。

![https://ithelp.ithome.com.tw/upload/images/20231006/20149562TqIE7s6XyI.png](https://ithelp.ithome.com.tw/upload/images/20231006/20149562TqIE7s6XyI.png)

值得注意的是，部分組件支持內存中的通信，如 prometheus.exporter.unix，使得同一過程中的組件可以直接互相交流。此外，控制器提供了方便的更新機制，允許使用者通過 HTTP 端點或信號快速重新載入配置文件，確保系統配置始終是最新的。

## Module 模組

Module 是建立 Grafana Agent Flow 配置的其中一種方法，可以作為一個組件進行載入。這些模組非常適用於參數化配置，以創建可重用的管道，在這裡的設計精神與 Terraform 如出一徹。

Grafana Agent Flow的模組包含：

- Arguments：配置模組的設置。
- Exports：模組對模組消費者暴露的命名值。
- Components：當模組運行時需要運行的Grafana Agent Flow組件。

透過 Module Loader，模組被載入到Grafana Agent Flow中，Module Loader 被視為一個 Grafana Agent Flow 組件，它能夠檢索模組並運行其中定義的組件。這些加載器的主要職責包括檢索要運行的模組源，為模組運行中創建一個組件控制器，將參數傳遞給已加載的模組，並從已加載的模組中暴露導出值。這些模組加載器通常被稱為 module.LOADER_NAME，具體的加載器列表可以在 Grafana Agent Flow 組件列表中找到。

模組的設計目的是為了靈活性，它可以從任何地方檢索配置，如本地文件系統、S3存儲桶或HTTP端點。每個模組加載器組件都支持不同的檢索模組源的方式。例如，最通用的模組加載器組件 module.string 可以從另一個Flow組件的導出值中加載模組。

舉例來說，以下是一個模組的範例，該模組管理一條管道，該管道過濾掉提供給它的debug和info級別的日誌行。該模組可以保存到一個文件中，然後用作寫入Loki之前的處理步驟：

1. 建立出一條可以過濾掉 Debug 與 Info 等級的日誌管道，其中可以設定輸入參數、執行組件、輸出值。

```jsx
argument "write_to" {
  optional = false
}

loki.process "filter" {
  stage.match {
    selector = "{job!=\"\"} |~ \"level=(debug|info)\""
    action   = "drop"
  }
  forward_to = argument.write_to.value
}
export "filter_input" {
  value = loki.process.filter.receiver
}
```

2. 接下來我們就可以使用上面的 Module 作為管道的一部分執行，實現在寫入日誌之前先過濾掉多餘日誌。

```jsx
loki.source.file "self" {
  targets = LOG_TARGETS
  forward_to = [module.file.log_filter.exports.filter_input]
}

module.file "log_filter" {
  filename = "/path/to/modules/log_filter.river"

  arguments {
    write_to = [loki.write.default.receiver],
  }
}

loki.write "default" {
  endpoint {
    url = "LOKI_URL"
  }
}
```

## Clustering 叢集

Grfana Agent 叢集功能使得其能夠協同工作，以實現工作負載分配和高可用性。它有助於以最少的資源和運營開銷創建水平可擴展的部署。

使用上必須要特地聲明 clustering 設定：

```jsx
prometheus.scrape "default" {
    clustering {
        enabled = true
    }
    ...
}
```

目標自動分配是叢集最基本的使用情境，它允許在所有對等點上運行的抓取組件在它們之間分配抓取負載。

當新節點加入或現有節點消失時，會檢測到叢集狀態更改。所有參與組件都會在本地重新計算目標所有權並重新平衡它們正在抓取的目標數量，而無需通過網絡明確傳達抓取目標設定，此舉幫助我們分配動態擴展的 Grafana Agent 數量以在高峰期間分配工作負載，提供了非常大的彈性。

# Grafana Agent Flow 設定檔工具

正如前面所提到的，Grafana Agent Flow 是一個非常新的專案，但它堅定的被 Grafana 團隊認定為是 Grafana Agent 的未來走向，Statics mode 與 Statics mode Kubernetes Operator 最終都將與 Flow mode 靠攏，其中一個原因是 Grafana Agent 見證了 Terraform 在 IaC（Infrastructure as Code） 領域上跨平台的高度兼容性及組件使用的彈性。為此，Grafana 除了為了我們提供更多元的監控收集服務之壞，更提供許多轉換工具來幫助我們更快的進入 Grafana Agent Flow 的生態中，現在就讓我們趕緊來瞧瞧。

## Grafana Agent Cli - Convert

使用指令 grafana-agent convert 功能，可以將我們的現有的設定檔轉成 Grafana Agent 設定檔格式。

目前支持無痛轉換的設定檔有：

- Prometheus
- Promtail

現在我們就使用一個 Prometheus 抓取指標的設定檔，將它轉換成 Grafana Agent Flow 設定檔格式：

```jsx
# Prometheus
global:
  scrape_timeout:    45s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:12345"]

remote_write:
  - name: "grafana-cloud"
    url: "https://prometheus-us-central1.grafana.net/api/prom/push"
```

接下來使用 grafana-agent convert 指令來產生 Grafana Agent Flow 設定檔

```jsx
AGENT_MODE=flow; grafana-agent convert --source-format=prometheus --output=OUTPUT_CONFIG_PATH INPUT_CONFIG_PATH
------
# OUTPUT_CONFIG_PATH.river
prometheus.scrape "prometheus" {
  targets = [{
    __address__ = "localhost:12345",
  }]
  forward_to     = [prometheus.remote_write.default.receiver]
  job_name       = "prometheus"
  scrape_timeout = "45s"
}

prometheus.remote_write "default" {
  endpoint {
    name = "grafana-cloud"
    url  = "https://prometheus-us-central1.grafana.net/api/prom/push"
  }
}
```

## ****Grafana Agent Configuration Generator****

近期 Grafana 團隊又開啟了一個 agent-configurator 專案，提供強大的前端設定檔編寫頁面，使我們不必先啃完陌生艱難的文件，才能開始使用 Grafana Agent Flow，並且提供與 Grafana Cloud 友善的完整設定，實在是剛入門的使用者一大福音。

有興趣的同學可以前往體驗：https://grafana.github.io/agent-configurator/。

> 注意：此功能還處於實驗階段，並還沒完成全部組件的設定。
>

![https://ithelp.ithome.com.tw/upload/images/20231006/20149562DLDEKe4iHa.png](https://ithelp.ithome.com.tw/upload/images/20231006/20149562DLDEKe4iHa.png)

# 實戰演練

在接下來的實戰中，我們將利用先前建立好的 Kube-Prometheus-Stack 及其相關設定，來當作 Grafana Agent 的指標 remote_write 端點，其中詳細安裝過程可以回頭複習一下「Kube-Prometheus-Stack 實戰系列」。唯一不同的是我們將解開 Prometheus 的遠端寫入 remote_write 的封印。

## 開啟 Prometheus remote_write

```jsx
# values.yaml
....
prometheus:
  enabled: true
  prometheusSpec:
    enableRemoteWriteReceiver: true
```

我們只要簡單的將原先的設定檔 enableRemoteWriteReceiver 參數調成 true 即可馬上執行 Helm 更新。

```jsx
helm upgrade --install prometheus-stack prometheus-community/kube-prometheus-stack --values=values.yaml -n prometheus --create-namespace
```

## 使用 Grafana Agent Flow 對 Kubernetes 叢集抓取指標

接下來，我們規劃個使用情境，來讓我們更清楚 Grafana Agent Flow 是如何透過組件互相協作。現在我們有個抓取 Kubernetes 叢集內資源指標的需求，我們必須先透過 Grafana Agent Flow 對 Kubernetes 進行服務發現，鎖定需要抓取的目標資源，並且建立主要抓取指標並傳遞到目的地的組件，所以我們還會需要有一個 Prometheus Remote Write 的組件來作為指標遠端寫入的終點。

以上的敘述中，我們可以理解成：

1. 對抓取目標建立 Kubernetes 服務發現的資源組件
2. 對 Prometheus 遠端寫入的輸出組件。
3. 執行對資源組件抓取指標並且輸出到目的的執行組件。

以上組件預期會成為具備輸入、過程、輸出的一條管道，並且組件之間形成的有向無環圖，防止了自我參照或循環參照的問題。

```jsx
# configmap.yaml
agent:
  configMap:
    create: true
    content: |
      logging {
      	level  = "info"
      	format = "logfmt"
      }

      discovery.kubernetes "pods" {
      	role = "pod"
      }

      discovery.kubernetes "nodes" {
      	role = "node"
      }

      discovery.kubernetes "services" {
      	role = "service"
      }

      discovery.kubernetes "endpoints" {
      	role = "endpoints"
      }

      discovery.kubernetes "endpointslices" {
      	role = "endpointslice"
      }

      discovery.kubernetes "ingresses" {
      	role = "ingress"
      }
      
      prometheus.scrape "pods" {
        targets    = discovery.kubernetes.pods.targets
        forward_to = [prometheus.remote_write.default.receiver]
      }
      
      prometheus.scrape "nodes" {
          targets    = discovery.kubernetes.nodes.targets
          forward_to = [prometheus.remote_write.default.receiver]
      }

      prometheus.remote_write "default" {
        endpoint {
          url = "http://prometheus-stack-prometheus.prometheus:9090/api/v1/write"
        }
      }
```

> 眼尖的同學可能有發現 discovery.kubernetes 的數量跟 prometheus.scrape 輸出的數量有差異，在後面我們將一起見證 Grafana Agent Flow 為我們帶來的方便之處。
>

```jsx
# values.yaml
fullnameOverride: grafana-agent-flow

agent:
  mode: 'flow'
  clustering:
    enabled: true
  enableReporting: false
controller:
  type: 'statefulset'
  replicas: 3
  volumeClaimTemplates: []
```

在上面的設定我們理所當然的選擇了 Flow mode，並且開啟了 clustering 功能幫忙分擔負載，最重要的是我們關閉了 Grafana 為我們貼心預設開啟的回傳使用資料 enableReporting: true 設定。

## 執行並驗證 Grafana Agent 設定

首先確認 Grafana Helm Repo 是否存在。

```jsx
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

接下來就可以直接安裝 grafana/grafana-agent。

```jsx

helm upgrade --install grafana-agent-flow grafana/grafana-agent --values values.yaml --values configmap.yaml -n monitoring --create-namespace
```

### 驗證 Pipelines 執行

導出 Grafana Agent Flow UI 中的 Graph 頁面查看：

```jsx
kubectl port-forward service/grafana-agent-flow-cluster 8080:80 -n monitoring
------
Forwarding from 127.0.0.1:8080 -> 80na-agent-flow-cluster 8080:80 -n monitoring
Forwarding from [::1]:8080 -> 80
```

![https://ithelp.ithome.com.tw/upload/images/20231006/20149562J0n40JNjki.png](https://ithelp.ithome.com.tw/upload/images/20231006/20149562J0n40JNjki.png)

在上面的 configmap.yaml 中我們明確的聲明六個 Kubernetes 服務發現組件，並且刻意的只抓取其中的 pods 跟 nodes 組件遠端寫入我們的目標 Prometheus 儲存，而在 Grafana Agent Flow 的 UI 介面中，清楚明瞭的展示我們對管道的設定並且串連起組件之間各自的關係圖，其中可以清楚看出關於 services、ingresses、endpointslices、endpoints 如預期的並未被任何組件關聯。

### 驗證 Prometheus 是否成功遠端寫入

讓我們導出 Prometheus UI 查看指標是否成功寫入：

```jsx
kubectl port-forward service/prometheus-stack-prometheus 9090:9090 -n prometheus
------
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
```

![https://ithelp.ithome.com.tw/upload/images/20231006/201495629O3AIHiSeX.png](https://ithelp.ithome.com.tw/upload/images/20231006/201495629O3AIHiSeX.png)

在 Prometheus UI 中，我們可以對 job="prometheus.scrape.*" 相關 Grafana Agent 組件標籤進行查詢，即可查看是否指標順利的遠端寫入進 Prometheus 中。

# 結論

現在，我們已深入探討並完成了Grafana系列工具中的Grafana Agent部分。雖然在我看來，這個扮演監控數據收集角色的部分可能不如其他監控領域專案那麼引人矚目，但當深入到繁瑣的生產環境時，我們就會發現架構設計和管道規劃的複雜度。每當我們針對不同的監控領域增加一種監控收集服務且需要了解新的設定檔時，這都將帶來沉痛的學習成本。Grafana Agent的出現恰好解決了這些問題，它巧妙地將各個領域的差異化進行了抽象和簡化，讓我們能夠更迅速、更有效地享受完整監控的優勢。可以肯定地說，Grafana再次為我們展現了如何顛覆並領航整個行業的新生態。

---

References

[Collect and forward Prometheus metrics |  Grafana Agent documentation](https://grafana.com/docs/agent/v0.35/flow/getting-started/collect-prometheus-metrics/)

[Grafana Agent Configurator](https://grafana.github.io/agent-configurator/)

[Collect and forward Prometheus metrics |  Grafana Agent documentation](https://grafana.com/docs/agent/v0.35/flow/getting-started/collect-prometheus-metrics/)

[Grafana Agent Configurator](https://grafana.github.io/agent-configurator/)