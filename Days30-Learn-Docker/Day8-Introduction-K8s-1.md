###### tags: `鐵人賽` `Kubernetes`

# Day8 深入淺出 Kubernetes(一)

昨天簡單介紹並快速體驗了 Kubernetes 的基本功能，今天開始就要來深入一點去了解。

## Kubernetes Clusters

首先我們昨天成功建立了 Kubernetes Clusters，但是 Clusters 是甚麼呢?他的結構又是如何呢?
Kubernetes Cluster 是一組用來執行容器化應用程式(containerized application)的節點機器，透過協調連接單個 unit 在一起工作的高可用性計算機集群。聽不懂嗎？沒關西，看張圖就會瞭解了。
如圖(一)中表示 Clusters 具有至少一個 Master 作為協調，以及多個 Node 用來運行多個應用程序的程序，因而稱作節點機器。
![](https://i.imgur.com/45lgXBf.png)
圖(一) Cluster 架構

Kubernetes 會自動將容器化的應用程序部署到集群，而無需將它們專門綁定到單個機器。透過這種方式部屬可以更加有彈性，且 Kubernetes 以更有效的方式自動在整個集群中分配和調度應用程序容器。能夠輕鬆達成效能及資源利用最大化。

剛剛提到了 master 負責管理群集，Node 則是各個單獨的 VM 用來運行 APP。除此之外仔細看得話，每個節點上都具有一個 Kubelet，Kubelet 是甚麼呢？
Kubelet 是用於管理節點並與 master 通信的代理人，如果 Master 是老闆 Kubelet 就是經理、主管。而結點上還有 Docker 則是用來處理 container 的工具。
至於具體上 Kubelet 是怎麼溝通呢？Node 會透過 Master 公開的 Kubernetes API 來與 Master 通信，而使用者也就是我們，也可以透過 Kubernetes API 直接操作 Clusters，而不需要直接進到 Clusters 裡面。

## Deployments

還記得怎麼開始建立 Kubernetes Clusters 嗎？

```bash
minikube start --driver=docker
```

快速建立好 Clusters 之後就可以部屬容器化的 APP 了，但是要如何部屬呢？首先需要先創建一個部屬 Kubernetes 的配置，透過配置 Kubernetes 才知道如何創建和更新應用程序。

創建一個簡單 API 範例

```bash
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```

而在我們創建完應用程序後，K8s 會持續監視這些應用程序(with K8s Deployment Controller)。如果運行應用程序的節點出現故障或被刪除，便會將應用程序替換為群集中另一個節點上的應用程序。也就是昨天提到 K8s 的特點之一自我修復功能(self-healing)。
圖(二)為部屬完容器化 APP 後的 Clusters 結構。
![](https://i.imgur.com/U8UxG2S.png)
圖(二) 部屬容器化 APP

## Proxy 代理

正常來說，這個時候外部的我們是訪問不到 Clusters 的應用程序，除了可以使用昨天提到的 expose 來公開，也可以使用接下來會運用的技巧，我們可以使用 kubectl 命令創建一個 proxy，並透過 proxy 將通信轉發到 Clusters 的專用網絡中，當然這樣是比較麻煩啦，畢竟要一直監聽。

```bash
kubectl proxy
```

執行完的輸出

```
Starting to serve on 127.0.0.1:8001
```

接下來我們就能直接訪問 localhost 的 8001 端來訪問 APP 了。
訪問 http://localhost:8001/api 會得到

```json
{
  "kind": "APIVersions",
  "versions": ["v1"],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "172.17.0.3:8443"
    }
  ]
}
```

## 總結:

明天會來學習 Kubernetes 如何乘載應用程序並如何觀察，以及公開 APP 的理論及方法。

參考文獻:
[Kubernetes 官方文件](https://kubernetes.io/docs/home/)
