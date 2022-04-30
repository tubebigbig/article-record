###### tags: `鐵人賽` `Kubernetes`

# Day9 深入淺出 Kubernetes(二)

昨天在將我們的應用程序部屬的時候，其實 K8s 默默幫我們做了許多事情，就讓我們來介紹部屬完之後發生了甚麼事。

## Pods Overview

當我們要部屬應用程序時就會創建用來承載的 Pod，Pod 會綁定在 K8s 安排的節點上保持到其被終止或刪除。每個 Pod 可以當作是一個具有 Volumes 並共享 namespace 的 Docker，能夠擁有**Volumes**、**運行的配置**以及**唯一的 cluster IP 位址**，Pod 更可以透過共享同個 IP 位址及端口達成一個 Pod 包含多個容器的情況。如圖(一)為部屬完應用程序後的 Pod 結構。
![](https://i.imgur.com/IK5V651.png)
圖(一) Pods 結構

## Nodes Overview

Node 昨天提到過是一個個單獨的 VM 用來運行應用程序，而一個 Node 中可以擁有多個 Pod。在運行上會由 Master 來處理連接在其身上的所有 Node 調度 Pod 的方法。如圖(二)為部屬完應用程序後的包含各個 Pod 的 Node 結構。
![](https://i.imgur.com/MY69UjI.png)
圖(二) Node 結構

## 如何觀察以及排查錯誤

在將我們的應用程序部屬的時候，是不是都不知道應用程序的狀態是如何呢？K8s 有提供了幾種觀察的方式可以用來故障排除。我們將會使用以下幾種指命來觀察及排查錯誤

- kubectl get
  > 顯示資源，包含名稱、當前狀態、重啟狀態、壽命等
- kubectl describe
  > 顯示資源的詳細資訊
- kubectl logs
  > 映出 Pod 的 log
- kubectl exec
  > 在 Pod 中的 container 執行命令

**kubectl get**

```bash
kubectl get pods
```

執行完的輸出

```
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-57978f5f5d-bhq2s   1/1     Running   0          59m
```

**kubectl describe**

```bash
kubectl describe pods
```

執行完的輸出

```
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-57978f5f5d-bhq2s   1/1     Running   0          59m
PS C:\Users\liang> kubectl describe pods
Name:         kubernetes-bootcamp-57978f5f5d-bhq2s
Namespace:    default
Priority:     0
Node:         minikube/172.17.0.3
Start Time:   Tue, 22 Sep 2020 21:56:17 +0800
Labels:       app=kubernetes-bootcamp
              pod-template-hash=57978f5f5d
Annotations:  <none>
Status:       Running
IP:           172.18.0.2
IPs:
  IP:           172.18.0.2
Controlled By:  ReplicaSet/kubernetes-bootcamp-57978f5f5d
Containers:
  kubernetes-bootcamp:
    Container ID:
...(太長省略)
```

**kubectl logs**

```bash
kubectl logs kubernetes-bootcamp-57978f5f5d-bhq2s
```

可以透過訪問 API 映出 LOG

```shell
PS ~User> kubectl exec -ti kubernetes-bootcamp-57978f5f5d-bhq2s bash
# 進入container執行命令
root@kubernetes-bootcamp-57978f5f5d-bhq2s:/# curl localhost:8080
# 訪問localhost:8080
```

執行完的輸出

```
Kubernetes Bootcamp App Started At: 2020-09-22T13:57:45.802Z | Running On:  kubernetes-bootcamp-57978f5f5d-bhq2s

Running On: kubernetes-bootcamp-57978f5f5d-bhq2s | Total Requests: 1 | App Uptime: 4596.365 seconds | Log Time: 2020-09-22T15:14:22.167Z
```

**kubectl exec**

```bash
kubectl exec kubernetes-bootcamp-57978f5f5d-bhq2s env
```

執行完的輸出

```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubernetes-bootcamp-57978f5f5d-bhq2s
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
NPM_CONFIG_LOGLEVEL=info
NODE_VERSION=6.3.1
HOME=/root
```

## 總結：

原本是打算在今天學習並撰寫部屬後的架構及概念、觀察及排查錯誤的方法以及 K8s Services 的概念還有 expose APP 的方法，但是一方面是只來得及把前兩個打成文章，另一方面是擔心篇幅過長，所以就拆成兩天來撰寫了。因此，明天的目標就是學習 K8s Services 的概念還有 expose APP 的方法。

參考文獻:
[Kubernetes 官方文件](https://kubernetes.io/docs/home/)
