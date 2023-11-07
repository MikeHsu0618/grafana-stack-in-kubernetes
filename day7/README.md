# 可觀測性宇宙的第七天 - Kubernetes 安裝（docker desktop for mac）

# 概述

經常爬實作文章的我，常遇到的問題就是，明明所有操作與設定檔都跟教學上的一模一樣，但怎麼跑出來的，只有一堆錯誤，再經過反覆查詢後，才發現原來運作環境或服務版本不一樣導致而成。而作為一個希望以完整度來減少大家學習負擔的系列文，這裡將會盡力降低任何影響變數，提供相同的運作環境以及環境安裝方式一定是必須的吧！

當今的技術市場上提供了多種工具，使開發者能夠在本機環境上輕鬆搭建 Kubernetes 集群。在這一系列中，我們將使用 Docker Desktop 內建的 Kubernetes 進行安裝教學並作為日後文章中的本地操作基礎。

# Docker Desktop

Docker Desktop 是一款由 Docker 公司為 Mac 和 Windows 桌面用戶推出的產品，讓開發者可以在本地電腦上方便地運行 Docker 和 Kubernetes。其安裝與設置過程極為直觀，一旦安裝完成，開發者便能從系統托盤或應用程序菜單直接管理 Docker。更值得一提的是，Docker Desktop 還內建了 Kubernetes，意味著開發者無需任何其他設定即可在本機運行 Kubernetes 集群。無論是初學者還是經驗豐富的開發者，Docker Desktop 都為我們提供了一個一致、彈性的容器開發環境。

> 本系列文內容運作版本為 Docker Desktop 4.22.1 (118664) & Kubernetes v1.27.2
>

![https://ithelp.ithome.com.tw/upload/images/20230922/20149562YgtqrhYxQJ.png](https://ithelp.ithome.com.tw/upload/images/20230922/20149562YgtqrhYxQJ.png)

![https://ithelp.ithome.com.tw/upload/images/20230922/20149562AxGLODTr3u.png](https://ithelp.ithome.com.tw/upload/images/20230922/20149562AxGLODTr3u.png)

## 安裝 Docker Desktop（MacOS）

1. 官方載點：https://dockerdocs.cn/docker-for-mac/install/#google_vignette
2. 將 docker.dmg 拖曳至 Application。

![https://ithelp.ithome.com.tw/upload/images/20230922/20149562IB6wrR8dgc.png](https://ithelp.ithome.com.tw/upload/images/20230922/20149562IB6wrR8dgc.png)

3. 點開 docker.app ，看到初始介面就是安裝成功了。

![https://ithelp.ithome.com.tw/upload/images/20230922/20149562n1zfWhXldk.png](https://ithelp.ithome.com.tw/upload/images/20230922/20149562n1zfWhXldk.png)

> 可以看到 Docker Desktop 的主色系由經典的淺藍，變成現在稍微深點藍，很多這幾年才接觸 Docker 老朋友，應該跟我一樣還要一點時間習慣。
>

## ****開啟 docker desktop 中內建的 Kubernetes****

1. 點擊右上角 setting 選項我們可以找到 Kubernetes 相關設定，這點選 Enable Kubernetes 並按下 Apply & Restart 會花一小段時間在安裝 Kubernetes 所需的相關 image。

![https://ithelp.ithome.com.tw/upload/images/20230922/20149562l5pdruwS2z.png](https://ithelp.ithome.com.tw/upload/images/20230922/20149562l5pdruwS2z.png)

2. 經過一段時間的等待沒有意外的話可以在我們的 Docker Desktop GUI 上看到左下角的 Kubernetes 服務亮起了綠燈，以及 Kubernetes 相關的容器也都順利的被啟動了。

![https://ithelp.ithome.com.tw/upload/images/20230922/20149562tslxxKZwKq.png](https://ithelp.ithome.com.tw/upload/images/20230922/20149562tslxxKZwKq.png)

## **查看 Local Kubernetes 狀態**

1. 取得集群資訊

```jsx
kubectl cluster-info
--------
Kubernetes control plane is running at https://127.0.0.1:51737/9ee6b7233425d4f0e72a0c444cd94d67
CoreDNS is running at https://127.0.0.1:51737/9ee6b7233425d4f0e72a0c444cd94d67/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

2. 查看 Nodes

```jsx
kubectl get nodes
--------
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   30m   v1.27.2
```

3. 查看 Kubernetes 與 Kubectl 版本

```jsx
kubectl version --short
--------
Flag --short has been deprecated, and will be removed in the future. The --short output will become the default.
Client Version: v1.25.10
Kustomize Version: v4.5.7
Server Version: v1.27.2
WARNING: version difference between client (1.25) and server (1.27) exceeds the supported minor version skew of +/-1
```

如此一來，我們已經建立好我們的 Kubernetes 運行環境。

# 結論

安裝完 Kubernetes 後，是不是跟我一樣有種摩拳擦掌、躍躍欲試的感覺啊！想想當初第一次接觸到 Kubernetes 時，當下版本才到 v1.20 ，短短兩年就到了 v1.27，而現在最新釋出版本則是 v1.28。說到 Kubernetes 的迭代速度，絕對是軟體界的翹楚，根據官方發佈策略，Kubernetes 每年計畫有三次主要版本更新，大約每三四個月一次，所以你總是會感覺到怎麼永遠都有新功能要學啊，並且除了更新速度快之外，官方僅保證隊最新的三個小版本提供修補程式更新，以達成防止使用者落後主要版本，導致產生不可預期的版本傾斜問題。而雲端 Kubernetes 使用平台如 GKE、AKS、EKS，也遵守的官方版本更新政策，所以在實務工作環境上，學會如何迭代 Kubernetes 不外乎也是一個重要課題。