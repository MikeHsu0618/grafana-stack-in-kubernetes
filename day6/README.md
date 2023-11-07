# 可觀測性宇宙的第六天 - Grafana 上的可觀測性宇宙

# 概述

如果講到 Grafana 這家以開源為精神指標的公司，我想很多人第一個反應都跟我一樣，就是那個「可以把資料變成好看圖表的 Dashboard 工具」吧。直到逐漸地接近真相後，我才發現之前的自己好傻好天真。

Grafana 不僅僅是一個可視化圖表工具，他更像是一個給數據專家盡情發揮的畫布，表達出隱藏在數據背後的真正含義。Grafana 在數據可視化的深度和廣度圓圓超出我最初的認知，在每個功能、每個插件都無疑的是可觀測性領域的不可質疑的領先地位，接下來我們來一起一窺 Grafana 的可觀測性宇宙。

# 你以為的 Grafana

如同我所提到的，對大多數人來說，當他們想到 Grafana，腦海中浮現的往往只是一個將 API 的數據拉取下來，再用以展示各種質感十足的 Dashboard 工具。這樣的認知大概率來自 Grafana 初期給人的印象或是大多數人接觸到的僅是其表層功能。但這樣的認知只涵蓋了冰山一角，因為事實上，Grafana 早已跨足並深入到可觀測性的各個角落，打造出一整套完整的產品線，它不再僅僅是一個展示工具，而是整個可觀測性宇宙的核心。

![https://ithelp.ithome.com.tw/upload/images/20230921/20149562R19T4W1Sxm.png](https://ithelp.ithome.com.tw/upload/images/20230921/20149562R19T4W1Sxm.png)

# 事實上的 Grafana

在 2023 年的今天，Grafana Lab 不再只是一個簡單的可視化介面工具。事實上，透過其高度相容的插件，它已經轉型為數據的樞紐，連接了多種監控和日誌系統。不僅如此，Grafana Lab 更進一步開發和收購了多種領域的可觀測性工具，與 Grafana 進行深度整合，打造出一套現象級別的開源可觀測性宇宙。這一切的目的都是為了幫助開發者和維運人員更輕鬆地解讀各種數據源。

身為一家堅持開源理念的公司，Grafana Lab 發展出了 OSS（Open Source Software）、Cloud 和 Enterprise 這三種平台模式。OSS 版本讓全球使用者可以免費使用或者參與到 Grafana 的各個專案中。同時，對於那些不想或沒有能力自行架設可觀測性工具的企業，Grafana Lab 提供專業而成熟的企業級解決方案，這也成為他們的盈利模式。所得的資源再回頭投入開源專案的開發，形成了一個取之於開源，回饋於開源的良性循環。

> 隨著使用在大型企業中的普及，Grafana Labs 於 2016 年後推出了兩款付費產品：Grafana Enterprise，提供企業級組織所需的功能；2017 年， Grafana Cloud 是一個支持 Graphite 的完全託管指標平台。
>

![https://ithelp.ithome.com.tw/upload/images/20230921/20149562fEOFtsrJS9.png](https://ithelp.ithome.com.tw/upload/images/20230921/20149562fEOFtsrJS9.png)

# Grafana 可觀測性宇宙 - LGTM

![https://ithelp.ithome.com.tw/upload/images/20230921/201495628CDglnY9lI.png](https://ithelp.ithome.com.tw/upload/images/20230921/201495628CDglnY9lI.png)

在先前的介紹中，我們不斷提到 Grafana Lab 近年來的目標是致力於建立完整的可觀測性宇宙技術棧，而在今年 3月15 號宣佈收購了 Continuous Profiling 領域處於領導地位的 Pyroscope，進而用僅僅兩個半月的時間，將自家的 Grafana Phlare 與 Pyroscope 合併成 Grafana 新專案 Grafana Pyroscope，可以說 Grafana 終於湊齊了最後一塊拼圖。眼尖的朋友們應該也注意到在先前的圖中，就已透露出 Grafana 補全了對應可觀測性三本柱加上持續剖析的各種專案。

如今的 Grafana 已經擁有了十餘種成熟專案，撇除甫剛整合的 Grafana Pyroscope，我們在接下來的內容中，將把重點放在 Grafana 精心為可觀測性三本柱規劃的 LGTM 技術線：

## Loki（Logging）

於 2018 年年底的 KubeCon 中首次亮相，由於當時的社群上並沒有一套低成本且如 grep 般簡單暴力的日誌服務，進而驅動 Grafana Loki 的誕生。

## Grafana（Visualization）

Grafana 誕生於 2013 年，。“我創建 Grafana 是為了做一些與 Kibana 類似的事情，但專注於時間序列指標。我的目標是讓更廣泛的受眾能夠訪問時間序列數據，更輕鬆地構建儀表板，使圖表和儀表板更具交互性。”Torkel Ödegaard 說道。

## Tempo（Tracing）

2021 年 Grafana Labs 在自家開發者大會上，發布了 Grafana 8.0 同時宣布 Grafana Tempo 進入正式版本 v1.0。

## Mimir、Prometheus（Monitoring）

2022 年 3 月 29 號 Grafana 宣布正式對外開源 Grafana Mimir， 完全兼容 Prometheus但其目標絕不僅僅成為一個更好的 Prometheus，它給自己的定位是成為可觀測性中 metrics 後端存儲的終極方案。

Grafana 允許團隊在一個地方對所有的可觀測性數據進行無縫的可視化和跳轉，並且提供開箱及用的設定以及低成本的輕量解決方案。

## LGTM 的授權走向

如今 Grafana 的 LGTM 技術線都已為 AGPLv3 授權。有趣的是，除了推出不久的 Mimir 原本就是 AGPLv3 外，Grafana、Loki、Tempo 都在 2021 年時從 Apache 2.0 轉為 AGPLv3。Apache 2.0 雖然是一個極為靈活的開源授權，但它可能不足以保護創作者免於某些大型企業的潛在剝削，尤其是商業使用卻未貢獻回開源社群的情況下使用其開源服務，形成一種現行的開源流氓。Grafana Lab 轉向 AGPLv3 可能是為了更好地防止此種狀況，並確保他們的開源工作能得到適當的尊重和回饋。他們堅定地認為，這一決策能夠平衡他們以開源為核心的精神，並確保長期的可持續發展。

## LGTM 的無縫跳轉可視化

Grafana 藉由專精的技術策略，不僅實現了Logging、Tracing、和Metrics的無縫可視化，更使各系統間的互動跳轉變得流暢。具體的技術細節如下所示：

1. **Metrics 至 Logs**：這是基於服務發現和一致的 Labels 實現。
2. **Logs 至 Metrics**：透過 LogQL 或 Labels，能有效地從日誌中提取 Metric 指標。
3. **Logs 至 Traces**：依賴關聯字段（如 traceID）或是自動化日誌來實現。
4. **Traces 至 Logs**：這一過程是基於 Traces 中的 Labels 來完成定位。
5. **Traces 至 Metrics**：利用來自 Spans 的 Metric 指標來實現。
6. **Metrics 至 Traces**：此過程由 Prometheus 的 Exemplars 來完成。

透過這些技術組合，Grafana 確保了可觀測性資料之間的流暢互動。

![https://ithelp.ithome.com.tw/upload/images/20230921/20149562soRDDxJZeI.png](https://ithelp.ithome.com.tw/upload/images/20230921/20149562soRDDxJZeI.png)

或許看到這裡的你們會越來越好奇，Grafana 是如何將如此複雜的數據及概念互相關聯、整合並進一步地實現可視化，在 Grafana 官方資源中可以找到非常多範例，用以學習且精進。

## Metrics to Logs

![https://ithelp.ithome.com.tw/upload/images/20230921/20149562pIMF6zKQG7.png](https://ithelp.ithome.com.tw/upload/images/20230921/20149562pIMF6zKQG7.png)

## Logs to Traces

![https://ithelp.ithome.com.tw/upload/images/20230921/20149562AurzVCUjeA.png](https://ithelp.ithome.com.tw/upload/images/20230921/20149562AurzVCUjeA.png)

## Traces to Metrics

![https://ithelp.ithome.com.tw/upload/images/20230921/20149562wCzAyIXLIl.png](https://ithelp.ithome.com.tw/upload/images/20230921/20149562wCzAyIXLIl.png)

# 結論

在之前的幾篇文章中，我們已經深入探討了「可觀測性」與「監控」之間的差異，也詳細分析了近年來可觀測性為何受到大量的關注。之後，我們進一步闡述了可觀測性的關鍵指標，進而轉向 Grafana 在這一領域的成就，特別是它的「LGTM」技術線 - 由 Loki、Ｇrafana、Tempo、與 Mimir 所組成。我們必須要認同，面對當今的分佈式系統複雜性，Grafana 將可視化三個字發揮得淋漓盡致。

現在，我們正準備進入充滿期待的實戰部分。在接下來的章節中，我會逐步指導大家如何搭建屬於自己的可觀測性平台，並利用 Grafana 的 LGTM 技術進行完美整合。

---
References

[Correlate Your Metrics, Logs & Traces with the curated OSS observability stack from Grafana Labs](https://www.youtube.com/watch?v=qVITI34ZFuk&ab_channel=Grafana)

[How to manage your metrics, logs, and traces using Grafana](https://grafana.com/go/webinar/getting-started-with-grafana-lgtm-stack/)

[一文带你了解 Grafana 最新开源项目 Mimir 的前世今生_Mimir_Grafana 爱好者_InfoQ写作社区](https://xie.infoq.cn/article/2723176da5693f6085c6b1e78)

[Grafana, Loki, and Tempo will be relicensed to AGPLv3 | Grafana Labs](https://grafana.com/blog/2021/04/20/grafana-loki-tempo-relicensing-to-agplv3/)