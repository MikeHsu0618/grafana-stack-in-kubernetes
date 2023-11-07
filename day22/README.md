# 可觀測性宇宙的第二十二天 - Grafana Loki 介紹 - 性價比的王者

# 概述

在先前，我們成功地運用 Prometheus 進行 Kubernetes 叢集的基本 APM 監控，這無疑是一個很棒的第一步。但當 Prometheus 的監控指出某個服務的 CPU 和記憶體使用率驟增，甚至該服務不斷重新啟動時，我們的第一反應通常是查詢日誌以了解是否存在異常。此情境突顯了，除了監控工具外，日誌可能是最能協助我們了解服務間互動的可觀測性工具。

在接下來的章節中，我們將深入探討 Grafana 團隊為我們提供的日誌解決方案 Grafana Loki 的基礎概念與架構。

![https://ithelp.ithome.com.tw/upload/images/20231007/20149562680S56tOoa.png](https://ithelp.ithome.com.tw/upload/images/20231007/20149562680S56tOoa.png)

# Grafana Loki 是什麼

## 背景

在很早之前，Grafana 團隊發現很多人都跟他們一樣認為當今社群上，沒有一套足夠穩定且費用低廉的日誌工具，甚至吹起一股減少日誌量的反模式的聲音，而 Grafana 團隊開始有幾個對日誌的想法出現：

- 保持簡單，像是 Grep 一樣。
- 日誌應該是成本低廉的，不應要求任何人減少日誌量。
- 指標 (Metrics)、日誌 (Logs)、追踪（traces）需要一起工作。

![https://ithelp.ithome.com.tw/upload/images/20231007/20149562TJ5wDyOkb9.png](https://ithelp.ithome.com.tw/upload/images/20231007/20149562TJ5wDyOkb9.png)

而這些社群上對日誌的需求，催生了 Grafana Loki 的誕生。

## 介紹

Loki 於 2018 年 12 月在北美的 KubeCon 上推出，是一款由 Grafana 團隊開源的高度可擴展、高可用且支持多租戶的日誌聚合系統。該系統主要旨在處理分布式大型系統中的大量日誌問題。Loki 不僅擁有分佈式架構，還與 Prometheus 和 Grafana 緊密結合，使其能迅速處理大量的日誌數據。此項目受到 Prometheus 的啟發，其官方定義為：“Like Prometheus, But For Logs”。

與其他日誌聚合系統相較，Loki展現出以下特點：

1. **可擴展性**：Loki 的設計目的是實現高擴展性，從小型樹莓派運行到每天實務上數 PB 的日誌都得以勝任。在其最常見的“簡單可擴展模式”配置中，Loki 將請求初始化對於獨立的讀取和讀取路徑，從而可以獨立地進行擴展，這導致可以快速適應於任何時間的工作負載的大型安裝。如果需要，Loki 的每個組件也可以作為在 Kubernetes 內原始運行的微服務來運行。
2. **多組戶性**：Loki 允許多個租戶共享單個 Loki 實例。在多租戶性中，每個租戶的數據和請求完全與其他租戶隔離。通過分配租戶 ID 在代理中配置多租戶性。
3. **第三方集成**：多個第三方代理（客戶端）通過插件支持Loki。這允許我們保留現有的可觀察性設置，同時也向Loki 發送日誌。
4. **高效存儲**：Loki 以高度壓縮的數據塊形式存儲日誌數據。同樣，由於 Loki 僅索引一組標籤，因此它比其他日誌聚合工具要小。
5. **LogQL**：Loki的查詢語言 - LogQL 是Loki的查詢語言。已經熟悉 Prometheus 查詢語言 PromQL 的使用者會發現 LogQL 既熟悉又靈活，非常適合對日誌進行查詢。這種語言還可以從數據日誌中生成指標，這是超越聚合的強大功能。
6. **告警**：Loki 包括一個稱為 ruler 的組件，可以持續對我們的日誌進行查詢評估，並根據結果進行操作。這允許我們監控日誌中的異常或事件。Loki 與 Prometheus Alertmanager 或 Grafana 內的警報管理器整合。
7. **Grafana 整合**：Loki 與 Grafana、Mimir 和 Tempo 整合，提供完整的可觀察性，以及日誌、指標和跟踪之間的無縫關聯。

![https://ithelp.ithome.com.tw/upload/images/20231007/20149562t5wCrafcLb.png](https://ithelp.ithome.com.tw/upload/images/20231007/20149562t5wCrafcLb.png)

# 為什麼要使用 Loki？

Loki是不僅具有上面提到的技術優勢，還為組織帶來了許多不容忽視的好處。以下是使用 Loki 的主要理由：

1. **成本效益**：Loki 僅索引元數據，不對整行日誌進行索引，大幅減少存儲和運營成本。這種方法使索引大小約為數據的 1/10000，大大節省了經費。
2. **存儲策略**：Loki 使用物件存儲（如AWS S3）來儲存日誌，這比需要大量硬碟的存儲方案成本低得多。
3. **運營簡便**：Loki 避免了依賴結構化日誌，從而減少了與前處理工具的配合和更改所帶來的問題。此外，由於不需要結構化模式，Loki 更靈活可以應對多種日誌格式和用例。
4. **團隊協作**：Loki 減少了開發人員和運營人員的認知負擔，允許他們專注於自己的主要工作，並在需要時輕鬆查詢日誌。
5. **廣泛應用**：Loki 可以在單一機器上運行，並根據需要水平擴展。它可以適應從小型設備如 Raspberry Pi 到大型集群的各種使用場景。
6. **與 Prometheus 的無縫整合**：Loki的日誌查詢語言 LogQL 借鑒了 Prometheus 的 PromQL，使用者可以在日誌和指標之間無縫切換。

總之，Loki 的優點還是顯而易見，它是一款快速、成本效益高、高度可擴展的日誌工具，能夠與 Prometheus 標籤無縫整合，並允許用戶在指標和日誌之間輕鬆切換。

> 相信很多人會好奇 Loki 與另一個常被拿來做日誌工具 Elasticsearch 的各種比較，剛好小弟正好在工作內容上的一部份是在處理 Elastricsearch 轉移到 Grafana Loki 的遷移工程，由於三十天的篇幅不太夠，如果有興趣的話的同學也可以敲碗讓我知道。
>

# 組件 Components

![https://ithelp.ithome.com.tw/upload/images/20231007/20149562BGHXXTRYHB.png](https://ithelp.ithome.com.tw/upload/images/20231007/20149562BGHXXTRYHB.png)

## ****Distributor****

Distributor 是負責處理客戶端傳入的數據流的服務。它是寫入日誌數據的第一站。當分發器接收到一組數據流時，每個流都會進行正確性驗證，並確保其在配置的租戶（或全局）限制內。然後，有效的數據片段被拆分成批次並平行發送到多個 Ingesters。

> 放置一個負載均衡器在 Distributor 前面是很重要的，以確保正確平衡流量。
>

分發器是一個無狀態的組件，這使得它易於擴展，並從寫入路徑上的最關鍵組件 Ingesters 中卸載盡可能多的工作，並且帶有各種預處理、速率限制的作用，獨立地擴展這些驗證操作的能力意味著 Loki 也可以保護自己不受拒絕服務攻擊，否則可能會使 Ingesters 超載。Distributor 就像在前門的保鑣，確保每個人都穿著得體且有邀請函。這也允許我們根據我們的複制因子擴展寫操作。

以上提到的細節如下：

1. 預處理：
    - 功能：標籤規範化，確保標籤結構的一致性。
    - 方法：對標籤進行排序，例如將 {foo="bar", bazz="buzz"} 轉換為 {bazz="buzz", foo="bar"}。
    - 好處：確保 Loki 可以確定地進行數據緩存和哈希。
2. 速率限制：
    - 功能：根據租戶設定限制傳入日誌的速率。
    - 方法：檢查每個租戶的速率限制，並根據分發器的數量調整。
    - 好處：允許在叢集級別設定速率限制；靈活調整分發器數量以應對不同負載。
3. 轉發：
    - 功能：將驗證過的數據轉發至 Ingester。
    - 特點：分發器是前置驗證單元，轉發是最終步驟。
    - 好處：確保只有合格的數據進入 Ingester，保障數據品質。
4. 複製因子：
    - 功能：確保數據的高可用性和持久性。
    - 特點：默認值為3，即數據將被分發到 3 個不同的 Ingester。
    - 好處：即使單個ingester失效，數據仍然安全。
5. 哈希：
    - 功能：決定哪些 Ingester 接收特定數據流。
    - 方法：使用一致的哈希和複製因子。
    - 好處：確保數據均勻分布，提高系統的擴展性。
6. Quorum 一致性：
    - 功能：保證查詢結果的一致性。
    - 方法：所有分發器使用相同的哈希環，並使用Dynamo風格的quorum一致性。
    - 好處：提供可靠的查詢結果，即使在部分節點失效的情況下也能保持一致性。

## ****Ingester****

Ingester 是Loki系統中的核心組件，其主要職責是在寫路徑中將日誌數據寫入長期存儲後端（如 DynamoDB, S3, GCS 等）並在讀路徑上為內存查詢返回日誌數據。

在 Ingester 還有幾個重要細節如下：

1. 日誌流與塊：
    - 接收的每個日誌流都會在內存中形成多個“塊”。
    - 在可配置的時間間隔後，這些“塊”會被刷新到後端存儲。

      ![https://ithelp.ithome.com.tw/upload/images/20231007/201495620RVBQgkx2G.png](https://ithelp.ithome.com.tw/upload/images/20231007/201495620RVBQgkx2G.png)

2. 塊的壓縮與只讀標記：
    - 當達到容量、超時或發生刷新操作時，當前塊被壓縮且標記為只讀。
    - 壓縮的塊被新的可寫塊替代。

      ![https://ithelp.ithome.com.tw/upload/images/20231007/20149562DnTmeCbNh6.png](https://ithelp.ithome.com.tw/upload/images/20231007/20149562DnTmeCbNh6.png)

3. 數據安全性：
    - 如果Ingester突然崩潰或退出，未刷新的數據會丟失。
    - Loki通常配置多個副本（通常為3）來減少這種風險。
    - 為了進一步增強數據的持久性，Loki配備了提前寫日誌，可在重啟後回放。
4. 哈希與去重：
    - 每當數據被刷新到永久存儲時，基於其租戶、標籤和內容進行哈希。
    - 這確保相同的數據不會被重複寫入，但在寫入副本失敗的情況下，後端存儲可能存在多個不同的塊對象。
5. 時間戳排序：
    - Loki 默認接受非順序寫入。
    - 當配置為不接受這種寫入時，Ingester 確保接收的日誌按時間戳升序。
    - 對於相同的納秒時間戳，如果內容與前一行相同則忽略，不同則接受。

## Compactor

Compactor 在 Loki 系統中扮演著關鍵角色，主要負責索引的去重、數據去重及保留工作。它不僅提高了存儲效率，還為使用者提供了一個更加整潔和有效的查詢體驗。

1. 核心功能：
    - 去重：Compactor 能夠識別和消除重複的索引，從而優化存儲空間。
    - 數據保留：它允許精確的數據保留策略。當使用 Compactor 進行數據保留時，不再需要 Table Manager。
2. 運作策略：
    - 建議只運行一個Compactor實例，即作為單例模式運行。
    - 其去重和保留策略是冪等的，即重啟後，Compactor會繼續未完成的任務。
3. 運作算法：
    - Compactor 會循環進行去重和數據保留，並遵循特定的間隔策略。若落後於預定的間隔，它會儘快進行操作。
    - 具體算法包括：為每天的每個表進行去重，識別需要刪除的塊，保存其引用，然後上傳修改過的索引文件。
4. 數據保留注意事項：
    - 雖然索引會適用保留算法，但塊不會立即被刪除。它們將被異步刪除。
    - 刪除的塊會經過一段延遲期，確保不會立即被刪除，因為這能防止配置錯誤或者其他組件還在引用舊的塊，從而確保查詢的正常執行。

通過Compactor，Loki 系統確保了高效的存儲使用和流暢的查詢體驗，並且確保在配置或操作上的任何失誤不會對數據造成無法修復的損害。

> 注意：一次只能運行 1 個壓縮器實例，否則可能會產生問題並可能導致數據丟失。
>

## ****Query frontend****

Query frontend 在確保查詢效率和系統的穩定性上起著至關重要的作用。以下是主要功能：

1. 排隊機制：
    - 資源保護：查詢前端保證在遇到可能導致記憶體溢出的大查詢時，能夠在失敗後重試，允許管理員對查詢的記憶體進行保守配置，或者樂觀地並行運行更多小查詢。
    - 查詢平衡：使用先進先出（FIFO）隊列，確保多個大請求不會在單一的查詢器上串行執行，而是均勻分布在所有查詢器上。
    - 公平調度：防止單一租戶對其他租戶進行服務拒絕攻擊（DOS），通過在租戶之間公平地安排查詢來實現。
2. 查詢分割：
    - 為了避免大規模的查詢（例如跨多天的查詢）在單一查詢器中導致記憶體問題，查詢前端將它們分割為多個小查詢。這些小查詢並行在下游的查詢器上執行，然後將結果重新組合。
3. 結果緩存：
    - 查詢前端支持緩存指標查詢結果並在後續查詢中重用。若緩存的結果不完整，查詢前端將計算所需的子查詢並在下游查詢器上並行執行它們。此外，查詢前端可以選擇性地與其步驟參數對齊查詢，以提高查詢結果的可緩存性。目前，結果緩存與所有Loki緩存後端兼容，包括memcached、redis和內存緩存。

## ****Querier****

Querier 負責解讀和執行使用者基於 LogQL 查詢語言的查詢要求，其工作不僅包括從寫入緩存的 Ingester 提取日誌，還涉及從長期存儲中提取資料。

以下是其工作流程及核心功能的概述：

1. 資料來源優先順序：當收到查詢要求時，查詢器首先向所有 Ingester 發起請求，尋找其內部的即時緩存數據。只有在找不到所需數據時，查詢器才會退而向後端存儲發起相同的查詢請求。
2. 數據去重：由於Loki的系統結構特點，查詢過程中可能會接收到重複的數據。為解決此問題，查詢器內置了一套去重機制，能夠識別和過濾出具有相同的納秒級時間戳、標籤集和日誌消息的重複數據。
3. 讀取路徑中的複製因子：查詢器在執行查詢時會考慮到系統的複製因子。舉例來說，當複製因子設為 3 時，我們至少需要兩次查詢執行才能確保數據的完整性和準確性。

# 存儲系統 Storage

## 存儲模式 Storage mode

Grafana Loki 區別於其他日誌系統，其核心概念是只索引有關日誌的標籤集，與 Prometheus 標籤相似。而日誌數據則被壓縮並存儲在如S3或GCS的對象存儲中，或者直接在本地文件系統上。然而，日誌數據本身被儲存分成「索引（index）」以及「塊（chunk）」兩部份儲存，兩者可以被儲存於對象存儲服務（如 GCS、S3）中，甚至是存儲在本地文件系統（filesystem）中。不論如何，輕量索引加上高度壓縮的塊簡化了操作並且顯著降低了 Loki 的成本。

![https://ithelp.ithome.com.tw/upload/images/20231007/20149562x1Z5s4exYr.png](https://ithelp.ithome.com.tw/upload/images/20231007/20149562x1Z5s4exYr.png)

在 Loki 2.0 之前，塊跟索引數據需要各自存儲在單獨的後端中，索引就存在 Nosql / Key-Value 數據庫而塊儲存在對象存儲服務或文件系統。在 Loki 2.0 帶來了一個名為「boltdb-shipper」的索引機制，這被稱為 Single Store，僅需一個對象存儲即可存放索引和數據塊，大大的減少 Loki 本身對其他服務的依賴。而在 Loki 2.7 引入實驗性質的 TSDB 模式，在 Loki 2.8 之後取代 boltdb-shipper 正式成為 Loki 推薦的持久儲存模式。

1. 索引存儲：
    - Cassandra：儘管已被棄用，但仍是 Loki 的可能數據塊存儲選項。
    - 雲端 Nosql 服務：如 BigTable 和 DynamoDB，均已被棄用。
    - BoltDB：用於概念驗證部署和 Loki 的開發。
2. 存儲後端選項：
    - 文件系統：最簡單的數據塊後端，適用於單一二進制部署和Loki的本地開發。
    - 對象存儲：包括GCS、S3、Azure Blob Storage、IBM COS、Baidu BOS 和 Alibaba OSS 等。
    - 其他選擇：如 MinIO 等，實現 S3 API 的服務。
3. Single Store 模式：
    - TSDB：從 Loki 2.8 開始，提供與 boltdb-shipper 相同的功能，同時減少總擁有成本並提高查詢性能。
    - BoltDB：在開發期間稱為 boltdb-shipper，將塊存儲用於「塊」和「索引」，只需要一個存儲來運行 Loki，且性能可與專用索引類型相媲美。

> TSDB 僅僅在兩個小版本就取代 boltdb-shipper，受益於 Prometheus 對於 TSDB 社群大規模成熟使用經驗，使其的可以同時實現減少存儲成本並提升查詢效能的作用。在後面我們將搶先體驗 TSDB 實戰，有機會的話，我們還可以深入探討 TSDB 在哪個地方彌補的 boltdb-shipper 的不足。
>

## 存儲格式 ****Storage schema****

![https://ithelp.ithome.com.tw/upload/images/20231007/20149562XJ5ssaH8qR.png](https://ithelp.ithome.com.tw/upload/images/20231007/20149562XJ5ssaH8qR.png)

Loki 的目標是實現向後兼容，保留在其內部進行開發的空間，並且開放促進更好、更高效的存儲開發，所以 Loki 允許我們迭代升級到這些新的存儲模式，並起可以在底層無感切換使用，這使升級底層存儲模式這類的大工程變成輕而易舉。例如上圖表示：從 2021 年 6 月 1 號後的區間使用 v11 版本存儲格式，但 2022 年 4 月 10 號之後使用 v12 版本存儲格式。

# 金絲雀 ****Canary****

Loki Canary 是一個獨立的應用服務，用於審核 Grafana Loki 集群的日誌捕獲性能。Canary 服務將會生成日誌像 Loki 叢集正常發送，並且另以 Websocket 與 Loki 叢集通訊，以便形成有關叢集性能的資訊。

![https://ithelp.ithome.com.tw/upload/images/20231007/20149562r8pcdFh7Ws.png](https://ithelp.ithome.com.tw/upload/images/20231007/20149562r8pcdFh7Ws.png)

Canary 運作相當簡單，可以配置時間間隔和可配置的大小寫入日誌，這些日誌由 promtail 讀取並發送到 Loki。同時，Canary 打開與 Loki 的 Websocket 連接，並為其寫入的日誌提供即時的 tail 日誌。Canary 生成一系列指標，然後將其抓取並由 Prometheus 發出警報。這些指標包括丟失、無序或重複的任何日誌的計數器。此外，通過比較金絲雀放入日誌中的時間戳，而 Grafana 團隊還創建了從創建日誌到通過 Websocket 接收日誌的響應時間的直方圖。

![https://ithelp.ithome.com.tw/upload/images/20231007/20149562bfg8Osbf8S.png](https://ithelp.ithome.com.tw/upload/images/20231007/20149562bfg8Osbf8S.png)

上圖顯示 Loki 在 9:40 左右執行部署，導致延遲略有增加，並且 Websocket 連接上丟失了一些日誌。然而，對丟失日誌在經過後續查詢證實它們實際上安全地存儲在 Loki 中，並且在部署過程中沒有丟失任何日誌！

# 多租戶 ****Multi-tenancy****

![https://ithelp.ithome.com.tw/upload/images/20231007/20149562AGWyhesUdB.png](https://ithelp.ithome.com.tw/upload/images/20231007/20149562AGWyhesUdB.png)

Grafana Loki 是一個擁有多租戶功能的系統，其中租戶 A 的請求和數據與租戶 B 可以是隔離的。如果我們在 Grafana 中需要特別指定租戶 ID 才能 Ｍ 向 Loki API 的請求，因為開啟多組戶功能後，每個請求需要包含一個 HTTP Header「X-Scope-OrgID」以識別該請求的租戶。

> 當配置為 auth_enabled: false 時，Loki使用單一租戶，並且在Loki API請求中不需要「X-Scope-OrgID」作為 Header 值。單一租戶的ID將是字符串fake。
>

# 結論
在本篇文章中，我們將深入探討 Grafana Loki 及其組件和核心概念。Loki 在 Tempo、Mimir 系統中是最為穩定和成熟的，其內部蘊含著豐富且值得深入討論的細節。由於 Grafana 的官方文件以其簡約而聞名，許多資料過於零碎發散，需要不斷的廣泛搜尋拼湊，在撰寫本文時我嘗試以最合理的順序，精練地介紹 Loki 的關鍵觀念。其他更細緻的部分，也許我們將在實戰演練中探討，或可能在進行效能調校時一同探索。下面，現在就讓我們進入實戰演練，親身體驗 Grafana Loki 的強大吧。

---

References

[Loki’s Path to GA: Loki-Canary Early Detection for Missing Logs | Grafana Labs](https://grafana.com/blog/2019/07/18/lokis-path-to-ga-loki-canary-early-detection-for-missing-logs/)

[Loki Canary |  Grafana Loki documentation](https://grafana.com/docs/loki/latest/operations/loki-canary/)

[Grafana Loki Storage |  Grafana Loki documentation](https://grafana.com/docs/loki/latest/operations/storage/)

[Grafana 系列文章（十）：为什么应该使用 Loki - 掘金](https://juejin.cn/post/7197239663102705724)

[All the non-technical advantages of Loki: reduce costs, streamline operations, build better teams | Grafana Labs](https://grafana.com/blog/2020/09/09/all-the-non-technical-advantages-of-loki-reduce-costs-streamline-operations-build-better-teams/)

[Grafana, Loki, and Tempo will be relicensed to AGPLv3 | Grafana Labs](https://grafana.com/blog/2021/04/20/grafana-loki-tempo-relicensing-to-agplv3/)

[Multi-tenancy |  Grafana Loki documentation](https://grafana.com/docs/loki/latest/operations/multi-tenancy/)

[How to run faster Loki metric queries with more accurate results | Grafana Labs](https://grafana.com/blog/2023/07/05/how-to-run-faster-loki-metric-queries-with-more-accurate-results/)

[How to collect and query Kubernetes logs with Grafana Loki, Grafana, and Grafana Agent | Grafana Labs](https://grafana.com/blog/2023/04/12/how-to-collect-and-query-kubernetes-logs-with-grafana-loki-grafana-and-grafana-agent/)

[Watch: 5 tips for improving Grafana Loki query performance | Grafana Labs](https://grafana.com/blog/2023/01/10/watch-5-tips-for-improving-grafana-loki-query-performance/)

[10 things you didn’t know about LogQL | Grafana Labs](https://grafana.com/blog/2022/05/12/10-things-you-didnt-know-about-logql/)

[Grafana 系列文章（九）：开源云原生日志解决方案 Loki 简介 - 掘金](https://juejin.cn/post/7196699593836937276)

[[DevOps]Grafana + Loki + Promtail Logging system 小試牛刀](https://medium.com/@NeroHin/devops-grafana-loki-promtail-logging-system-小試牛刀-b922ba8ed0d8)

[Grafana Loki 简明教程](https://www.qikqiak.com/post/grafana-loki-usage/)