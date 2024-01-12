![https://ithelp.ithome.com.tw/upload/images/20240112/20149562oL7IkBTcjw.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562oL7IkBTcjw.png)

# 前言

如果說 2023 是可觀測性社群大放異彩的一年，那在紐約舉辦的 ObservabilityCON 2022 年度盛會，可以說是 Grafana 團隊展現對可觀測性宇宙擴展的野心也不過份。

在 2022 年的 ObservabilityCON 中，Grafana 團隊正式開源了兩個專案，這兩個專案與傳統的可觀測性三大支柱（Metrics、Logs、Traces）不同。其中一個是代表 Continuous Profiling 的 Grafana Phlare，該專案於 2023 年三月與同領域著名專案 Pyroscope 合併，成為了 Grafana Pyroscope。另一個則是我們的焦點 — Grafana Faro。有趣的是，即使在 2024 年的今天，作為可觀測性代名詞的 OpenTelemetry 社群仍在探索前端瀏覽器可觀測性的清晰定義，我們即將利用 Grafana Faro 來探討 Grafana 和 OpenTelemetry 對前端監控和效能指標的深入見解。

# 前端可觀測性的重要性

在 2023 年的 [ObservabilityCON](https://www.youtube.com/watch?v=Kt1B-oea3Ls) 大會上，Grafana 團隊再次明確地強調了 RED 指標的關鍵性，指出「Humans are impatient」，所以一個服務「是否正常運作」和「是否使人愉悅」，與使用者的體驗密切相關，這些因素對使用者的整體消費體驗具有深遠影響，有時能直接影響業務成敗。從 Amazon、Google 等網路巨頭的經驗中，我們看到了反應時間的延遲與業務績效之間存在密切且顯著的關係。即使是毫秒級的延遲增加，也可能導致可觀的業務損失。因此，業界對可觀測性的關注持續增強，期望通過精確的監控和分析，找到並改善服務之間的潛在瓶頸，從而優化整體性能。

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562TQvaSewltp.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562TQvaSewltp.png)

Grafana 團隊同時也提出了另一個觀點「Performance Golden Rule」是一個在網站性能優化領域常被提及的概念，最初由 Steve Souders，一位在網站性能優化領域的先驅和專家提出：

> **用戶感知到的 80-90% 響應時間是在前端發生的。**
>

在這理論的 Response Time / Concurrnet Users 矩陣圖中，可以得出當同時使用人數越低或後端服務負載不高時，前端的處理時間在影響效能的比重就越大。可以看出前端效能優化的重要性完全不亞於後端，進而點出 Grafana Faro 在效能指標、量測與可觀測性的重要之處。

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562hqdilGcWX8.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562hqdilGcWX8.png)

# 利用 RUM 工具實現前端可觀測性

當前數位時代中，為用戶提供高效能且流暢的體驗對任何線上服務至關重要。這正是實際用戶監測（Real User Monitoring，簡稱 RUM）在前端開發和生產環境中成為不可或缺工具的原因。RUM 不僅能幫助我們深入了解用戶的互動體驗，還能通過收集和分析用戶行為、效能指標及其他可觀測性數據，提供寶貴的洞察。這就是為什麼工具如 Grafana Faro 在當今的技術生態系統中變得如此關鍵。

Grafana Faro，作為一款 RUM 工具，專為捕捉和分析實際用戶的交互而設計。它不僅追蹤用戶的具體行為，如點擊和滾動，還能詳細監控頁面加載時間和服務器響應時間等關鍵效能指標。透過 Grafana Faro，我們能夠精準定位影響用戶體驗的瓶頸，從而快速作出改進。

此外，Grafana Faro 提供的深入可觀測性指標，如瀏覽器錯誤日誌、API 請求狀態，以及用戶設備信息等，對於理解應用在不同環境下的表現至關重要。這些數據不僅有助於進行跨瀏覽器和設備的優化，也確保所有用戶都能獲得更好的使用體驗。

![https://ithelp.ithome.com.tw/upload/images/20240112/201495621gFu4CyEJc.png](https://ithelp.ithome.com.tw/upload/images/20240112/201495621gFu4CyEJc.png)

# Grafana Faro 是什麼？

> Grafana Faro 是一個開源的 Web SDK，可以僅僅使用兩三行代碼輕鬆嵌入到 Web 應用服務中，收集 Real User Metrics(RUM) 資料，其中包括效能指標、Console Log、Exception、Events、Traces。
>

Grafana Faro 可以理解為一個佈署在瀏覽器終端的 Javascript Agent，自動地收集各種瀏覽器資訊和追蹤訊號，透過 Faro Collector（Grafana Agent / OpenTelemetry Collector）進一步分發到作為儲存後端的 Grafana Loki 和 Grafana Tempo。並且以 Grafana 作為靈活的可視化介面，輕鬆串連起這些原本在每個用戶設備中猶如黑盒子般的前端可觀測性資訊，並且可以與後端數據交互關聯，獲得無縫、全端的系統樣貌。

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562MGZTeqs6dC.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562MGZTeqs6dC.png)

> Grafana Cloud 平台於 2023 年四月以 Grafana Faro 為底層，提供了名為 Frontend Observability 的 Public Preview 版本。
>

## 使用 Grafana Faro 的優勢

當然，市面上不是只有 Grafana Faro 一套前端監控工具，依然有許多像是 Datadog、Sentry、Elastic 等老牌監控 SaaS 服務廠商在前端領域耕耘許久。大部分都屬於商業閉源專案，並且使用上都需要透過該平台託管的監控介面。少部分開源的 Sentry 官方則是只有提供 Docker Self-host 版本，並且需要額外維護另一套複雜資料庫如 Postgres、Kafka、Clickhouse，實在令人眼花撩亂。

而 Grafana Faro 則是不只與自家的 Grafana 全家桶高度整合，並且對齊了可觀測性社群的前進方向，使我們能夠彈性的在其之上，繼續堆疊我們的可觀測性架構：

- 使用 Grafana 作為唯一可視化介面，減輕切換平台的操作負擔。
- 面向 OpenTelemetry 整合與使用。
- 使用日誌格式存儲量測數據於 Grafana Loki，保持客製與擴充彈性，如效能指標、日誌、異常、事件和追蹤的類型。
- 整合 React 生態元件，使開發人員得到更細顆粒的見解，如 `ReactRouterVersion` (v4-v6) 將會在每次切換路由時，產生特定事件傳送至儲存後端分析。
- 集中 Prometheus 的監控以及告警（藉由 Grafana Loki、Grafana Tempo）。

## 實現 **Observability 的目的**

可觀測性的核心不僅僅在於數據的收集，而是希望通過使用可觀測性工具或平台來解答關鍵的業務問題，從而提高系統的透明度和理解度。具體來說，我們希望能夠回答以下問題：

- **服務範圍**：請求經過了哪些服務，以及這些服務如何相互作用？
- **服務行為**：在處理請求的過程中，每個服務具體執行了哪些操作？
- **性能瓶頸**：如果出現性能緩慢，問題的瓶頸在哪裡？
- **錯誤定位**：當請求失敗時，錯誤的具體發生位置是哪裡？
- **關鍵路徑分析**：哪些元素構成了請求的關鍵路徑？
- **異常檢測**：當前執行狀態與正常系統行為相比有哪些異常？
- **時間分析**：造成處理時間延長的原因是什麼？

可觀測性的目的是為了深入了解系統的運行狀態，並在必要時快速定位並解決問題，從而提升系統的穩定性和效率。

## Grafana Faro 與 OpenTelemetry Trace 的結合

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562PMvJIn7Dln.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562PMvJIn7Dln.png)


OpenTelemetry 是雲端原生基金會（Cloud Native Computing Foundation ）中的一個可擴展的開源可觀測性框架和工具集，旨在建立為遙測資料建立單一開放式標準和管理遙測數據，促進程式語言、框架和環境之間的一致性，例如追蹤、指標和日誌。透過該技術收集各種資料監控有如黑盒子般的應用程式。至關重要的是 Opentelemetry 並不與特定供應商綁定，可以與各種可觀測性儲存後端一起靈活運用，包括 Jaeger、Prometheus 還有我們熟悉的 Grafana 全家桶。

OpenTelemetry 最核心的價值在於，它為可觀測性數據的收集與傳輸定義了一套標準化格式。這一標準促使來自不同環境和語言的應用服務端的多樣數據（例如 OTel Instrumentation、OTel API、OTel SDK）能夠透過 OTLP 協議被 OTel Collector 統一地接收和處理。

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562vaYysCRbpt.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562vaYysCRbpt.png)

當我們談論可觀測性時，往往首先想到的是後端服務中複雜微服務架構產生的日誌、追蹤和指標資訊。這些資訊通過可視化界面串聯起來，為我們提供不同維度的上下文線索，幫助我們洞察「known unknowns」。所以，後端系統的複雜性使得我們格外重視後端的可觀測性。然而，仔細思考就會發現，完整的追蹤應該從用戶端，如網頁或移動設備發起，而我們通常只將後端微服務的第一個接入點作為起點，卻忽略了前端可觀測性的重要性。Grafana Faro 只需要短短幾行程式碼就能夠與 OpenTelemetry 無縫整合，並能讓我們真正的實現一個請求從前端到後端的完整追蹤鏈路。

# Grafana Faro Instrumentations

> 儀器（Instrumentation）：比裝置（Device）還小的單位。在可觀測性領域中，通常是指框架和函式庫的插件，它們使用來記錄收集重要的操作，例如HTTP 請求、DB 查詢、日誌、錯誤等等。
>

Faro Web SDK 透過以下量測儀器在前端收集數據。一旦收集了必要的數據，Grafana Faro Web SDK 會透過 HTTP 請求將其傳送到 Grafana Agent，這裡的數據被分類為日誌、異常、事件、測量值和追蹤。

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562xL1Q5V0Pzm.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562xL1Q5V0Pzm.png)

接著，Grafana Agent 將數據分別傳送到 Loki 和 Tempo。對 Tempo 而言，追蹤數據是通過 OpenTelemetry 協議（OTLP）發送的，而日誌、異常、事件和測量值則保存在 Loki 中，並附加了額外的 Metadata。

然而，追蹤數據也可以發送到其他 OTLP 服務器，如 Jaeger。在這種情況下，我們要麼需要使用插件在 Grafana 儀表板中查看數據，要麼直接在 Jaeger 中查看。

當數據保存在 Loki 或 Tempo 時，可以使用 Grafana 簡單地查詢數據並在儀表板上查看，或使用 Explore 頁面。

## Faro **WebInstrumentations**

在預設情況下，Faro Web SDK 啟用所有 **WebInstrumentation**，而我們也可以透過手動指定 Instrumentation 的方式調整。

```yaml
export function getWebInstrumentations(options: GetWebInstrumentationsOptions = {}): Instrumentation[] {
  const instrumentations: Instrumentation[] = [
    new ErrorsInstrumentation(),
    new WebVitalsInstrumentation(),
    new SessionInstrumentation(),
    new ViewInstrumentation(),
  ];
	...
  return instrumentations;
}
```

### **ConsoleInstrumentation**

根據啟用的日誌級別過濾器（debug、error、log）收集日誌。

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562IVuEHA1JlF.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562IVuEHA1JlF.png)

### ErrorsInstrumentation

收集未捕獲的錯誤，提取其堆棧跟踪（如果可用）並報告給服務器。

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562elOjBFf8Hy.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562elOjBFf8Hy.png)

### WebVitalsInstrumentation

網站體驗核心指標（Core Web Vitals）是Google於2020年推出的指標，用來量測訪客在頁面的瀏覽體驗狀況，主要用於量測瀏覽器中網站的實際性能，以便改善用戶體驗：

- TTFB（第一個位元組的時間）
- FCP（首次內容繪製）
- CLS（累積佈局偏移）
- LCP（最大內容繪製）
- FID（首次輸入延遲）
- INP（與下一次繪製的交互）→ 將於2024 年 3 月[取代 FID](https://web.dev/blog/inp-cwv?hl=en)

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562wLcoRV8Grh.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562wLcoRV8Grh.png)

### SessionInstrumentation

幫助關聯在應用中的單個會話期間，特定終端用戶所遇到的錯誤、日誌和事件。

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562jtCX4SF6xU.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562jtCX4SF6xU.png)

### ViewInstrumentation

幫助關聯在應用的特定部分所發生的錯誤、日誌和事件。

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562LEt6o7Kd5K.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562LEt6o7Kd5K.png)

## Faro TracingInstrumentation

TracingInstrumentation 能夠收集使用者每次跟前端服務互動時產生的詳細流程追蹤資料，並且透過 W3CTraceContextPropagator 將 Traceparent 添加在每個 Request Header 中，向下個目的地傳播。

官方預設關閉 TracingInstrumentation，由於追蹤會產生大量資料並且可能影響應用程式效能，所以我們需依照自身需求來評估是否啟用。

TracingInstrumentation 底層則是使用以下 OpenTelemetry instrumentations：

- [@opentelemetry/instrumentation-document-load](https://www.npmjs.com/package/@opentelemetry/instrumentation-document-load)
- [@opentelemetry/instrumentation-fetch](https://www.npmjs.com/package/@opentelemetry/instrumentation-fetch)
- [@opentelemetry/instrumentation-xml-http-request](https://www.npmjs.com/package/@opentelemetry/instrumentation-xml-http-request)

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562XhWOBfdxEU.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562XhWOBfdxEU.png)

## Grafana Faro ReactIntegration

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562zmCJlM5Mpk.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562zmCJlM5Mpk.png)

Grafana 團隊在 2018 年走到了一個極為艱難的十字路口：從 Angular 1 遷移到 Angular 2 還是選擇其他語言框架，例如 React。最終，Grafana 團隊決定使用 React 徹底改寫 Angular 1 版本的 Grafana，因為其龐大的生態系以及廣泛程度。

為了可以更加深入的觀察 React 前端應用程式，Faro Web SDK 優先對於 React 生態進行整合，使我們可以簡單的透過幾行程式碼，更深入的觀查 React 中的下列功能：

- **Error Boundary**：包裹著指定組件，提供這些組件產生的 Error Stack Traces，並能夠正常呈現頁面其餘部分。
- **Component Profiler**：擷取元件重新渲染和就緒的持續時間。
- **React Router (v4-v6)**：會針對每次路由器變更向後端傳送事件。
- **SSR support**

### Error Boundary

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562LlXURbFtAq.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562LlXURbFtAq.png)

### **Component Profiler**

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562zw435aLEAx.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562zw435aLEAx.png)

### React Router

![https://ithelp.ithome.com.tw/upload/images/20240112/20149562kEHAQcnxfi.png](https://ithelp.ithome.com.tw/upload/images/20240112/20149562kEHAQcnxfi.png)

# OpenTelemetry 前端可觀測性中所面臨的挑戰

![https://ithelp.ithome.com.tw/upload/images/20240112/201495624v7aVfJhMD.png](https://ithelp.ithome.com.tw/upload/images/20240112/201495624v7aVfJhMD.png)

我們知道 OpenTelemetry 旨在建立為遙測資料建立單一開放式標準和管理遙測數據，但其在代表 OpenTelemetry 前端函式庫的 OpenTelemetry Browser 頁面中，至今仍然是處於初期摸索階段。可以看出在變化多端的瀏覽器環境中，制定一套通用統一的規範在實現上有一定的難度。

此外，從 OpenTelemetry Client Instrumentation SIG 的[會議文件](https://docs.google.com/document/d/13Z9CD-JlT_HQwx1qJnkkZnGSLe6qb67qHxQ3tu_pevM/edit)中，我們可以大略得知，現今在前端實施 RUM（Real User Monitoring）時所面臨到的挑戰大概包括：

- **隨機事件與長時間跨度**：有些數據類型不完全適合追蹤模型。例如，一些計時事件、網絡更改或隨機錯誤可能在沒有跨度進行時發生。這些事件本身沒有持續時間，不適合跨度模型。另外，某些數據可以表示為跨度，但可能持續時間非常長，例如瀏覽器頁面會話可能開啟數小時或數天。
- **用戶互動**：用戶互動是客戶端遙測的關鍵測量指標。但如何表示這些互動並不清楚，它們可以簡單地表示為無持續時間的事件，也可以表示為跨度，因為它們會觸發其他操作。
- **各種觸發事件的認定**：包括推送通知、網絡插座、跨iframe消息等。
- **Session**：長時間運行的 Session 難以表示為跨度。
- **共同數據模型**：網頁和移動端有一些重疊，比如用戶互動。但也有不同的概念，比如瀏覽器頁面視圖或移動應用啟動。在移動端，不同平台（如Android、iOS）之間存在差異。
- **因果關係**：是否應該定義因果關係/分區，以便能夠將不同的HTTP請求鏈接到後端和由特定用戶互動（比如在SPA中點擊了一個按鈕）引起的其他事件，以測量從互動時刻到某個後續事件發生的整個用戶互動持續時間。
- **缺乏瀏覽器支持**：在某些情況下，瀏覽器目前可能不提供足夠的API。例如，JS 腳本無法讀取文檔、圖像、CSS 等資源的 Responce Header。

這些挑戰顯示了在RUM實施中需要考慮的多方面和複雜性，從技術實施到數據模型的選擇，以及用戶行為和系統設計之間的交互。

# 結論

市面上的監控服務百家爭鳴，在 RUM 領域上更是每個監控廠商的兵家必爭之地，畢竟我們從上面的介紹中可以得知，OpenTelemetry 前端可觀測性距離成熟仍需要一段不小的努力。但 Grafana Faro 提供給我們一種擴充性高又不必額外維護資料庫的選擇，更好的提升 Grafana Loki、Grafana Tempo 的使用價值，並且全力支援整合開源社群的主流生態，如 OpenTelemetry Instrumentation、React、Web Vitals，給予我們更全方位的角度洞悉前端服務的世界，尤其是在各個生產環境中的終端用戶中，這些實際量測資料顯著格外富有價值。

總之，強化前端可觀察性對於提供良好的使用者體驗至關重要。讓我們利用 Grafana、Loki、Tempo 等工具並採用 OpenTelemetry 等標準，使開發人員能夠深入了解應用程式效能、有效解決問題，並最終提高前端應用的整體品質。

---

References:

[Grafana Agent v0.37: Feature parity between Static and Flow mode, easy migration configs, and more | Grafana Labs](https://grafana.com/blog/2023/10/10/grafana-agent-v0.37-feature-parity-between-static-and-flow-mode-easy-migration-configs-and-more/)

[Frontend Observability with Grafana Faro – Mayflower Blog](https://blog.mayflower.de/15107-grafana-faro-frontend-observability.html)

[[Grafana] Faro Web SDK 學習筆記](https://blog.kevinyang.net/2023/07/16/faro-web-sdk-study/)

[Introducing Grafana Faro, an open source project for frontend application observability | Grafana Labs](https://grafana.com/blog/2022/11/02/introducing-grafana-faro-oss-application-observability/)

[Grafana Agent Configurator](https://grafana.github.io/agent-configurator/)

https://github.com/GoogleChrome/web-vitals

https://github.com/cedricziel/grafana-faro-browser-extension

https://github.com/pateroski/grafana-faro

https://github.com/liamoddell/Grafana-Faro-Chrome-Extension

https://github.com/cedricziel/faro-shop

https://github.com/cbos/grafana-scenes-playground