# 可觀測性宇宙的第十九天 - Grafana Agent 介紹 - 交織可觀測性的鵲橋

# 概述

在接下的篇章中，我們將繼續探討 Grafana 團隊在各個監控領域所主導的開源專案，除了專注在可觀測性三本柱的各個領域之外，扮演將每個監控數據的收集任務輕鬆彈性組織起來的角色，顯得格外重要，想想看那些 Logs、Ｍetrics、Traces 等監控數據，各有在其領域上的協議及格式，更別提我們的運行環境可能在 Kubernetes、Linux、Ｗindows、Edge devises 上，很少有一套工具可以完全包辦一個監控領域上的數據收集工作，更何況是涵蓋到所有監控領域並兼容各種數據格式及客戶端應用。而身為監控領域的開源領頭團隊 Grafana，藉由在 Grafana Cloud 成熟且被大規模驗證過的心血，提出試圖統一所有領域監控數據傳遞者的全面解決方案「Graana Agent」，就讓我們來瞧瞧他的厲害之處吧！

![https://ithelp.ithome.com.tw/upload/images/20231004/201495621vXXGALbV8.png](https://ithelp.ithome.com.tw/upload/images/20231004/201495621vXXGALbV8.png)

# Grafana Agent 是什麼

根據官方對 Grafana Agent 的描述，它是一個內建多功能的開源遙測收集器，專為收集指標、日誌和追踪而設。Grafana Agent 使用經過實戰驗證的代碼，完全兼容 Prometheus、Loki 和 Tempo 的遙測技術堆棧。Grafana Agent 可以將指標轉發到任何與 Prometheus 兼容的端點，將日誌轉發到任何與 Loki 兼容的端點，並將追踪轉發到任何與 OpenTelemetry 兼容的端點。

Grafana Agent 計劃是在 Grafana Labs 啟動的，並在 2020 年 3 月正式宣布。Grafana Agent 採用 Apache 2.0 許可協議進行發布。

從簡單的幾句敘述中，我們足以看出 Grafana Agent 的長處及目標：「通過對指標、日誌和追踪使用相同的服務發現規則，簡化遙測數據收集」，原本需要各自設定的收集器服務，現在通通只需要一套就可以通通搞定，並且可以兼容擴充，發揮更大的上限。

## Grafana Agent in Prometheus Server

![https://ithelp.ithome.com.tw/upload/images/20231004/20149562HqF86oyB5G.png](https://ithelp.ithome.com.tw/upload/images/20231004/20149562HqF86oyB5G.png)

Grafana Agent 在 Prometheus 可以藉由本身輕量級的代理模式進行對目標指標的抓取，並遠端寫入對應的後端儲存，這種代理模式同時解決了 Prometheus Server 上的幾個問題：

- 資源佔用：傳統的 Prometheus Server 本身作為一個數據收集、儲存、查詢的集合體，有一定的資源門檻，而代理模式作為一個輕量的數據收集氣，它不存數據，只負責以 Wal（Ｗrite-ahead logs）形式收集收集指標，並且在推向遠端存儲後端即刪除，大大減少資源的佔用。
- 邊緣運算和資源緊湊環境：在邊緣和資源緊湊環境中，可能沒有足夠資源運行起整座 Prometheus Server。此時，代理模式的輕量特性，就是為此場景而生。
- 數據持久性問題：傳統的 Prometheus Server 本地存儲可能會因為各種原因（如節點故障）導致數據丟失。而Prometheus Agent可以通過將數據個體到遠程存儲，降低這種風險。
- 簡化數據遠程寫入：雖然 Prometheus Server 本身支持遠程寫入功能，但在實踐中，遠程寫入配置可能會變得複雜。代理模式專門為此目的而設計，使得遠程寫入配置和管理更加簡單。
- 減少存儲和查詢的複雜性：由於代理模式不涉及本地數據存儲和查詢，因此與 Prometheus Server 相比，它大大簡化了操作和管理的複雜性。

## Grafana Agent vs Prometheus Agent

有些眼尖的同學看到這裡，心中一定都不約而同起了一個疑問：「怎麼感覺跟 Prometheus Agent 有八十幾趴相似？」 。沒錯，事實上 Grafana 團隊與 Prometheus 社群的開發一直離不開彼此，並且互相影響著對方。

Prometheus Agent 的誕生和 Grafana Agent 有著密不可分的關聯。在 2020 年，Grafana 團隊推出了名為 Grafana Agent 的工具，它是 Prometheus 專案分支出來的一個簡化版本，只專注於遠程寫入指標，而不涉及 TSDB，並且開始大規模的實施在 Grafana Cloud 上，得到正面的肯定。

這個設計受到了許多關注和興趣。為了滿足社區的需求，Prometheus 團隊在同年的 12 月舉行了一次開發者峰會，決定將 Grafana Agent 的功能整合到官方的 Prometheus 專案中。經過數月的討論和開發，2021 年 10 月，該功能正式被合併到 Prometheus v2.32 中。簡而言之，Grafana Agent 的設計和概念為 Prometheus 帶來了新的啟示，並促使它在其主要版本中加入了 Agent 概念的功能，這被認為是 CNCF 生態系中的一個遊戲規則改變者。

# Grafana Agent Modes

Grafana Agent 是一款不受供應商限制的、內建功能完整的遙測數據收集器。它的設計旨在靈活、高效能，並且與多種生態系統如 Prometheus 和 OpenTelemetry 相容。

得益於開源社群的活躍，逐漸演變出目前的三種 mode：

1. Static mode：Grafana Agent 的默認、原始版本。
2. Static mode Kubernetes operator：專門管理在 Kubernetes 中運行靜態模式 Grafana Agent 資源的版本。
3. Flow mode：對 Grafana Agent 的新穎、更靈活的重新構想版本。

我們不需要只選用一種模式，對 Grafana Agent 我們可以依照實際需求混合使用。

## **Stability**

到目前為止 Grafana Agent 共同存在著三種 mode，其中 Static mode Kubernetes operator 還在 Beta。

> Flow模式被認為是 Grafana Agent 項目的未來。最終，Static mode 和 Static mode Kubernetes Operator 的所有功能都將添加到 Flow 模式中。此目標目前還處於早期階段，Loki、Tempo 等等的Helm 安裝包項目，目前還是預設是用 Static mode Kubernetes operator。
>

## Choose which kind of Grafana Agent

因為目前 Grafana Agent 功能已大致成形，但根據社群意願及判斷未來趨勢，還需要進行大量的整合作業，而官方根據目前這三種模式的應用場景給我們了選擇建議：

1. Static mode：靜態模式是於2020年3月3日首次推出的 Grafana Agent 的原始版本，也是最成熟的版本。您應該使用靜態模式當：
    - 需要使用 Grafana Agent 最成熟的版本。
    - 需要與 Grafana Cloud 進行整合。
    - 需要使用還未作為 Flow mode 組件提供的 Static mode 整合。
2. Static mode Kubernetes operator：此版本於 2021年6月17日首次推出，目前處於 Beta 階段。它是為了與 Prometheus Operator 相容而推出的，允許 Static mode 支援 Prometheus Operator 的資源，如 ServiceMonitors、PodMonitors 和 Probes。您應該使用此版本當：
    - 需要能夠從 Prometheus Operator 項目收集 Prometheus 指標，使用如 ServiceMonitors、PodMonitors 和 Probes。
    - 不需要收集追蹤指標。
3. Flow mode：此版本於2022年9月29日首次推出，是 Grafana Agent 的穩定版本。流模式的推出是基於重新構想 Grafana Agent，著重於供應商中立、易用性、提高的除錯能力，以及通過採用代碼作為配置模型來滿足專業用戶的需求。流模式被認為是 Grafana Agent 項目的未來。最終，靜態模式和靜態模式 Kubernetes 運營者的所有功能將被加入流模式。您應該使用流模式當：
    - 需要特有的流模式功能。
    - 需要更輕鬆地使用 UI 來除錯配置問題。
    - 需要收集 OpenTelemetry 的指標、日誌和跟踪的完整支援。
    - 需要支援來自 Prometheus Operator 項目的 PrometheusRule 資源，以配置 Grafana Mimir。
    - 需要能夠將 Prometheus 和 Loki 管道轉換為 OpenTelemetry Collector 管道，反之亦然。
    - 需要收集 Grafana Pyroscope 的檔案的支援。

# Grafana Agent Usage

![https://ithelp.ithome.com.tw/upload/images/20231004/20149562OLW4oHK9af.png](https://ithelp.ithome.com.tw/upload/images/20231004/20149562OLW4oHK9af.png)

使用 Grafana Agent 前後我們可以很容易的對照：

|  | Before | After |
| --- | --- | --- |
| Prometheus(Metrics) | Promethus(server/agent) … | Grafana Agent |
| Grafana Mimir (Metrics) | Promethus(server/agent) … | Grafana Agent |
| Grafana Loki(Logs) | filebeat / fluentd / promtail … | Grafana Agent |
| Elastricsearch(Logs) | filebeat / fluentd / promtail … | Grafana Agent |
| OpenTelemetry(Traces) | otel(collector/agent) / jaeger(collector/agent) / zipkin … | Grafana Agent |
| Grafana Tempo(Traces) | otel(collector/agent) / jaeger(collector/agent) / zipkin … | Grafana Agent |

在了解了 Grafana Agent 的強大之處後，我們可以把它理解為所有遙測資料的收集者，所有我們以前用來傳遞、收集監控資料的任何媒介，都可以試著將它轉換成 Grafana Agent，此時重新審視你的監控系統架構，將會是前所未有的一目瞭然。

## Grafana Agant on Metrics

![https://ithelp.ithome.com.tw/upload/images/20231004/20149562f7tPgrs3kD.png](https://ithelp.ithome.com.tw/upload/images/20231004/20149562f7tPgrs3kD.png)

## Grafana Agant on Logs

![https://ithelp.ithome.com.tw/upload/images/20231004/20149562jEH85HRazE.png](https://ithelp.ithome.com.tw/upload/images/20231004/20149562jEH85HRazE.png)

## Grafana Agant on Traces

![https://ithelp.ithome.com.tw/upload/images/20231004/20149562cIxi5znUOH.png](https://ithelp.ithome.com.tw/upload/images/20231004/20149562cIxi5znUOH.png)

## Observability Ecosystem

### Ｗith Grafana Agent

![https://ithelp.ithome.com.tw/upload/images/20231004/20149562FPHsuM4LEw.png](https://ithelp.ithome.com.tw/upload/images/20231004/20149562FPHsuM4LEw.png)

### Ｗithout Grafana Agent

![https://ithelp.ithome.com.tw/upload/images/20231004/20149562DGPD6kWDXX.png](https://ithelp.ithome.com.tw/upload/images/20231004/20149562DGPD6kWDXX.png)

# 結論

在這個章節，我們深入探討了 Grafana Agent。Grafana 團隊對它有著遠大的期望，不僅希望它是一個簡單的 Agent，更期待它成為跨平台、跨運作環境的監控整合工具，能減少監控系統建置和維護的複雜性，甚至有望成為業界的共同標準。Grafana Agent 也確實因應實際需求和社群的反饋，進行了多次的迭代和改進，充分展現了開源社群的力量。下一步，我們將深入 Grafana Agent 的內部運作，探索其在不同模式下的結構及運作細節，讓我們一起期待這精彩的實戰解析！

---

References

[Introducing programmable pipelines with Grafana Agent Flow | Grafana Labs](https://grafana.com/blog/2022/09/29/introducing-programmable-pipelines-with-grafana-agent-flow/)

[Why we created a Prometheus Agent mode from the Grafana Agent | Grafana Labs](https://grafana.com/blog/2021/11/16/why-we-created-a-prometheus-agent-mode-from-the-grafana-agent/)

[How to collect and query Kubernetes logs with Grafana Loki, Grafana, and Grafana Agent | Grafana Labs](https://grafana.com/blog/2023/04/12/how-to-collect-and-query-kubernetes-logs-with-grafana-loki-grafana-and-grafana-agent/)

[](https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/designs/prometheus-agent.md)

[Introducing Prometheus Agent Mode, an Efficient and Cloud-Native Way for Metric Forwarding | Prometheus](https://prometheus.io/blog/2021/11/16/agent/)