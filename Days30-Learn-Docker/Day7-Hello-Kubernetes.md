###### tags: `鐵人賽` `Kubernetes`

# Day7 你好 Kubernetes

雖然還不算學習完 Docker，不過有點喜歡 K8s 就先來學習他一下。
Kubernetes 俗稱 K8s 是一套開源的平台能夠自動部屬以及管理多個容器(containerized)的工作量和服務，來談談他具有的幾種特性。

- Service discovery and load balancing
  `可以expose container以及透過負載平衡、分配網路流量使其運行順暢。`
- Storage orchestration
  `可以自動掛載自行決定的儲存空間，不論是在local或是cloud。`
- 自動部屬以及 rollbacks
  `自動部屬的好處應該不用說了吧，可以自動幫你部屬container，或是刪除舊的container再將資源配置給予新的container。`
- Automatic bin packing
  `待補 想不到更好的解釋`
- 自我修復
  `自動重啟失敗的容器或是替換容器`
- Secret and configuration management
  `可以在不用重建container的狀況下部屬或更新機敏資料或是環境設定，因此不需要將機敏資料設置在一開始的建置檔案`

## 環境建置

> 雖然 Docker Desktop 就有提供 Kubernetes 的功能了，但是小弟搞了一個多小時也沒搞好就放棄了，想到以後還是會在 linux 使用還是先來學 Minikube 吧。

要使用 Kubernetes 首先先安裝`minikube`以及`kubectl`，Minikube 是一套用來本地端開發使用得工具，能夠建立單節點 Kubernetes 集群，雖然單節點而已但是拿來個人學習使用已經夠用了。kubectl 則是一套命令行工具用來控制 Kubernetes 集群。

安裝完之後就能開始我們的第一個 Kubernetes 了。
首先使用 minikube 來開始。

```bash
minikube start --driver=docker
```

> --driver 代表要執行的 driver，這邊是使用 Docker，如果不填的話則是代表在主機而非虛擬機執行，除非是專業人士不然一般不建議... [driver flag 介紹](https://kubernetes.io/docs/setup/learning-environment/minikube/#specifying-the-vm-driver)

建立完 cluster 接下來就換 kubectl 登場了!使用 kubectl 創建一個名為 echoserver 的範例 image 的簡單實例

```bash
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
```

執行完的輸出

```
deployment.apps/hello-minikube created
```

對了!這時候還沒結束唷要訪問我們建立的 image 還需要將其公開才行
使用 expose 將 image 公開

```bash
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

執行完的輸出

```
service/hello-minikube exposed
```

要確認服務是不是有確實公開了可以使用 get pod 確認唷

```bash
kubectl get pod
```

當輸出的 ready 為 1/1 且狀態為 running 就代表成功執行了。

```
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-5d9b964bfb-9hfkf   1/1     Running   0          59s
```

但是我們要如何訪問我們建立且公開的 image 呢?
那就要來取得網址拉，透過`--url`flag 取得網址

```bash
minikube service hello-minikube --url
```

執行完的輸出

```
🏃  Starting tunnel for service hello-minikube.
|-----------|----------------|-------------|------------------------|
| NAMESPACE |      NAME      | TARGET PORT |          URL           |
|-----------|----------------|-------------|------------------------|
| default   | hello-minikube |             | http://127.0.0.1:51903 |
|-----------|----------------|-------------|------------------------|
http://127.0.0.1:51903
❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.
```

訪問輸出上的網址就可以看到我們部屬的實例了，內容大致如下

```


Hostname: hello-minikube-5d9b964bfb-9hfkf

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.3
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://127.0.0.1:8080/

Request Headers:
	accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
	accept-encoding=gzip, deflate, br
	accept-language=en-US,en;q=0.9,zh-TW;q=0.8,zh;q=0.7
	connection=keep-alive
	host=127.0.0.1:51903
	sec-fetch-dest=document
	sec-fetch-mode=navigate
	sec-fetch-site=none
	upgrade-insecure-requests=1
	user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102 Safari/537.36

Request Body:
	-no body in request-

```

如果要刪除實例的話也很簡單只要刪除即可(?
首先先刪除服務(Service)

```bash
kubectl delete services hello-minikube
```

輸出如下

```
service "hello-minikube" deleted
```

再來要刪除部屬(Deployment)

```bash
kubectl delete deployment hello-minikube
```

輸出如下

```
deployment.extensions "hello-minikube" deleted
```

最後再使用 minikube 把剛剛建立的 Kubernetes cluster 停止並刪除即可
停止

```bash
minikube stop
```

輸出如下

```
Stopping "minikube"...
"minikube" stopped.
```

刪除

```bash
minikube delete
```

輸出如下

```
Deleting "minikube" ...
The "minikube" cluster has been deleted.
```

參考文獻:
[Kubernetes 官方文件](https://kubernetes.io/docs/home/)
