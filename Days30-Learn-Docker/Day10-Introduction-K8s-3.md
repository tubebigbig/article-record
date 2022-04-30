###### tags: `鐵人賽` `Kubernetes`

# Day10 深入淺出 Kubernetes(三)

昨天學會了如何觀測並排查錯誤，但是要訪問 APP 總不可能都用 Proxy 監聽吧。所以呢，今天就要來學習何謂 Services 以及如何 expose 我們的 APP。

## Service

是一個能夠將 Pod 上的應用程序公開的方法。僅需要撰寫好 YAML 檔，K8s 就會自動幫我們運作。在公開的時候 K8s 會為 Pod 提供屬於自己的 IP address 以及一個 DNS 名稱。

那為甚麼要公開我們的應用程序呢？

假設今天後端跟前端各有自己的 Pod，當他們之間要互相交互的時候就必須透過某個方式找到對方，尤其是當今天後端有許多的 Pod，每次運行的時候都會使用到不同的 Pod，就需要自己來連接後端的 Pod 到前端；但是今天我們只需要公開 Pod，那剩下的事情 K8s 就會幫我們解決。

首先查看一個 Service 的範例 YAML 檔，當我們運行此檔案時，Service selector 的 controller 會不斷尋找跟 selector 匹配的 label(如例則是 app = my-first-app)以及 TCP port 為 9487 的 Pod，再將所有 Pod 發佈到"first-service"的地方。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: first-service
spec:
  selector:
    app: my-first-app
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9487
```

而 Service 也可以公開多個 ports，只要稍微修改 YAML 檔的設定，如下則是公開了 http 以及 https 兩個 port。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: first-service
spec:
  selector:
    app: my-first-app
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9487
    - name: https
      protocol: TCP
      port: 443
      targetPort: 8484
```

當然還有許多 Service YAML 檔的設定方法，想要了解的就再去深入摟。[Service 介紹](https://kubernetes.io/docs/concepts/services-networking/service/)

## 公開 APP

首先，先依照前幾天的作法將我們的 APP deploy 好。
再確認好 APP running 之後，先來學習使用查看 service 狀態的命令。

```bash
kubectl get services
```

執行完的輸出的 Kubernetes 就是我們 K8s 本體。

```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   52m
```

**1.公開**
再來就來使用 expose 命令公開我們的 APP。

```bash
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
```

output

```
service/kubernetes-bootcamp exposed
```

**2.查看公開後狀態**
公開完再來查看一次我們的 services，可以看到我們的 APP 已經公開了，並且公開在 30177 PORT。

```bash
kubectl get services
```

output

```bash
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP          57m
kubernetes-bootcamp   NodePort    10.104.239.126   <none>        8080:30177/TCP   5s
```

除了可以從 get services 看到 PORT，也能透過 describe 查看 PORT 以及更多詳細資訊。

```bash
kubectl describe services/kubernetes-bootcamp
```

output

```
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   app=kubernetes-bootcamp
Annotations:              <none>
Selector:                 app=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.104.239.126
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30177/TCP
Endpoints:                172.18.0.2:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

如果是執行在本機端就能直接訪問了，但是我們是建立在 docker 上的因此還需要建立外部訪問 Pod 的通道。
透過 minikube 提供的 service 命令就能建立通道了。

```bash
minikube service kubernetes-bootcamp --url
```

output

```
🏃  Starting tunnel for service kubernetes-bootcamp.
|-----------|---------------------|-------------|------------------------|
| NAMESPACE |        NAME         | TARGET PORT |          URL           |
|-----------|---------------------|-------------|------------------------|
| default   | kubernetes-bootcamp |             | http://127.0.0.1:55754 |
|-----------|---------------------|-------------|------------------------|
http://127.0.0.1:55754
❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.
```

接下來訪問 localhost:55754 就能看到我們的 APP 了!!

## label

**查看 label**
還記得我們上面提過 label，如果沒有指定的話，在我們部屬的時候 K8s 會為我們自動建立名稱，能夠透過 describe 指令查看 deployment，就可以找到我們的 label。

```bash
kubectl describe deployment
```

output

```
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Wed, 23 Sep 2020 22:59:30 +0800
Labels:                 app=kubernetes-bootcamp
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=kubernetes-bootcamp
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
...(省略)
```

**修改 label**
在部屬完後想要修改也是可以的，只要使用 label 指令即可(Pods 詳細名稱從 describe 取得)。

```bash
kubectl label pod kubernetes-bootcamp-57978f5f5d-s9pkm test=v1
```

output

```
pod/kubernetes-bootcamp-57978f5f5d-s9pkm labeled
```

再來查看修改後的 label，可以看到已經將 test=v1 增加進去了。

```bash
kubectl describe pods kubernetes-bootcamp-57978f5f5d-s9pkm
```

output

```
Name:         kubernetes-bootcamp-57978f5f5d-s9pkm
Namespace:    default
Priority:     0
Node:         minikube/172.17.0.3
Start Time:   Wed, 23 Sep 2020 22:59:30 +0800
Labels:       app=kubernetes-bootcamp
              pod-template-hash=57978f5f5d
              test=v1
Annotations:  <none>
Status:       Running
IP:           172.18.0.2
IPs:
  IP:           172.18.0.2
Controlled By:  ReplicaSet/kubernetes-bootcamp-57978f5f5d
Containers:
...(省略)
```

## 刪除 service

如同 Day6 所學的可以使用 delete 指令再搭配今天學的 label 來指定刪除 service，刪除之後就不能透過外部來訪問 Pod 了。

```bash
kubectl delete service -l app=kubernetes-bootcamp
```

output

```
service "kubernetes-bootcamp" deleted
```

## 總結:

今天了解了 Service、label 以及 expose pod 的方法，明天就來學習如何 How to scale my app 以及透過 Rolling Update 來更新 APP。

參考文獻:
[Kubernetes 官方文件](https://kubernetes.io/docs/home/)
