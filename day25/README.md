# 可觀測性宇宙的第二十五天 - Grafana Tempo 介紹 - 容納百川的盡頭

# 概述

在本系列章節中，我們即將踏入分佈式追蹤的神聖領域，這是一個伴隨著雲原生技術和 Kubernetes 容器編排工具的興起，在近十幾年逐漸形成的專業領域。隨著微服務的複雜性日益加劇，人們漸感其如同黑盒子般的除錯體驗，因此開始尋找各種理想的解決方案。分佈式追蹤雖具有比傳統日誌更高的概念門檻，但得益於社群的積極參與和不斷努力，它逐漸變得更加完善和易於理解。作為開源可觀測性的代表性組織，Grafana 團隊推出了其自家研發的 Grafana Tempo，這一工具秉持著易用性和高性價比的原則，與 Grafana 的可視化界面緊密整合，為我們帶來了前所未有的可觀測性體驗。面對 Grafana Tempo 如此多的優勢，我們還有什麼理由不被它深深吸引呢？

![https://ithelp.ithome.com.tw/upload/images/20231010/20149562Hm3aKLAWZB.png](https://ithelp.ithome.com.tw/upload/images/20231010/20149562Hm3aKLAWZB.png)
# Grafana Tempo 介紹

Grafana Tempo 是一款由 Grafana Labs 推出的開源大規模分佈式跟踪後端系統，以 Go 語言實現，設計之初就注重易用性和高效益。與市場上知名的 Jaeger、Zipkin 和 OpenTelemetry 等分佈式追踪協議系統相比，Tempo 以其深度集成 Grafana、Prometheus 和 Loki 的特點，以及僅依賴於對象存儲進行運行的低成本和簡單操作而脫穎而出。

Grafana Tempo支持與任何開源跟踪協議一起使用，無論是 Jaeger、Zipkin 還是 OpenTelemetry，都能與之兼容。它專注於批次提取數據，進行緩存，然後將數據寫入GCS，S3 或本地硬碟，高性價比且易於操作。相較於其他解決方案，Tempo以其簡單而高效的設計，能夠應對大容量的分佈式追踪需求。

為了增強搜索，大多數開源分散式追蹤框架都會對追蹤中的許多欄位進行索引，包括服務名稱、操作名稱、標籤和持續時間，為此大多依賴於 Elasticsearch 或 Cassandra 這類複雜索引系統。後來，Grafana Labs 意識到，追蹤信號不一定需要使用索引，通過日誌和樣本就可以發現追蹤信號，因此不必為其添加索引花費額外的成本。基於這些考慮，他們創建了 Grafana Tempo，一個精簡、簡單且按 id 進行追踪的存儲機器，滿足用戶對於分佈式追踪的各種需求。

## 分佈式追蹤是什麼？

分佈式追踪是一項技術，專門用於搜集應用程式內部與端到端請求相關的資料，以實現對整個系統的監控和分析。在這個過程中，一個追踪會被分解成一個或多個「跨度 Span」，每個跨度代表請求中的一次呼叫，這可能是一個微服務，或者是微服務內的一個函數。追踪的基本結構如下所示：

![https://ithelp.ithome.com.tw/upload/images/20231010/20149562GR1IljZKUV.png](https://ithelp.ithome.com.tw/upload/images/20231010/20149562GR1IljZKUV.png)

追踪的起點是根跨度，其後可以跟隨一個或多個子跨度。這些子跨度可根據呼叫堆疊的深度進行嵌套。每個跨度都會包含服務名稱、操作名稱、持續時間，還可以選擇性地附帶額外的元數據。

## 分佈式追踪工具主要架構

![https://ithelp.ithome.com.tw/upload/images/20231010/20149562OJuVIeV9Q9.png](https://ithelp.ithome.com.tw/upload/images/20231010/20149562OJuVIeV9Q9.png)

現有的分佈式追踪工具的工作方式大致相同，通常使用相同的架構組件。然而，在實現方式和使用的追踪協議上，它們之間可能存在差異。以下將以Grafana Tempo為例來說明：

上圖顯示了 Grafana Tempo 如何與追踪協議、對象存儲和 Grafana 一起工作。與大多數分布式追踪生態系統一樣，Grafana Tempo 依賴四個主要組件來發揮其用途：

1. 儀表（Instrumentation）：簡單來說，這個組件提供了被收集的遙測數據，在這種情況下是追踪信號（trace），但它也可以生成其他工具使用的指標和日誌。
2. 管道（Pipeline）：分布式追踪工具通常同時從多個來源接收遙測數據，因此需要隨著應用規模的擴大，優化工具使用的資源。為此，追踪管道執行諸如數據緩衝、追踪批次處理和後續路由到存儲後端等功能。對於這個重要任務，Grafana Tempo 可以使用 Grafana Agent，這是在 Grafana 生態系中用來抓取指標和日誌的相同遙測收集器。
3. 存儲後端（Storage Backend）：收集到追踪後，它們必須被存儲，以供稍後可視化和分析。我們需要一個能夠隨著應用的擴展而擴展，同時在這個過程中不消耗太多資源的存儲後端。Grafana Tempo 採取了一種創新的方法：它不依賴資源負擔沈重的資料庫，而是使用如 Amazon S3、Google Cloud Storage 和 Microsoft Azure Blob Storage 等流行的對象存儲解決方案作為其追踪存儲後端。如果需要，可以使用 Memcached 或 Redis 來優化查詢性能。
4. 可視化層（Visualization Layer）：Tempo 可以與 Grafana 完美結合用於可視化並創建在 Kubernetes 中互相關聯指標、日誌和追踪的動態儀表板。

這些組件共同形成了 Grafana Tempo 分布式追踪工具的基礎，使其能夠靈活應對各種應用場景和需求。

## 分布式追踪工具比較

市面上有眾多工具值得我們深入了解幾個最受歡迎的開源分佈式追踪解決方案，它們都是為容器化運行環境所設計且工作原理都相似，因此對它們進行高層次的比較相對直接。

![https://ithelp.ithome.com.tw/upload/images/20231010/20149562Wjp4CJ1c4d.png](https://ithelp.ithome.com.tw/upload/images/20231010/20149562Wjp4CJ1c4d.png)

- Zipkin：開源分布式追踪解決方案的先驅之一。最初由 Twitter 開發，並用 Java 編寫，其特性包括支持 Cassandra 和 Elasticsearch 作為存儲後端。
- Jaeger：一個比 Zipkin 較新的項目。它最初由 Uber 創建，但後來成為了 Cloud Native Computing Foundation（CNCF）的孵化器項目。Jaeger 使用 Golang 編寫，就像 Zipkin 一樣，它支持 Cassandra 和 Elasticsearch 作為存儲後端。
- Grafana Tempo：由 Grafana Labs 開發，Tempo 從一開始就被構建為高度可擴展、高性價比且靈活。它還支持對象存儲追踪數據，並且與 Zipkin、Jaeger 和 OpenTelemetry 協議兼容。

# Grafana Tempo 組件

當我們深入了解 Grafana Tempo 的各個組件前，值得一提的是 Grafana Tempo 和 Grafana Loki 甚至是 Grafana Mimit 在組件概念上都展現出了高度的相似性，這種相似性意味著，當我們學習或熟悉了其中一套工具後，將能夠非常迅速和輕松地掌握另一套，並且大幅減輕了我們的維護難度及時間成本。

![https://ithelp.ithome.com.tw/upload/images/20231010/20149562qg7IA0IJwX.png](https://ithelp.ithome.com.tw/upload/images/20231010/20149562qg7IA0IJwX.png)

## Distributor

- 負責接收來自不同格式（如 Jaeger、OpenTelemetry、Zipkin）的跨度。
- 使用分佈式一致性哈希環，通過哈希 traceID 將跨度路由到 Ingesters。
- 使用 OpenTelemetry Collector 的 Receiver Layer。

## Ingester

- 將追蹤信號打包成塊，創建布隆過濾器和索引，並將其存儲到後端。
- 后端的塊按照特定的布局生成，如下形式呈現：

```php
<bucketname> / <tenantID> / <blockID> / <meta.json>
                                      / <index>
                                      / <data>
                                      / <bloom_0>
                                      / <bloom_1>
                                        ...
                                      / <bloom_n>
```

## Query Frontend

- 負責為傳入的查詢切割搜索空間。
- 切分 blockID 空間成多個分片並將這些請求加入隊列。
- 提供HTTP端點：GET /api/traces/<traceID>。

## Querier（查詢器）

- 負責在ingesters或後端存儲中查找請求的追踪ID。
- 根據參數，從 ingesters 和後端存儲中檢索數據。
- 提供HTTP端點，但通常不直接使用，查詢應發送到查詢前端。

## Compactor（Optional）

- 負責減少後端存儲中塊的總數量。
- 通過流式處理，將塊從後端存儲拉出並寫入。

## Metrics generator（Optional）

Metrics-generator 是一個可選的 Tempo 元件，它從 Ingester 組件的追蹤數據中產生對應指標。如果存在，Distributor 會將接收到的跨度寫入Ingester 和 Metrics generator。指標產生器使用 Prometheus  Remote Write 處理跨度指標並將指標寫入 Prometheus 資料源。我們將隨後對其進行詳細介紹。

# ****Server-Side Metrics -**** Metrics generator

服務端指標是一種從將被寫入的指標中額外產生 Metrics 的功能，而 Ｍetrics-generator 組件就是擔任此角色的存在，Distributor 組件會同時將接受到的跨距寫入 Ingester 及 Metrics-generator 中，目的在於使 Metrics-generator 專注於處理生成寫入 Metrics 的分工職責。

Metrics-generator 內部會運作一組 Processor（處理器），每一組處理器都可以透過將被寫入的指標再產生出額外附加價值的 Metrics。

目前，提供的以下處理器可以使用：

- Service Graph 服務圖
- SpanMetrics 跨度指標

![https://ithelp.ithome.com.tw/upload/images/20231010/20149562HJ0aqfx81w.png](https://ithelp.ithome.com.tw/upload/images/20231010/20149562HJ0aqfx81w.png)

## SpanMetrics 跨度指標

Span Metrics 處理器從接收到的追蹤數據中生成指標，包括請求、錯誤和持續時間（RED）指標。

Span Metrics 會生成兩種指標：

- 一個計數器，用於計算請求的次數
- 一個直方圖，用於追踪所有請求的持續時間分佈

即便我們已有監控指標，Span Metrics 能為系統提供更深入的監控。生成的指標將展示追踪在我們的應用服務中傳播的程度，從而提供對應用層面的洞察。

最後，但同樣重要的是，Span Metrics 降低了使用 exemplars 的入門門檻。Exemplar 是在給定時間間隔內進行的測量的具體追踪代表。由於追踪和指標在 Metrics-generator 中共存，因此可以自動添加 exemplars，為這些指標提供額外的價值。

![https://ithelp.ithome.com.tw/upload/images/20231010/20149562T4bTRLqDWD.png](https://ithelp.ithome.com.tw/upload/images/20231010/20149562T4bTRLqDWD.png)

## Service Graph View 服務視圖

Grafana Tempo 中的 Service Graph 利用由 Metrics-generator（或 Grafana Agent）生成的指標，展示跨度請求率、錯誤率、持續時間以及服務圖。一旦完成設置要求，這個預配置的視圖便立即可用。

通過使用服務圖視圖，我們可以：

- 發現一致性出錯的跨度以及它們發生的頻率。
- 獲取有關我們服務中所有跨度呼叫的整體頻率。
- 確定服務中最慢的查詢完成所需的時間。
- 根據率、錯誤和持續時間值（RED信號）檢查包含過濾特定跨度的所有追踪數據。

![https://ithelp.ithome.com.tw/upload/images/20231010/20149562lsmxHteZa3.png](https://ithelp.ithome.com.tw/upload/images/20231010/20149562lsmxHteZa3.png)

# 結論

在仔細了解 Grafana Tempo 之後，我們可以看到，Grafana 團隊一如既往地帶給我們超乎預期的驚喜。他們不僅解決了現有分佈式追蹤工具在成本上的挑戰，還借鑒了自身的開源工具架構來開發出 Grafana Tempo。更值得一提的是，他們在這個基礎上創建了前所未有的 Metrics generator 概念，更進一步提升了每個追踪信號的價值。

對於一直堅持到現在同學，無論是感受到了耳目一新的啟發，還是覺得學習有些吃力，打起精神！我們現在即將進入實戰演練環節，探索 Grafana Tempo 的神秘面紗！