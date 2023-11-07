# 可觀測性宇宙的第八天 - Kubernetes Lens 推坑 K8S GUI 神器介紹


# 概述

俗話說得好，「工欲善其事，必先利其器」。對於現今的 Kubernetes 使用者的我們來說，這句話再恰當不過。在探索 Kubernetes 的深淵中，雖然有人是虔誠的指令派或排斥 GUI 派，但有時我們還是希望有一個直觀、易於操作的界面工具來幫助我們更加熟悉和管理集群，這就是 Kubernetes Lens 的出場時刻。

在去年我曾經介紹過 Kubernetes Dashboard 這個官方維護的 Kubernetes UI，當時這東西的出現，正中我這個還沒辦法脫離介面的 UI 派胃口，曾經我以為或許 Kubernetes Dashboard 就是這個領域的終點了吧，直到同事跟我介紹了這款 Kubernetes Lens。

# Kubernets Lens 是什麼？

Kubernetes Lens 是 Kubernetes 的開源 IDE。它讓雲原生開發人員實時管理和監控集群，從而簡化了 K8s 管理。

2020 年，Mirantis 從 Kontena 購買了 Lens ，後來將其開源並免費提供。包括Google 在內的許多 Kubernetes 和雲原生生態系統先驅都支持它。

它配備了一目瞭然、友好的圖形互動介面，可以直接幫助我們從介面控制和管理叢集，並且還提供從 Pod 到 Cluster 層級的儀表板，讓我們輕鬆的了解叢集中發生的各種細節，例如 ConfigMap、Network、Storage、Workload。

在日益複雜的生產環境中，單純只靠 Kubectl 指令管理一個叢集、甚至是多個叢集，已經逐漸成為一個大問題，而 Kubernetes Lens 的出現，大大的減少我們 Kubernetes 使用者的負擔，甚至是熱門的 Kubernetes 服務管理器 HELM 都能輕鬆的在 Kubernetes Lens，在接下來的文章中，我們將跟這些好用的工具緊密合作，這也是為什麼我強烈推薦給你們的原因。

# Kubernetes Lens Desktop 安裝


1. 進入 [Kubernetes Lens](https://k8slens.dev/) 官網選擇指定版本下載 Lens desktop

![https://ithelp.ithome.com.tw/upload/images/20230923/20149562waGht97REo.png](https://ithelp.ithome.com.tw/upload/images/20230923/20149562waGht97REo.png)

> 注意：由於 Kubernetes Lens 是一款以 electron 寫成的 node 服務，所以在安裝上可能會遇到 node 版本的問題，目前在 Kubernete s Lens v6 以上版本中，建議使用至少 node 16 以上的版本。
>

2. 下載完後拖曳過去安裝就以打開服務了。

![https://ithelp.ithome.com.tw/upload/images/20230923/20149562yChQRSt6w8.png](https://ithelp.ithome.com.tw/upload/images/20230923/20149562yChQRSt6w8.png)

3. 第一次安裝完成，需要註冊帳號並且選擇訂閱方案，Kubernetes Lens 提供的 FREE 方案就非常能滿足，大部分人的日常用途。

![https://ithelp.ithome.com.tw/upload/images/20230923/20149562DEm0VnNtct.png](https://ithelp.ithome.com.tw/upload/images/20230923/20149562DEm0VnNtct.png)

4. 接下來終於可以看到 Kubernetes Lens 精美的首頁畫面

![https://ithelp.ithome.com.tw/upload/images/20230923/20149562VZuDZqFtvS.png](https://ithelp.ithome.com.tw/upload/images/20230923/20149562VZuDZqFtvS.png)

5. 點擊左上的 Catalog 沒有意外就可以看到我們剛剛安裝的 Kubernetes in docker-desktop 叢集，這裡將會列出我們本地 ~/.kube/config 內所記錄的所有叢集資訊，並且釘選在左側的側邊欄上，方便我們快速切換不同集群。

![https://ithelp.ithome.com.tw/upload/images/20230923/20149562HOajNqQgHL.png](https://ithelp.ithome.com.tw/upload/images/20230923/20149562HOajNqQgHL.png)

6. 最後我們從 Kubernetes Lens 的介面中，可以輕易地感受到它的強大之處。

![https://ithelp.ithome.com.tw/upload/images/20230923/20149562bcZJL3PjFg.png](https://ithelp.ithome.com.tw/upload/images/20230923/20149562bcZJL3PjFg.png)
# Kubernetes Lens Features

Kuberentes Lens 之所以方便強大的原因在於其功能豐富並且能幫我們處理許多細節：

- 降低設置叢集訪問的複雜性並使我們能夠自動添加叢集。
- 自動發現本地 kubeconfig 文件並允許我們跨幾乎任何基礎設施管理叢集。
- 在命名空間中安裝輕量的 Prometheus 以提供指標來呈現儀表板。
- 與各種 Kubernetes 工具的高度整合，Helm 就是一個很好的例子，它可以幫助您使Helm Chart 和 Release 易於在 Kubernetes 中部署和管理。
- Kubernetes Lens Extensions 使我們能夠添加新的自定義功能和可視化效果

還有許多在付費版上才有的進階功能就讓各位細細挖掘，接下來就來介紹一下我自己認為實用的功能。

## Prometheus Metrics

在 Kubernetes Lens 儀表板的各項指標來源，其實都來自於 Prometheus。但如果我們一開始學習 Kubernetes 對 Prometheus 不太熟悉也沒關係，Kubernetes  Lens 也替我們提供一鍵安裝的服務，只要選定指定叢集並且點擊右鍵 Setting，就可以看到 Builtin Metrics Provider 提供給我們的安裝選項，基本上全部裝起來就可以在儀表板中，呈現所有方便的圖表了喔。

![https://ithelp.ithome.com.tw/upload/images/20230923/20149562aT8rITKl1b.png](https://ithelp.ithome.com.tw/upload/images/20230923/20149562aT8rITKl1b.png)

如果原本就已經擁有了自建的 Prometheus 也可以讓 Kubernetes Lens 指定特定 Prometheus Endpoint。

![https://ithelp.ithome.com.tw/upload/images/20230923/201495625w5PfXjPRl.png](https://ithelp.ithome.com.tw/upload/images/20230923/201495625w5PfXjPRl.png)

如果想知道詳細使用到的指標資訊也可以到這裡查看：https://github.com/lensapp/lens/tree/master/packages/technical-features/prometheus/src。

### HELM 管理的高度整合

Kubernetes Lens 中的 Helm 功能區塊，讓習慣在使用 Helm 的朋友一定會覺得很親切，可以瀏覽已下載好的 Chart 之外，同時可以管理每個已經部署上的 Release 服務，並且可以輕鬆更新 values 以及 rollback 到歷史版本。

![https://ithelp.ithome.com.tw/upload/images/20230923/20149562dwu1pchHiu.png](https://ithelp.ithome.com.tw/upload/images/20230923/20149562dwu1pchHiu.png)

![https://ithelp.ithome.com.tw/upload/images/20230923/20149562pk3uBuJEQ3.png](https://ithelp.ithome.com.tw/upload/images/20230923/20149562pk3uBuJEQ3.png)

### 查看 Kubernetes Container Logs

在我們還沒有建立專門查看 Logs 的日誌工具之前，我們可以透過 Kubectl 指令或 Kubernetes API 來取得相關 Container Logs，但在 Kubernetes Lens 中，一切變得更加容易，只要點選指定 Pod 後，選擇右上方左邊數來第三個 icon，即可在下方出現該 Container Logs 的介面。

![https://ithelp.ithome.com.tw/upload/images/20230923/20149562g5i7tl865G.png](https://ithelp.ithome.com.tw/upload/images/20230923/20149562g5i7tl865G.png)

### 查看 Kubernetes Lens Logs

我們同樣可以查看 Kubernetes Lens 本身的 Logs， 將根據操作系統將應用程序日誌存儲到以下位置：

- MacOS：~/Library/Logs/Lens/
- Windows：%USERPROFILE%\AppData\Roaming\Lens\logs\
- Linux：~/.config/ Lens/logs/

```jsx
ls ~/Library/Logs/Lens/
------
lens-renderer-cluster-04a55bbd86fd0f41f1a3ccf4a7cb19e1-frame.log  lens-renderer-root-frame.log
lens-renderer-cluster-158319267a57c664f7c3ebc6171eec16-frame.log  lens.log
lens-renderer-cluster-158319267a57c664f7c3ebc6171eec16-frame1.log lens1.log
lens-renderer-cluster-5cd4775358606a07a5430518548fda96-frame.log  lens10.log
lens-renderer-cluster-6de39e3445e84bb02d537e226ac012b1-frame.log  lens11.log
lens-renderer-cluster-6de39e3445e84bb02d537e226ac012b1-frame1.log lens12.log
lens-renderer-cluster-8fc41f9e289de66d0778349f2dc357c8-frame.log  lens13.log
lens-renderer-cluster-8fc41f9e289de66d0778349f2dc357c8-frame1.log lens14.log
lens-renderer-cluster-9b1d0e53ca5b953af1f91ff96f545ea3-frame.log  lens15.log
lens-renderer-cluster-9b1d0e53ca5b953af1f91ff96f545ea3-frame1.log lens2.log
lens-renderer-cluster-9ee6b7233425d4f0e72a0c444cd94d67-frame.log  lens4.log
lens-renderer-cluster-9ee6b7233425d4f0e72a0c444cd94d67-frame1.log lens5.log
lens-renderer-cluster-c216db051a1963f59b12f793286ba22e-frame.log  lens6.log
lens-renderer-cluster-cfa46a9b77423a3016912910420cfcb2-frame.log  lens7.log
lens-renderer-cluster-cfa46a9b77423a3016912910420cfcb2-frame1.log lens8.log
lens-renderer-cluster-f54056e493335cf608139c7c8af649ab-frame.log  lens9.log
lens-renderer-cluster-f54056e493335cf608139c7c8af649ab-frame1.log
```

也可以使用指令在 Debug 模式下啟動 Kubernetes Lens 來查看更多詳細日誌。要在 Debug 模式下啟動應用程序，請在啟動應用程序之前提供`DEBUG=true`環境變量，例如：`DEBUG=TRUE /Applications/Lens.app/Contents/MacOS/Lens` 。

# 結論

對我個人而言，Kubernetes Lens 是好用程度超越 Kubernetes Dashboard 的存在，也是初學者以及指令苦手的救星，很多人在初期對 Kuberentes 還不太熟悉，更難以使用指令去幫助他們了解這個無邊的全新世界。有了這些 GUI 工具或 IDE 的存在，讓我可以從另一個角度去了解到 Kuberentes 的運作方式，同時也反向學習到更多相關指令跟功能，讓更多本想打退堂鼓的小白們有了一絲希望跟另一個學習的方法呢。

---

References

[Kubernetes Lens: How To Enhance Your Kubernetes Cluster](https://cast.ai/blog/kubernetes-lens-how-to-enhance-your-kubernetes-cluster/)

[Lens Documentation](https://docs.k8slens.dev/)

[Monitoring Your Kubernetes Cluster at a Glance with Lens](https://www.mandsconsulting.com/monitoring-your-kubernetes-cluster-at-a-glance-with-lens/)