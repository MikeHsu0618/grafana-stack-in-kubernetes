# 可觀測性宇宙的第十二天 - Prometheus 全家桶介紹

# 概述

終於，經歷了介紹可觀測性與監控的差異後，我們了解 Grafana 在這個領域佈局許久並且準備繼續深掘，而我們也早已準備好秤手工具，進入知識量爆炸的 Prometheus 實戰環節。現在，我們將要開起在巨人肩膀上的旅途，因爲可觀測性的領域所需具備的知識層面真的很廣，抓穩了，火箭準備起飛！

# Prometheus 全家桶比較

在當今的監控領域，Prometheus 不僅僅是一個獨立的工具，它已經逐漸成為了一個強大的生態系統的核心。不少工具和服務，無論是直接或間接，都是受到 Prometheus 的影響而生的。這些相關的工具和服務，為 Prometheus 提供了豐富的插件功能，極大地擴展了其監控的邊界和能力。因此，在實際的監控應用中，我們很少只依賴 Prometheus 本身，更多的是一套綜合利用了多種工具的綜合解決方案，從而最大化地發揮 Prometheus 的威力。

於是，為了幫助開發者和運維人員更好地利用這套生態，有許多開源社群秉持著分享的精神，貢獻了一系列的 Prometheus 套裝工具，就是我們常聽到的 Prometheus 全家桶。這套工具不僅能快速幫助我們部署起一個完整的監控系統，還確保了在各種情境下，我們都能有信心地依賴它來監控我們的系統和應用。接下來，就讓我們來先看看目前幾款熱門人選吧！

## Prometheus Operator

Prometheus Operator 是 Prometheus 在 Kubernetes 中一個非常重要的部分，最早由 CoreOS 團隊創建，旨在為 Kubernetes 提供 Prometheus 監控的完整設置。它簡化了 Prometheus 和 Alertmanager 在 Kubernetes 上的部署和管理。在 Kubernetes 命名習慣中，凡是帶有 Operator 的服務，有很高機率都自帶著 Custom Resource Definitions（CRDs）功能，他讓我們輕鬆的如同撰寫 Kubernetes 內建資源設定檔一樣，輕鬆建立、管理 Prometheus、Alertmanager、ServiceMenitor 各種無縫與 Kuberentes API 整合的資源，使其在 Kubernetes 環境中部署和運行變得更加簡單和自動化，而不是花費大量時間成本在維護 Prometheus 本身。

![https://ithelp.ithome.com.tw/upload/images/20230927/201495629awqJutSYu.png](https://ithelp.ithome.com.tw/upload/images/20230927/201495629awqJutSYu.png)

## **Prometheus Operator vs. Kube-Prometheus vs. Kube-Prometheus-Stack**

沒錯，身為一個熱門開源專案，Prometheus 連全家桶的選擇都讓人有幸福的選擇障礙。之所以特別提出來講，是因為常常有像我一開始一樣的初學者，看到眼前的複雜資訊無法消化其背後差異，就讓我們在開始使用他們之前，先了解他們的關係吧。

### **Kube-Prometheus 和 Kube-Prometheus-Stack 差異**

Kube-Prometheus 和 Kube-Prometheus-Stack 都是旨在於藉由 Prometheus Operator 為 Kubernetes 叢集量身打造一套完整且多元的監控服務而生，我們將在後面比較差異。其中這兩者與 Prometheus Operator 常常被搞不請楚的狀況主要有兩個，其一是 Kube-Prometheus 跟 Prometheus Operator 一樣都是由 CoreOS 團隊當時所創建的專案，所以他們是屬於相同團隊下的專案關係，另一個原因是 Kube-Prometheus-Stack 這個也以 Prometheus Operator 為核心建立成的 Helm Chart 專案，這個專案在一開始不叫做 Kube-Prometheus-Stack 而是撞名的「Prometheus Operator」，之後這個名為「Prometheus Operator」的 Helm Chart 專案才改名為「 Kube-Prometheus-Stack 」。

> Kube-Prometheus-Stack 官方的聲明如下：
*Note: This chart was formerly named `prometheus-operator` chart, now renamed to more clearly reflect that it installs the `kube-prometheus` project stack, within which Prometheus Operator is only one component.*
>

此外，Kube-Prometheus 和 Kube-Prometheus-Stack 兩者皆提供了一套工具和配置，用於在 Kubernetes 上建立完整的 Prometheus 監控環境。最大的差別在於 Kube-Prometheus-Stack 是一個更集成、包含更多功能和外部整合的 Helm Chart，而 Kube-Prometheus 主要依靠 jsonnet 和 kubectl 進行部署，提供一套直接的 YAML 文件來部署，適合需要更細致客製化的情境。

# 為什麼推薦 Kube-Prometheus-Stack

正如剛提到的，Kube-Prometheus-Stack 是一個在 Kubernetes 上建立完整 Prometheus 監控環境的 Helm chart，大大幫我們減輕前期設定的負擔以及學習成本，並且預置了多種監控組件，以及與之相關的 Grafana 圖表和 AlertManager 告警規則，儼然已經成為現在大多數社群入門的首要選則。

以下是 Kube-Prometheus-Stack 包含的主要組件，以及其簡單介紹：

1. Prometheus:
    - 主要的時序數據庫，負責數據收集和存儲。
    - 提供 PromQL，一種強大的查詢語言，用於時間序列數據分析。
2. Alertmanager:
    - 負責管理和處理 Prometheus 生成的各種告警。
    - 可整合多種通知方法，不限於 Email, Slack 等。
3. Grafana:
    - 可視化儀表板工具，用於展示 Prometheus 收集的監控數據。
    - Kube-Prometheus-Stack 提供了多種預先定義的圖表，許多有用的圖表和告警都來自 [kubernetes-mixin](https://github.com/kubernetes-monitoring/kubernetes-mixin) 專案，是開箱及用的 Kubernetes 監控。
4. node-exporter:
    - 在每個 Kubernetes 節點上運行，負責收集節點相關的系統和硬體指標。
5. kube-state-metrics:
    - 主要是為了提供 Kubernetes 對象的狀態信息。它是一個簡單的服務，監控 Kubernetes API server 並生成有關 Kubernetes 對象（例如 Deployments、Pods、Services 等）的指標。
6. Prometheus Operator:
    - 用於在 Kubernetes 中自動化 Prometheus 和 AlertManager 的配置和運維。
    - 允許用戶通過 Kubernetes 自定義資源 (CRDs) 來定義和管理 Prometheus 和 AlertManager 實例。
7. 其他 Exporters:
    - 例如 blackbox-exporter、kube-scheduler-exporter 等，用於提供特定應用或組件的監控指標。

總之，Kube-Prometheus-Stack 提供了一個全套的監控解決方案，非常適合希望快速部署且具有高度整合度的 Kubernetes 監控環境。

# 結論

在這篇文章中，我們詳細探討了 Prometheus 生態系統中的各種全家桶，特別是 Kube-Prometheus-Stack。回顧我初次踏入 Prometheus 的領域時，其學習曲線確實陡峭，尤其是面對眾多的選項和不知所措的知識點。但透過對事物的原理深入瞭解、底層結構的把握，以及對不同工具的優缺點分析，我們能夠對整體領域有更深入的認識。這也解釋了，為何許多資深前輩總是鼓勵我們從底層開始，深入研究官方文件，雖然一開始痛苦點，但絕對值得啊！

---

References

https://juejin.cn/post/7212776427166105637

https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

https://github.com/prometheus-operator/kube-prometheus

https://github.com/prometheus-operator/prometheus-operator/issues/2619