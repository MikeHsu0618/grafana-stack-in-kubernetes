# 可觀測性宇宙的第十七天 - Grafana 10 搶先看

# 概述

前面說到 Grafana Lab 一直以來都是以開源社群為成長動力，除了拓展更多關於監控領域的新專案外，Grafana Lab 對自己的本命專案 Grafana 也是細心規劃每一個功能，以及迭代節奏，轉眼間到現在，Grafana Lab 在 2023 年六月份也就是 Grafana 專案誕生第十週年中，迎來了 Grafana 10 的大改版。或許許多大神早已接觸 Grafana 多年，相關應用技巧跟觀念早就如火純青，但身為 Github 上面的知名開源專案，想必改版跟功能的推進也是非常快速的，所以就讓我們來一睹 Grafana 10 的魅力吧。

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562RtVQ8N5A32.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562RtVQ8N5A32.png)

# Grafana 10 的重大更新

為了慶祝其10年的里程碑，Grafana Lab 推出了Grafana 10，為開發者和企業展示了卓越的新功能、視覺化能力和協作機會。這個具有紀念意義的版本強調增強使用者體驗，使各種開發人員更容易使用，並支援資料擴散。

這個最新版本的Grafana擁有幾個新的面板，以提高視覺化能力。其中包括趨勢面板，設計用於繪製升序數字X軸資料，以及資料網格面板，為編輯儀錶板資料提供類似電子表格的介面。此外，時間序列面板現在使使用者能夠新增時間區域，以實現更簡化的視覺化。

團隊協作是Grafana 10版本的另一個關鍵重點。該平台的公共儀錶板功能，旨在與外部利益相關者分享，已經進行了改造，改進了功能，如折疊行、隱藏查詢和放大面板的能力。這些公共儀錶板現在可以在儀錶板部分的一個單獨類別中使用，使它們更容易管理。

Grafana Lab 的創始人 Tom Wilkie 分享了他對該版本的看法：Grafana 10提升了可觀察性資料的開發者使用者體驗，因此任何開發者都可以直接跳入並開始連接資料源、創建儀錶板，並將這些資源分享和擴展給隊友。

子資料夾的引入使使用者能夠以更有條理的方式組織儀錶板，基於業務單位、部門和團隊。此外，最新版本包括更新的安全功能，如增強的基於角色的訪問控制和簡化的SAML認證。

其他值得注意的更新包括新的Correlations功能，該功能使團隊能夠在資料源之間建立聯繫，以獲得全面的資料景觀視圖；一個新的前端庫，名為Grafana Scenes，用於從Grafana應用插件中建構儀錶板；以及一個專門的介面，以程式碼形式管理儀錶板。

## Feature Toggle 打開被隱藏的新功能

在這個軟體世界日益更新的時代，有些東西可能還沒進入眾人眼中就變成棄用功能，但也有些功能日後就此成為主導整個行業的決定性革新，現在就讓我們開啟 Grafana 10 推出的全部隱藏功能，並對他們有個初步的認識吧。

以下參數在 grafana.ini 設定檔中即可在 feature_toggles 中選用開啟：

| nestedFolders | 巢狀文件夾 |
| --- | --- |
| enableDatagridEditing | DataGrid 自由數據操作 |
| correlations | Correlations 數據源相關性 |
| editPanelCSVDragAndDrop | 支持CSV文件的拖拽導入數據（作為數據源） |
| exploreMixedDatasource | Explore 支持混合數據源 |
| publicDashboards | 公開圖表 |
| newTraceViewHeader | 新的Trace體驗 |

## Correlations

Correlation 是 Grafana 10 中最令人期待的隱藏功能，相信大家都見識到了 Grafana 在 Logs、Metrics、Traces 特定數據源中無縫跳轉切轉而量身定做的能力，現在 Grafana 的 Correlations 想要實現讓任何的數據源與數據源之間完全沒有無障礙的切換自如，無論是一個系統中的應用程序性能指標、另一個系統中的服務器登錄，還是另一個系統中的用戶活動數據，所有數據的統一視圖都將有助於在數據集之間創建有意義的連接。

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562XfWrTZ7mHs.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562XfWrTZ7mHs.png)

## Explore with Mixed Datasource

我們現在可以在 Expolore 中同時查詢多個數據源。從數據源選擇器中選擇 Ｍixed，並為每個查詢指定一個數據源，無疑是又加大了 Grafana 的操作上限啊。

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562gBb8GXFnTv.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562gBb8GXFnTv.png)

## ****Enhanced navigation and seamless onboarding****

改善 Grafana 中的導航體驗一直是每個 Grafana 版本的重點。這種增強的、直觀的用戶體驗包括帶有搜索的新標題、麵包屑和重新組織的菜單，意味著更少的麻煩和更多的時間來處理真正重要的事情：從數據中獲取有意義的見解。

關於搜尋列的更新，直接改變了我對 Grafana 的第一印象，對於日常使用者的我來說，他完全是原地升級的體驗，沒有任何的不適用，所有原本想要切換可視化介面只能從左邊的導覽列開始重新點選，但現在有了搜尋列，我只要用簡單的 control + K 就可輕易的連結到想去的區塊，不用一層一層點開繁瑣的子母階層。

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562kWqG1q0Dp5.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562kWqG1q0Dp5.png)

## Organize dashboard with Subfolders

現在我們不再需要搜索麻煩的搜尋所需要的 Grafana 儀表板。Grafana 的子文件夾功能使資源管理變得更加容易，該功能讓我們根據業務單位、部門、團隊或最適合團隊的任何內容將儀表板組織到文件夾中。

經過我們精心設計的子資料夾，使我們能夠為團隊創建更加自助的可觀察性解決方案。但在合理範圍內 ， 還可以在文件夾級別分配用戶、團隊或角色特定的權限。因此，您可以將對每個文件夾中的敏感信息的訪問限制為僅授權人員，同時增強數據的整體安全性。

借助這些用戶友好的文件夾結構，篩選儀表板以獲得正確的資訊變得更加容易，從而節省時間（和理智），同時保護機密數據。

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562N2GtznvBM0.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562N2GtznvBM0.png)

# Loki log context

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562yx41bwvuHm.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562yx41bwvuHm.png)

為了增強我們的日誌上下文查詢， Grafana 團隊引入了一個強大的新工具 - Loki log context。我們可以透過點擊日誌的上下文查詢預覽來啟動編輯器，並可以從此編輯器中來調整 LogQL 或標籤過濾去來修改查詢，使我們能針對所需的特定日誌，輕鬆的過濾出其前後日誌中的關鍵日誌。

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562SSvJViscDE.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562SSvJViscDE.png)

## ****Share Grafana dashboards with Public Dashboards****

通過對公開儀表板的分享，我們終於可以允許與組織外部的任何人共享我們的儀表板了，只要我們簡單的點擊儀表板介面中的分享連結。

> 注意：需要注意的是公開儀表板，可能會導致而外的資料查詢費用，和限定的 Data Source。
>

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562jyEWM4Ji7R.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562jyEWM4Ji7R.png)

## **SAML 認證設置界面**

通過最新的 Grafana 10，Grafana 為我們推出了全新的 UI 設定 SAML 流程，使我們無需重新啟動 Grafana 並降低了設定錯誤的可能性，並且保留原本使用啟動設定檔的方式，給那些想要一切都設定在檔案中的人們。

此更新的 UI 可實現快速、安全的 SAML 身份驗證配置，從而增強 Grafana 設置的安全性。

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562PHiDZirVv6.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562PHiDZirVv6.png)

## ****Grafana Cloud 預設關閉使用 Angular 相關 Plugins****

2018 年，Grafana 團隊做了一個艱難的決定：要不從 Angular 1 遷移到 Angular 2，要不就考慮其他選擇。

最終，Grafana 團隊決定改用 React，因為兩者的重構功能與 Angular 遷移到 Angular 2 相當，而且 React 提供了許多優勢。其團隊使用 React 為 Grafana 的整個前端提供支持：

1. React 的生態系統非常龐大：當 Grafana 團隊開始規劃開發這個新 Canvas 面板類型時，他們也得益於 React 上的龐大資源，順利減少幾個月的開發時間。
2. React 的性能使 Grafana 易於擴展，基於內置的渲染優化，以及基於類的組件的 shouldComponentUpdate 和 hook 的依賴數組，使我們更容易優化代碼的性能並最終提供出色的用戶體驗。
3. React 被廣泛使用：由於 Grafana 是開源的，基於最流行的前端庫有助於促進社區對 Grafana 的貢獻。

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562Y11AsXlAhJ.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562Y11AsXlAhJ.png)

## Canvas

Canvas 面板在 Grafana 10 中全面推出，它將 Grafana 的強大功能與自定義元素的靈活性結合在一起。畫布可視化是可擴展的、表單構建的面板，可用於在靜態和動態佈局中顯式放置元素。這使我們能夠以標準 Grafana 面板無法實現的方式設計自定義可視化和疊加數據，所有這些都在 Grafana 的 UI 中進行。

正如 Grafana 開發團隊所說的：並不是所有資料都適合用圖表呈現，有些地方用大眾早已熟悉的 Canvas 元素表示，才能更加理解數據背後的含義。

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562VWftg9a4sM.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562VWftg9a4sM.png)

## Grafana Scenes

[Grafana Scenes](https://grafana.github.io/scenes/?pg=blog&plcmt=body-txt)，一個新的前端函式庫，使開發者能夠直接在其 Grafana 應用服務插件中創建類似儀表板的體驗，例如查詢和轉換、動態面板渲染和時間範圍，最酷的事情是我們可以向其中添加各種動態行為。因此，要有條件地顯示儀表板的一部分，或根據用戶操作動態更改它，我們可以向面板添加按鈕，這顯然無法對一般我們認知的 Grafana 儀表板中執行。我們可以將所有這些設定用程式碼添加到其中，以創建更加動態和用戶友好的儀表板或數據應用程序。

值得注意的是，Grafana Scenes 還採用了可擴展性的概念，允許開發團隊創建高度動態和可定制的儀表板體驗。

以下看似熟悉又陌生的介面，就是由 Grafana Scenes 客製出來在 Grafana UI 上的傑作：

![https://ithelp.ithome.com.tw/upload/images/20231002/20149562gonJ2Y7wDq.png](https://ithelp.ithome.com.tw/upload/images/20231002/20149562gonJ2Y7wDq.png)

# 結論

Grafana 的發展很奇妙，不像一些其他流行的前端框架或工具。當我們看到像 Vue 和 React 這樣的前端框架，它們在大版本更新時，往往會有相當多的改變，使得開發者必須重新調整或學習新的觀念。但 Grafana 的情況有點不同。它在監控領域的地位，使其在版本迭代時更加注重功能的增強而非大幅度的改變，這也意味著，對於使用者來說，學習曲線並不會因為版本的更新而劇烈改變。

Grafana 團隊的這種策略，顯然是希望能夠確保工具的長期穩定性和生態的繁榮。社群的反饋和討論無疑在這其中扮演了非常關鍵的角色。很多實驗性功能，在正式加入之前，都會經過廣大社群的檢驗，這確保了只有真正有價值、且對大眾有用的功能會被納入預設功能。

這種尊重使用者和社群的態度，讓 Grafana 在監控領域中獲得了廣大的好評和認可。而這也是為什麼，即使在 Grafana 10 這種具有里程碑意義的版本中，大家仍然習慣性地稱它為「Grafana」，而不是特定的版本名稱。這不僅僅是一個名稱，更是對其持續穩定和用戶友好的極高讚賞。

至於那些隱藏的、未被正式納入的功能，對於好奇心重的開發者來說，確實是一片未開發的「新大陸」，等待著去探索和嘗試。當然，這需要一定的冒險精神和對新技術的熱情，但這也是技術領域中最吸引我的一部分。

---

References

[New in Grafana 10: A UI to easily configure SAML authentication | Grafana Labs](https://grafana.com/blog/2023/07/27/new-in-grafana-10-a-ui-to-easily-configure-saml-authentication/)

[New in Grafana 10: Better log context for better log analysis | Grafana Labs](https://grafana.com/blog/2023/09/12/new-in-grafana-10-better-log-context-for-better-log-analysis/)

[Top 10 Grafana features you need to know about](https://grafana.com/blog/2023/07/14/celebrating-grafana-10-top-10-grafana-features-you-need-to-know-about/)

[Watch Grafana 10 demos: New visualizations, plugin tools, and more from the latest Grafana release | Grafana Labs](https://grafana.com/blog/2023/06/29/watch-grafana-10-demos-new-visualizations-plugin-tools-and-more-from-the-latest-grafana-release/)

[Grafana 10 release: New panels, Grafana as code updates, data correlations, and more  | Grafana Labs](https://grafana.com/blog/2023/06/13/grafana-10-release-all-the-new-features-to-know/)

[Breaking changes in Grafana v10.0 |  Grafana documentation](https://grafana.com/docs/grafana/latest/breaking-changes/breaking-changes-v10-0/)

[An inside look at how React powers Grafana’s frontend  | Grafana Labs](https://grafana.com/blog/2023/06/28/an-inside-look-at-how-react-powers-grafanas-frontend/)