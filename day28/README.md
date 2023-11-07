# 可觀測性宇宙的第二十八天 - Grafana Mimir 介紹 - 監控指標的應許之地

# 概述

經過不懈的努力，我們已深入了解了 Grafana LGTM 家族中的大部分核心組件，從 Loki、Grafana 到 Tempo。現在，我們將進入其中最關鍵的一環：Grafana Mimir。在這個系列中，我們會重點探討Grafana Mimir，它是 Grafana Labs 新推出的開源、可擴展的時序指標存儲後端。作為Prometheus的可擴展長期存儲解決方案，它具有巨大的存儲容量。Mimir的架構與其他LGTM家族成員十分相似，且它仍在持續地進化和完善中。讓我們一同深入探索其奧秘。

![https://ithelp.ithome.com.tw/upload/images/20231013/20149562wBOvTIcyTq.png](https://ithelp.ithome.com.tw/upload/images/20231013/20149562wBOvTIcyTq.png)

# Grafana Mimir 介紹

Grafana Mimir 是 Grafana Labs 下一代開源、可擴展的時序資料庫存儲後端，專為指標設計，他被認為是 Cortex 分支與 Grafana Cloud 產品化的部分代碼結合而來，並使用 AGPLv3 授權。

透過 Grafana Mimir 我們可以實現近乎無限的基數，並且彈性的使用可水平擴展的組件架構，同時通過 Grafana Labs 成熟的大規模運營經驗，提供了生產環境等級的儀表板、告警，這一切都使得 Grafan Mimir 操作與維護都變得比 Cortex 更容易。更進一步地，Grafana Mimir 大大簡化配置並且移除了在 Cortex 難以維護的過時程式碼，在沒有技術包袱的情況下解決社群詬病許久的過度依賴第三方套件的問題，使得 Mimir 比 Cortex 更容易部署，並有更好、更全面的官方文件。

## 背景

當時 Tom Wilkie 於 2016 年在 Weaveworks 工作時，他和 Prometheus 的共同創始人兼維護者 Julius Volz 一起開始了 Cortex 專案。該專案的目的是構建一個可擴展的、與 Prometheus 相容的解決方案，設計成為 SaaS 供應模式。當 Tom Wilkie 加入 Grafana Labs 之後，他們與 Weaveworks 合作將 Cortex 移至開源社群，即 Cloud Native Computing Foundation（雲原生計算基金會）。Cortex 於 2018 年 9 月 20 日被接受為 CNCF 的沙盒專案，兩年後被提升為孵化專案。Grafana Labs 和 Weaveworks 都積極地貢獻，從 2019 年到 2021 年，Grafana Labs 的員工遠遠成為 Cortex 專案的最大貢獻者，佔據了約 87% 的提交。

Mimir 的願景不是成為「最好地可擴展 Prometheus」，而是成為「無論指標格式如何，最好的可擴展的時間序列資料庫」，Grafana Labs 執行長 Raj Dutt 真切地表示。

![https://ithelp.ithome.com.tw/upload/images/20231013/20149562hBDIGVhZvk.png](https://ithelp.ithome.com.tw/upload/images/20231013/20149562hBDIGVhZvk.png)

### 源自北歐的 Mimir 命名

Mimir 在北歐神話中是守護知識和智慧之泉的巨人，他的名字意為記憶與智者。北歐主神奧丁曾為了獲得智慧而來到泉水邊，以自己的右眼作為喝下泉水的代價，後來 Mimir 因捲入神族之間的戰爭而被斬斷頭顱，奧丁非常喜歡 Mimir，因此奧丁使用了魔法保存了 Mimir 頭部並使其能說話，日後奧丁更藉由與 Mimir 頭部諮詢來獲得意見，包括如何解決諸神黃昏的方法。這些故事都展示了知識和智慧在北歐神話中的重要性，以及人們為了追求它所願意付出的代價。Mimir的故事也是北歐神話中眾多複雜、有層次的故事之一，從 Grafana Mimir 的命名來看 Grafana Labs 對其在大規模監控領域的發展寄與強烈的想像空間。

# Grafana Mimir 組件

![https://ithelp.ithome.com.tw/upload/images/20231013/20149562golQP4I4e1.png](https://ithelp.ithome.com.tw/upload/images/20231013/20149562golQP4I4e1.png)

可以看到 Grafana Mimir 的架構基本上跟我們在前面章節認識過的 Loki 和 Tempo 相似，原因都是 Grafana Labs 借鑒于在 Prometheus 和 Cortex 專案上大力投資開發的經驗，使其重複利用這套讀寫分離的時序資料庫架構，快速孕育出各個偉大的專案。而有趣的是，因為 Grafana Mimir 是直接由 Cortex 分支出來建構而成，所以其在架構概念上高度相關，許多官方文件可以交互參閱無礙，演變成 2022 年府剛推出的 Grafana Mimir 反而是 LGTM 全家桶中，官方文件數量最多的專案。

其大多數組件是無狀態的，並且在進程重新啟動之間不需要任何數據狀態性。有些組件是有狀態的，依賴於持久性存儲以防止進程重新啟動之間的數據丟失。

## 寫入路徑架構 Write Path

![https://ithelp.ithome.com.tw/upload/images/20231013/20149562pk34dqdFhA.png](https://ithelp.ithome.com.tw/upload/images/20231013/20149562pk34dqdFhA.png)

在寫入路徑的架構下，每個組件的任務可以簡單理解為：

1. Ingester 組件:
    - 接收來自 Distributor 組件的指標。
    - 每一個推送請求都屬於一個租戶，且接收器會將接收到的樣本附加到存儲在本地磁盤上的特定於租戶的TSDB。
    - 接收到的樣本都會被保存在內存中並寫入到提前寫日誌 (WAL)。
    - 如果接收器突然終止，WAL 可以協助恢復記憶體中的序列。
    - 每當接收器接收到該租戶的第一批樣本時，都會延遲創建該租戶的 TSDB。
2. WAL (Write-Ahead Log, 提前寫日誌):
    - 當接收器接收到指標時，這些指標會被寫入WAL。
    - 記憶體中的樣本定期刷新到磁盤，並在創建新的 TSDB 塊時截斷 WAL。
    - WAL 存儲在持久硬碟上以便於在接收器突然終止時能夠恢復記憶體中的序列。
3. TSDB 時序數據庫:
    - 存儲時序數據的數據庫。
    - 每個新創建的塊都被上傳到長期存儲並保留在接收器中，直到配置的 retention-period 到期。
4. Compactor 組件:
    - 默認情況下，每個時序數據被複制到三個 Ingester 組件，且每個 Ingester 組件都將其自己的塊寫入長期存儲後端。
    - Compactor 組件會將來自多個 Ingester 組件的塊合併成一個單獨的塊，並刪除重複的數據。
    - Compactor 組件可以顯著減少了存儲使用。

此寫入架構中的每個角色都缺一不可，確保數據的一致性和可靠性。

## 讀取路徑架構 Read Path

![https://ithelp.ithome.com.tw/upload/images/20231013/20149562igDGbdG2VH.png](https://ithelp.ithome.com.tw/upload/images/20231013/20149562igDGbdG2VH.png)

而在讀取路徑的架構下，每個組件的主要職責和重點可以這樣簡單理解：

1. Query-Frontend 組件：
    - 負責接收任何對 Grafana Mimir 的查詢。
    - 將較長時間範圍的查詢分解成多個較小的查詢。
    - 檢查結果緩存。如果查詢的結果已經被緩存，則返回緩存的結果。
    - 將無法從結果緩存中獲取資料的查詢放入 Query-Frontend 組件的隊列中。
2. Query-Scheduler 組件(可選)：
    - 如果運行此組件，則 Query-Scheduler 組件將接管 Query-Frontend 組件維護查詢隊列的任物。
3. Queriers 組件：
    - 作為從隊列中拉取查詢任務的執行者。
    - 連接到 Store-Gateways 組件和 Ingester 組件以獲取執行查詢所需的所有數據。
    - 執行查詢後，將結果返回到 Query-Frontend 進行聚合。
4. Store-Gateway 組件：
    - Store-Gateway 組件是有狀態的，能夠從長期存儲（如 GCS、S3）中查詢數據塊。
    - 為了查詢時找到正確的數據塊，store-gateway 需要知道長期存儲後端的視圖，而其透過定期下載或掃描桶索引來保持此視圖更新。
5. Ingester 組及：
    - 當 Queriers 組件查詢時，會有部分請求來到 Ingester 組件查詢所需數據。

整體而言，在讀取路徑中，查詢首先被拆分和緩存檢查，然後由 Querier 組件從隊列中提取並執行，並最終將結果返回給客戶端。

## Prometheus 在 Grafana Mimir 中的角色

在 Grafana Mimir 長期儲存和水平擴展的解決方案中，Prometheus 扮演著一個核心的數據收集者和發送者的角色。

1. 數據收集：Prometheus 透過自己的擷取機制 (scraping) 進行資料收集。它會從各種數據源（例如：服務、節點或其他系統）抓取時序數據樣本。
2. 數據傳輸：在 Prometheus 的組態中，可以啟用 remote write 功能。當這個功能被啟用時，Prometheus 會將擷取到的時序數據樣本批量地通過其 remote write API 發送到 Grafana Mimir。這些數據在 HTTP PUT 請求的主體中被 Snappy 壓縮和 Protocol Buffer 格式化，使數據傳輸更高效。
3. 多租戶識別：為了區分來自不同來源或組織的數據，每個 HTTP 請求都需要帶有一個指定的租戶 ID 的標頭。這個租戶 ID 確保在 Mimir 中，不同的數據可以被適當地隔離和管理。請求的認證和授權是由外部的反向代理伺服器處理的，以確保資料的安全性。
4. 與 Mimir 的交互：
    - 寫操作：當 Prometheus 送出數據時，這些數據首先會被 Mimir 的 Distributor 組件接收。Distributor 組件負責收集和整合這些數據，然後再將它們發送到後端的儲存。
    - 讀操作：對於時序數據的查詢（如 PromQL 查詢），這些查詢請求首先會被 Mimir 的 Query Frontend 組件接收，並且再優化和分發查詢，使查詢更高效。

Prometheus 在 Grafana Mimir 解決方案中的主要角色是作為一個數據收集者和發送者，它不斷地從各種目標抓取時序數據，並將這些數據通過 remote write API 送入 Mimir，實現時序數據的長期儲存和水平擴展。

# 結論

Grafana Mimir，儘管在 2022 年才正式問世，但考量到其前身 Cortex 的淵源，它其實已具備深厚的背景和成熟的架構。作為追求極致擴展性的時序資料庫存儲後端，Grafana Mimir 在 Grafana LGTM 系列專案中，無疑是最為複雜的專案之一。然而，Grafana 團隊仍未忘初衷，依然為我們提供了單體模式或讀寫模式，大幅降低了初學者的入門門檻。若在使用中碰到疑難雜症，可能都可以從 Cortex 身上找到答案。既然這樣，我們還有什麼理由不實際來體驗一下 Grafana Mimir 的強大。

---

References

[How Grafana Mimir's store-gateway overcame out-of-memory errors](https://grafana.com/blog/2023/07/06/breaking-the-memory-barrier-how-grafana-mimirs-store-gateway-overcame-out-of-memory-errors/)

[Monitoring Kubernetes layers: Key metrics to know | Grafana Labs](https://grafana.com/blog/2023/01/25/monitoring-kubernetes-layers-key-metrics-to-know/)

[Video: How to migrate to Grafana Mimir in less than 4 minutes | Grafana Labs](https://grafana.com/blog/2022/04/25/video-how-to-migrate-to-grafana-mimir-in-less-than-4-minutes/)

[Grafana Mimir 调研](https://zhuanlan.zhihu.com/p/547989004)

[一文带你了解 Grafana 最新开源项目 Mimir 的前世今生_Mimir_Grafana 爱好者_InfoQ写作社区](https://xie.infoq.cn/article/2723176da5693f6085c6b1e78)

[Announcing Grafana Mimir, the most scalable open source TSDB in the world | Grafana Labs](https://grafana.com/blog/2022/03/30/announcing-grafana-mimir/)

[一文搞懂 Grafana Mimir-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2237893)

[使用 Grafana Mimir 实现云原生监控报警可视化](https://mp.weixin.qq.com/s/dHrwNsufHTe-pOi-v8xSJA)

[一文读懂 Prometheus 长期存储主流方案 - KubeSphere的个人空间 - OSCHINA - 中文开源技术交流社区](https://my.oschina.net/u/4197945/blog/5578010)