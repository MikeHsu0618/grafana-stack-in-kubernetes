# 可觀測性宇宙的第九天 - Helm 安裝包管理器介紹


# 概述

一件事物的興起，必然與其他事物有直接或間接的關係，這個比喻拿來放在 Helm 身上是再也貼切不過了。在這個容器化時代中，隨著各式各樣的微服務驅動著的 Kubernetes 巨輪，許多 Kubernetes 相關生態的應用也蓬勃發展，同時也指數般增加了架構、設定、管理上的複雜度，而 Helm 就是為此而生。

# Helm 是什麼

![https://ithelp.ithome.com.tw/upload/images/20230924/20149562FRuh3sQ3PU.png](https://ithelp.ithome.com.tw/upload/images/20230924/20149562FRuh3sQ3PU.png)

在首屆 KubeCon 上，Helm 最初以 "Helm Classic" 的名稱亮相。從一開始只是 Kubernetes 的子項目，到 2018 年 6 月升級為 CNCF 的獨立項目，Helm 的成長顯而易見。到了 2019 年 11 月，Helm 不僅結束了 Helm 2 的生命週期，還發佈了他們最新的 Helm 3。最後，於 2020 年 4 月，Helm 成為 CNCF 早期畢業的模範項目之一。

官方對於 HELM 的定義：

> Helm is a tool for managing Charts. Charts are packages of pre-configured Kubernetes resources.
>

Ｈelm 是一個 Kuberentes 應用的管理工具，確切點來說是用來封裝 Kubernetes 原生 YAML 設定檔，它會將一個服務裡面中各種元件的 YAML 設定檔轉統一打包成一個 Helm Chart，並且我們可以靈活的抽象出其中的參數，透過 Helm Template 同時管理、定義、安裝和升級 Kubernetes 應用服務，並且可以管理它們的依賴關係。

## Helm 三大概念

1. Chart： Chart 代表著整個 Helm Package，包含需要再 Kubernetes 叢集內部運作的所有服務、工具或設定所需要的資源定義，可以把它當成 Homebrew formula、Apt dpkg 在 Kubernetes 中的替換概念。
2. Repository：用來存放和共享 Charts 的地方，它就是 Helm 資源的核心倉庫。
3. Release: 是運行在 Kubernetes 叢集中 Chart 的實例，一個 Chart 可以透過不同的 Release Name 在同一個叢集安裝多次，每個 Chart 都會有自己的 Release 跟 Release Name，並且各自享有獨立的生命週期、版本。

有了上述這些概念之後，我們可以用一段簡單的敘述來解釋 Helm 在實務上扮演的角色：

使用 Helm 安裝 Charts 到 Kubernetes 叢集中，每次安裝都會創建一個新的 Release。而我們可以在 Helm 的 Repositories 中尋找到符合需求的 Chart。

## Helm Chart Registry

Helm 官方團隊於 2020 年 10 月宣布將原本的 Helm Hub 遷移到 Artifact Hub。

原先 Helm Hub 是在 Moneyes 項目基礎上建構而成，該項目是為了處理有限數量的 Helm Repository 而生，隨著 Helm 的使用量級顯著提升，它開始出現了一些局限性，而這時十分相容 CNCF 項目的儲存庫解決方案 Artifact Hub 就出現了。Artifact Hub 擁有了 Helm Hub 所有相同圖表並提供更快速的搜尋，並且支持和促進 CNCF 生態系統的更多內容，而不僅僅是圖表。

> 我們可以在 Artifact Hub 統一找到所有大量公開的 Helm Chart：https://artifacthub.io/
>

# Helm 基本使用

## 安裝

使用 Homebrew (macOS)

```jsx
brew install helm
```

## 更新 / 回滾版本

當我們隨著 Chart 的升級，或是修改了 Release 設定，都可以使用 `helm upgrade [RELEASE] [CHART] [flags]` 指令，Helm 會嘗試執行最小侵入式升級。

```jsx
helm upgrade my-release bitnami/wordpress -f values.yaml
```

事實上，我們每做一次更動，對 Helm 來說都是一個版本的 Release，我們可以使用 `helm history [RELEASE]` 指令來查看一個特定版本。

```jsx
helm upgrade my-release  
```

假設在一次發布過程中，發生了不符合預期的事情，也很容易通過`helm rollback [RELEASE] [REVISION]`命令回滾到之前的發布版本。

```jsx
helm rollback my-release 1
```

## 移除 Release

使用`helm uninstall`命令從集群中移除一個 Release。

```jsx
helm uninstall my-release
```

## 使用 Repository

使用`helm repo list`來查看設定的 Repository。

```jsx
helm repo list
```

使用`helm repo add`來添加新的 Repository。

```jsx
helm repo add dev https://example.com/dev-charts
```

像是我們熟悉的 `apt update` 一樣，我們也可以養成每次使用 helm repo 前都執行 `helm repo update` 來確保我們 Helm 客戶端是最新資料。

# Helm Diff Plugin

`helm diff` 是一個用過都大推的插件，可以讓你預覽 `helm upgrade` 將要更改的內容，基本上就像 `terraform plan` 一樣，將最新部署版本與即將更新的設定進行差異比較，畢竟不是每個人對每次都十足把握當前環境如同自己預期般的運作著。

```jsx
helm plugin install https://github.com/databus23/helm-diff
```

安裝完後，就可以使用 `helm diff upgrade` 在指令列看到前後兩次版本差異之處。

```jsx
helm diff upgrade my-release
```

![https://ithelp.ithome.com.tw/upload/images/20230924/201495622gliqQIIUM.png](https://ithelp.ithome.com.tw/upload/images/20230924/201495622gliqQIIUM.png)

# 結論

Helm 為我們大大降低運作起一個複雜 Kubernetes 服務的學習成本，同時也大大降低了進入 Kubernetes 進階應用的門檻，有了它我們現在才有機會在這裡輕鬆的探討 Grafana 宇宙，畢竟不是每個人都有辦法一次組織、管理幾十個設定檔，並且很輕鬆的迭代回滾版本。接下來的內容中，也將會大量使用 Helm 的基本指令來管理我們所有的練習的項目，在過程中我們可以不斷的體驗到他帶來的方便以及上手的快速。

---

References

[Helm | The History of the Project](https://helm.sh/docs/community/history/)

[Helm | Helm Hub Moving To Artifact Hub](https://helm.sh/blog/helm-hub-moving-to-artifact-hub/)

[Helm | 使用Helm](https://helm.sh/zh/docs/intro/using_helm/)