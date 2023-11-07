# 可觀測性宇宙的第一天 - Grafana LGTM 全家桶的起點
![https://ithelp.ithome.com.tw/upload/images/20230916/20149562RdEO0SrN11.png](https://ithelp.ithome.com.tw/upload/images/20230916/20149562RdEO0SrN11.png)
# 前言

這是一個從異世界歸來後，一個不小心踏入 Kubernetes 可觀測性宇宙的故事，一切都要從一個非本科小白在愚昧山丘的頂端遇見 Grafana 開始說起。

在過去的兩年間，出自於對網路世界的好奇，使我慢慢開始接觸了容器化、接觸了 Kubernetes，並且有能力使用 Golang 與 Node 等程式語言，編織出腦中所構想的微服務架構，這時候架設起單純的 Kubernetes 服務編排對我來說已經不成問題，而那時候我對於服務品質的了解，僅停留在 CPU 跟記憶體需要控制得當，有問題看看日誌除錯一下就好，並沒有過多想法，於是當下所見即所得的成就感，再度將我推向愚昧山丘的高峰。直到接觸了 Grafana 及 Prometheus 這等在我認知世界中不同維度的事物存在，瞬間再度將我從愚昧山丘上踢進無窮的深淵。從理解 Grafana 在可觀測性領域的強大中，我反思到自己對微服務、容器化世界的認知不足，並且開始思考其存在的真正意義。

Grafana 在現今的軟體世界中，幾乎已成為了監控領域的代名詞，其開源且強大的功能深受社群喜愛，作為一個涵蓋數據可視化、警報系統及多數據源整合的平台，Grafana 幫助使用者深入洞察系統問題，優化效能。正因其在監控領域的關鍵地位，本系列文將深入介紹 Grafana LGTM 全家桶，帶領各位進入 Grafana 為我們精心佈局的 Kubernetes 可觀測性宇宙，雖然我們不是數學家，但我知道這聽起來很酷！

![https://ithelp.ithome.com.tw/upload/images/20230921/20149562XCTodt0zPQ.png](https://ithelp.ithome.com.tw/upload/images/20230921/20149562XCTodt0zPQ.png)

# 目錄

接下來的日子裡，我將依自己的學習經驗為大夥由淺到稍微不淺的了解 Kubernetes 中的 `Grafana 全家桶` ，有興趣的同學可以先參考參考。

## 概念篇：萬丈高樓平地起，開始爬吧

- [可觀測性宇宙的第二天 - Kubernetes 介紹](https://ithelp.ithome.com.tw/articles/10320414)
- [可觀測性宇宙的第三天 - 可觀測性和監控是什麼？](https://ithelp.ithome.com.tw/articles/10321728)
- [可觀測性宇宙的第四天 - 可觀測性三本柱是什麼？](https://ithelp.ithome.com.tw/articles/10322323)
- [可觀測性宇宙的第五天 - 第四種可觀測性訊號？ Profile](https://ithelp.ithome.com.tw/users/20149562/ironman/6674)
- [可觀測性宇宙的第六天 - Grafana 上的可觀測性宇宙](https://ithelp.ithome.com.tw/articles/10324046)

## 行前準備篇：一個蘿蔔一個坑，看倌請入坑

- [可觀測性宇宙的第七天 - Kubernetes 安裝（docker desktop for mac）](https://ithelp.ithome.com.tw/articles/10324886)
- [可觀測性宇宙的第八天 - Kubernetes Lens 推坑 K8S GUI 神器介紹](https://ithelp.ithome.com.tw/articles/10325689)
- [可觀測性宇宙的第九天 - Helm 安裝包管理器介紹](https://ithelp.ithome.com.tw/articles/10326496)

## Prometheus 篇：現代監控世界的催生者

- [可觀測性宇宙的第十天 - Prometheus 介紹](https://ithelp.ithome.com.tw/articles/10327258)
- [可觀測性宇宙的第十一天 - Prometheus 基本觀念 - PromQL](https://ithelp.ithome.com.tw/articles/10328588)
- [可觀測性宇宙的第十二天 - Prometheus 全家桶介紹](https://ithelp.ithome.com.tw/articles/10329207)
- [可觀測性宇宙的第十三天 - Kube-Prometheus-Stack 實戰（一）](https://ithelp.ithome.com.tw/articles/10329845)
- [可觀測性宇宙的第十四天 - Kube-Prometheus-Stack 實戰（二）](https://ithelp.ithome.com.tw/articles/10330704)
- [可觀測性宇宙的第十五天 - Kube-Prometheus-Stack 實戰（三）](https://ithelp.ithome.com.tw/articles/10331330)

## Grafana 篇：探索可觀測性宇宙的起源

- [可觀測性宇宙的第十六天 - Grafana 入門介紹](https://ithelp.ithome.com.tw/articles/10332031)
- [可觀測性宇宙的第十七天 - Grafana 10 搶先看](https://ithelp.ithome.com.tw/articles/10332710)
- [可觀測性宇宙的第十八天 - Grafana Cloud - Kubernetes Monitoring 實戰介紹](https://ithelp.ithome.com.tw/articles/10333406)

## Grafana Agent 篇：如果有一天，世界沒有任何隔閡

- [可觀測性宇宙的第十九天 - Grafana Agent 介紹 - 交織可觀測性的鵲橋](https://ithelp.ithome.com.tw/articles/10334036)
- [可觀測性宇宙的第二十天 - Grafana Agent - Static mode Kubernetes Operator 實戰](https://ithelp.ithome.com.tw/articles/10334635)
- [可觀測性宇宙的第二十一天 - Grafana Agent - Flow mode 實戰](https://ithelp.ithome.com.tw/articles/10335237)

## Grafana Loki 篇：有些東西應該如 grep 般簡單且成本低廉

- [可觀測性宇宙的第二十二天 - Grafana Loki 介紹 - 性價比的王者](https://ithelp.ithome.com.tw/articles/10335935)
- [可觀測性宇宙的第二十三天 - Grafana Loki 實戰](https://ithelp.ithome.com.tw/articles/10336360)
- [可觀測性宇宙的第二十四天 - Grafana Loki 效能調校](https://ithelp.ithome.com.tw/articles/10336979)

## Grafana Tempo 篇：不斷追尋的過程中也不斷錯過

- [可觀測性宇宙的第二十五天 - Grafana Tempo 介紹 - 容納百川的盡頭](https://ithelp.ithome.com.tw/articles/10337516)
- [可觀測性宇宙的第二十六天 - Grafana Tempo 實戰](https://ithelp.ithome.com.tw/articles/10338074)
- [可觀測性宇宙的第二十七天 - Grafana Tempo 效能調校](https://ithelp.ithome.com.tw/articles/10338515)

## Grafana Mimir 篇：每件事物的存在都有其意義

- [可觀測性宇宙的第二十八天 - Grafana Mimir 介紹 - 監控指標的應許之地](https://ithelp.ithome.com.tw/articles/10338987)
- [可觀測性宇宙的第二十九天 - Grafana Mimir 實戰](https://ithelp.ithome.com.tw/articles/10339435)

## 結語：只有虛無，在寂靜的宇宙中

- [可觀測性宇宙的第三十天 - Grafana 全家桶最終章，接下來呢](https://ithelp.ithome.com.tw/articles/10339834)

## Bonus：如果當初還能為你多做些什麼

- [可觀測性宇宙的第三十一天 - Grafana Tempo 搭配 Odigos 實現 NoCode Observability](https://ithelp.ithome.com.tw/articles/10340539)

- Grafana Loki vs Elasticsearch
- Granafa Mimir - 更多的 Mimir
- Grafana Pyrscope - 可觀察性的第四支柱
- Grafana Faro - 前端也值得擁有可觀測性
- Grafana AlertManager - 告警的來源不只是來自於指標
- Prometheus AlertManager - Prometheus 與告警的完美結合
- Provision Grafana - 探討 Grafana 的版本控制
- Prometheus Operator CRDs 教戰守則
- OpenTelemetry 生態講
- OpenTelemetry 和 eBPF - 監控世界下一波浪潮？
- …

# 結論

在開賽的第一天，預祝每個奮鬥的鐵人們參賽順利，緊握 Kubernetes 象徵性的船舵，在無盡雲原生的宇宙汪洋中，駛向自己嚮往的學習目標。

這次的鐵人賽使我再度有機會站在巨人的肩膀上，描述我對於 Grafana 全家桶的微小見解，常常寫著寫著發現時間不早了、發現又有新的值得寫的主題、發現完全不可能在短短的三十篇文章中道盡整個宏觀的可觀測性世界，但我依然期望，可以幫助各位在過程中對其的輪廓逐漸清晰。在最後的段落我放了些，覺得很值得分享但無法在有限的三十天內向各位介紹的主題，或許能幫助你找到有興趣的研究方向，也或許我能在三十天後繼續補全這個系列。無論如何，如果你對任何主題有興趣或疑問，歡迎留下你的想法，或是敲破碗地成為我生出下一篇文章的動力。