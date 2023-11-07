# 可觀測性宇宙的第十六天 - Grafana 入門介紹

# 概論

在上一章，我們深入探討了 Prometheus 的實作細節，而其中不難發現，Prometheus 與 Grafana 之間有著密不可分的關聯。Grafana 似乎就像是專為 Prometheus 而生的存在，它的功能遠不止於此。Grafana 的真正強大之處不僅在於與 Prometheus 的深度整合，更在於其廣泛的適應性和可擴展性，能與多種數據源無縫接軌，提供使用者一個多元、全面的監控視覺體驗。

接下來我們將會正式進入 Grafana 篇章，帶領大家認識 Grafana 為什麼可以成為開源可視化工具中數一數二的選擇，以及其如何透過豐富的儀表板數據，貫穿我們所有介紹的監控工具核心。

# Grafana 是什麼

![https://ithelp.ithome.com.tw/upload/images/20231001/20149562JOBWv0Mcll.png](https://ithelp.ithome.com.tw/upload/images/20231001/20149562JOBWv0Mcll.png)

Grafana 的故事始於 2013 年，當時作者 Torkel Ödegaard，因為對當下的 Graphite 監控使用介面感到不滿，後來他發現了現在依然熱門的 ELK Stack 可視化工具 Kibana，並分叉了它，他的目標是創建一個能夠整合多種數據源，且具有高度客製化的可視化界面。於是Grafana 便誕生了，並迅速獲得了開源社群的熱烈迴響。

在短短的幾年內，Grafana 不僅僅提供了與 Prometheus 的深度整合，還與多種數據源如 Elasticsearch、InfluxDB、[Logz.io](http://logz.io/) 等實現了無縫的接軌。這使 Grafana 不只是一個數據視覺化工具，而是一個全方位的監控解決方案平台。

但 Grafana 的真正強大之處不止於此。其獨特的插件架構，允許開發者自行擴展其功能，使其成為一個真正「監控的核心所在」。不論是資料庫管理員、開發者，或是 IT 系統工程師，都能在 Grafana 中找到專屬於自己的監控視角。

隨著時間的推進，Grafana 的功能和生態系不斷豐富。從最初的數據視覺化工具，到現在的監控巨頭，Grafana 的旅程證明了一件事：有時，一個好的工具，真的能夠改變整個行業的規則和格局。

![https://ithelp.ithome.com.tw/upload/images/20231001/20149562gFEriBggmf.png](https://ithelp.ithome.com.tw/upload/images/20231001/20149562gFEriBggmf.png)

## Grafana 與 Prometheus 的差別

Grafana 和 Prometheus 都是當前監控和可視化領域中相當熱門的開源工具，但在整個監控領域中，它們各自擔當著截然不同的角色。以下概述了它們之間的主要差異：

1. **數據的收集與存儲**：Prometheus 主要著眼於數據的收集和存儲，其核心功能是捕獲和保留時間序列數據。相對地，Grafana 更專注於從多種數據源提取數據，然後將這些數據呈現為多種視覺形式。
2. **查詢和分析能力**：Prometheus 提供了一種強大的查詢語言 — PromQL，使用者可以對捕獲的數據進行即時查詢和分析。而 Grafana 則依靠各種數據源的能力來進行數據提取和展現。
3. **數據可視化**：Grafana 在數據可視化方面具有顯著優勢，它提供了豐富的圖表、儀表板和其他多種視覺表示形式。與此同時，儘管 Prometheus 也提供了一些基本的圖形化功能，但其主要焦點仍在於數據的收集和查詢。
4. **警報系統**：Grafana 備有先進的警報系統，允許使用者根據多種條件和數據閾值設置警報。而 Prometheus 雖然也提供了警報功能，但其機能在某些方面可能不如 Grafana 那麼靈活。
5. **生態系統與集成**：Prometheus 由於其廣泛的集成選項和導出器生態系統，特別適合監控 Kubernetes 集群。另一方面，Grafana 則因其能夠無縫整合和呈現來自各種數據源的數據而受到青睞。

總的來說，雖然 Grafana 和 Prometheus 在某些功能上存在重疊，但它們各自的擅長本事使它們在監控解決方案中形成了近乎無可取代的組合。

![https://ithelp.ithome.com.tw/upload/images/20231001/20149562H61XA2MGFF.png](https://ithelp.ithome.com.tw/upload/images/20231001/20149562H61XA2MGFF.png)

# Grafana Dashboard

這裡需要區分兩個概念：

- 儀表板（dashboard）: 一個或多個數據面板形成的集合。
- 面板（panel）：組成儀表板的其中一個圖表。

數據的可視化主要依賴於 Panel 來呈現。每種 Panel 都有其特定的強項，用於凸顯不同方面的數據信息。要搭建一個有效的數據圖表，我們首先需要對各種 Panel 有深入的認識。這樣，在儀表板的設計過程中，我們才能選擇最合適的選擇，並最大化地呈現其可視化效果，而在看到結果前，過程都會經過三層結構傳遞數據：Data Source & Plugin、Query、和 Transformation。

![https://ithelp.ithome.com.tw/upload/images/20231001/20149562SyzHLeSLUd.png](https://ithelp.ithome.com.tw/upload/images/20231001/20149562SyzHLeSLUd.png)

## Data Source & Plugin

Grafana 中的 Data Source 搭配 Plugin 可以理解為包含數據的各種來源，在現今的當下，Grafana 提供了 155 個多種數據源供我們使用，基本上已經涵蓋所有最主流的數據來源。無論是 SQL 數據庫、Grafana Loki、Grafana Mimir 還是基於 JSON 的 API，都可以作為數據源，甚至包括簡單的 CSV 文件。要進行儀表板的可視化，首要之務是選定一個合適的數據源。

儘管每種數據源都有其特有的結構和查詢方式，剛入門的我們可能會感到有些混淆，但 Grafana  允許我們在同一儀表板上集成和展示多個數據源的數據，讓我們能夠更整體地掌握和分析資訊。

![https://ithelp.ithome.com.tw/upload/images/20231001/201495626g1EQZR6hY.png](https://ithelp.ithome.com.tw/upload/images/20231001/201495626g1EQZR6hY.png)

## Query

查詢的功能就如同一扇門，幫助我們進入廣袤的數據海洋，並從中挑選出那些最有價值的數據。這些查詢不僅使數據可視化變得更加高效，而且能夠更精確地回答我們對系統和操作流程的疑問。拿一家運營著線上商店的公司來說，它可能渴望了解有多少客戶在將產品添加到購物車時碰到了問題。通過針對購物車功能的訪問指標進行查詢，能夠展示每秒鐘訪問此功能的用戶數量。

當與數據源互動時，非常重要的一點是要明白：每一個數據源都有它自己的專屬查詢語言。例如，Prometheus 數據源採用的是PromQL，LogQL適用於日誌資料的查詢，而特定的數據庫則可能採用SQL作為查詢語言。而在Grafana中，每一片可視化的領域，都建立在一系列的查詢基礎之上。

以下是一幅與 Prometheus 數據源相關的查詢編輯器示意圖。你會看到一個名為 container_cpu_usage_seconds_total 的查詢，這個查詢是使用PromQL所撰寫，它主要是為了取得某一特定指標的數據：

![https://ithelp.ithome.com.tw/upload/images/20231001/201495628vcucxQXed.png](https://ithelp.ithome.com.tw/upload/images/20231001/201495628vcucxQXed.png)

## ****Transformation****

在某些情境下，原生查詢出來的數據格式可能不完全符合我們的可視化需求。這時我們可以使用「Transformation」功能來調整這些數據。隨著我們使用場景越來越複雜以及多變時，他一定是我們的好幫手。

以下是一些常見的使用轉換的場景：

- 當您需要組合兩個字段，例如把「名字」和「姓氏」結合成「全名」。
- 當您有全為文字的 CSV 數據，但需要轉換其中的字段類型，如將文字轉為日期或數字。
- 當您希望進行過濾、連接、合併或其他類似 SQL 的操作，而這些操作可能不受您的數據源或查詢語言支持。

在面板編輯頁中，這個功能位於「Transform」頁籤下，我們只要選擇需要的類型，然後進行設定，甚至可以串接多種操作。

![https://ithelp.ithome.com.tw/upload/images/20231001/201495626LATwpD4pm.png](https://ithelp.ithome.com.tw/upload/images/20231001/201495626LATwpD4pm.png)

## Panel

在完成數據的 Query 和 Transformation 後，這些數據將登上 Grafana 顯示成我們熟悉的樣子，也就是所謂的面板（Panel）。面板就像一個精心設計的展示，它不僅將數據轉化為生動的可視化效果，還為我們提供一套工具，能夠隨心所欲地調整和操控展示的内容。

面板的設置允許您決定如何展現這些數據。舉例來說，我們可以透過面板右上方的選單，選擇希望的數據展示形式，無論是條狀圖、餅狀圖或是熱點圖，皆可隨您心意。每種可視化形式都有其獨特的設定選項，讓您能夠深度定制每個細節。

面板中也嵌入了您用於指定數據可視化的查詢部分。如下所示，一幅正在編輯中的表格面板圖像揭示了其內部的結構：底部展示著查詢細節，而右側則列出了面板的各項設定選項。透過這個圖像，我們可以清楚地看到數據源、插件、查詢和面板是如何完美融合，攜手合作展現出最佳的數據視覺效果。

![https://ithelp.ithome.com.tw/upload/images/20231001/20149562POfLK1HVXL.png](https://ithelp.ithome.com.tw/upload/images/20231001/20149562POfLK1HVXL.png)

## 探討 Grafana Panels

在資料可視化領域，資料的呈現形式至關重要。不同的圖表對相同的數據集會有完全不同的解讀和呈現方式，而選擇合適的圖表則是進行有效資料分析的第一步。了解各種圖表的特點與適用場景，可以幫助我們更好地在資料大盤中展現資料。

1. 依資料格式區分：
    - 柱狀圖、折線圖和餅狀圖通常需求時間序列的資料。它們用於表示在特定時間範疇內的單一或多種類型資料的變動趨勢或比例。
    - 狀態圖、表格和儀錶盤不嚴格要求時間序列的資料。狀態圖和儀錶盤常常用於展示總覽型資料，如速度、溫度和完成度等；而表格則適合展現多維度或複雜資料。
2. 依使用意向區分：
    - 資料比對：柱狀圖和折線圖都很合適，它們能清晰展示資料的變動和比較。
    - 數據佔比：餅圖、儀錶盤和狀態圖可以清楚展示資料的佔比。
    - 趨勢追踪：折線圖和面積圖可以呈現資料的變化趨勢。
    - 數據分佈：餅圖和散點圖更適合。
3. 其他特色：
    - 文字型圖表：用於展示文字信息，提供了高度的客製化選項，得益於Markdown和HTML的靈活性。
    - 表格：特別適用於展示日誌或多維度資料，如報表形式，且支援排序等功能，方便資料比較。

Grafana 已經提供我們十分豐富的圖表選擇，透過深入了解 Grafana 的面板和圖表功能，我們可以更有效地選擇和利用這些工具進行資料可視化，從而更好地解讀和呈現資料。

![https://ithelp.ithome.com.tw/upload/images/20231001/20149562HTxbGkbHzs.png](https://ithelp.ithome.com.tw/upload/images/20231001/20149562HTxbGkbHzs.png)

# Grafana Explore

在資料可視化和監控領域中，建立和健全的查詢到確切資料是一個首要任務，無論是想要了解系統性能還是系統監控日誌，有效的查詢都是不可或缺的，而 Grafana 的 Explore 功能區塊就是為此而生：

- 查詢中心：探索功能放大了查詢的核心，使其脫離了儀表板的背景，讓使用者能夠更專注於數據查詢。這使得查詢和資料探索過程更直觀、靈活。
- 迭代和優化：在開發過程中，我們往往需要進行多次的嘗試和調整以獲得所需的查詢結果。探索模式提供了一個理想的環境，允許使用者迭代查詢，直至獲得滿意的結果。
- 資料源概覽：Grafana 支援多種資料源，從時間序列數據庫到日誌資料源，探索功能使得在不同資料源間快速切換、查看和比較成為可能。
- 結果預覽：當您進行查詢時，可以立即在探索界面上看到查詢結果，無需跳轉到儀表板或其他介面。這使得檢查和驗證查詢的正確性變得非常方便。
- 圖表整合：一旦您在探索模式中構建了滿意的查詢，就可以輕鬆地將其加入到圖表中，以進一步進行資料的可視化和分享。

![https://ithelp.ithome.com.tw/upload/images/20231001/20149562pC22aV5Lj6.png](https://ithelp.ithome.com.tw/upload/images/20231001/20149562pC22aV5Lj6.png)

## ****Logs in Explore****

Logs in Explore 是一個強大的日誌記錄和日誌分析功能。它允許我們藉由為日誌場景量身定做的場景，查詢來自不同數據源的日誌，包括：

- Loki
- Elasticsearch
- Cloudwatch
- InfluxDB

借助 Explore，我們可以通過分析日誌並確定根本原因來有效地監控、排除故障和響應事件。它還可以幫助我們通過並排查看日誌與其他遙測信號（例如指標、跟踪或配置文件）關聯起來。

![https://ithelp.ithome.com.tw/upload/images/20231001/20149562Rts4yf2yPD.png](https://ithelp.ithome.com.tw/upload/images/20231001/20149562Rts4yf2yPD.png)

日誌查詢的結果顯示為單獨的日誌行和顯示所選時間段內日誌量的圖表。

## ****Tracing in Explore****

同樣的我們可以使用 Tracing in Explore 來查詢和可視化來自追蹤數據源的數據。

支持的數據源有：

- Tempo
- Jaeger
- Zipkin
- X-Ray

![https://ithelp.ithome.com.tw/upload/images/20231001/20149562j8BcV2vFy7.png](https://ithelp.ithome.com.tw/upload/images/20231001/20149562j8BcV2vFy7.png)

## Explore 的無縫跳轉及分割

相信許多人從系列一開始至此，已對我們文章中的圖片以及 Explore 的查詢介面熟悉了起來。Explore 不僅僅是一個查詢介面這麼簡單，更是 Grafana 為我們打造的全方位可觀測性視窗。

Grafana 致力於實現 Logging、Tracing 和 Metrics 監控三本柱的無縫切換跳轉，不僅僅是這樣，整合所有維度的監控和遙測資料，更是他們戰略布局的一部分。而通過其在監控領域的開源專案，Grafana 實現了與其它平台的深度整合，為使用者帶來了前所未有的體驗。

![https://ithelp.ithome.com.tw/upload/images/20231001/20149562ddTpLKGF8U.png](https://ithelp.ithome.com.tw/upload/images/20231001/20149562ddTpLKGF8U.png)

# 結論

已經跟我一起走到這裡的你們，是不是也跟我一樣開始熱血沸騰了呢，Grafana 的作為一個前端可視化介面的角度切入整個監控領域，一開始應該沒有人可以了想到，原來整個行業以及監控領域，竟然會圍繞著一個前端介面為核心建造，而 Grafana 的厲害之處就在於，他們總是能找到他們在監控領域的定位以及使用者的痛點，並且利用開源社群的力量持續壯大自己，並且建立起自己的商業事業群時，同時將開發量能再次回饋到開源專案中，成就了開源社群中少有的雙贏佳話，再次驗證了其「取之開源，回饋開源」的最佳典範。