###### tags: `鐵人賽` `Kubernetes`

# Day11 深入淺出 Kubernetes(四)

不知不覺已經十一天了呢，看著自己訂下的標題果然是深入淺出呢，要學的東西還很深只能寫一些淺的東西，順手翻閱了一下 kubernetes 到底有哪些工具可以使用，畢竟不可能一直都使用 minikube，瞬間資料多的琳琅滿目甚麼 Kubelet、Kubeadm、Helm、Dashboard、Kops 感受到了滿滿的惡意，不過也確信了自己的學習之路不會無聊，接下來就先讓自己經過官方文件的歷練接受更大的挑戰吧!

## Scaling APP

擴展?為甚麼要擴展?擴展其實就是增加 Pod 的複製品，首先最直觀的就是，當應用程序運行發生錯誤或是崩潰的時候，我們還是要維持應用程序的運行，因此就需要替代品來讓錯誤或是崩潰發生的當下能夠立刻切換，而這整個過程就是擴展的價值，擴展可以分成直接使用 ReplicaSet 或是搭配 Deployment 使用，而官方是這麼說明 ReplicaSet 的

> A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.
> 總的來說就是如果只單用 ReplicaSet 會錯失 Deployments 強大的功能，除非是不需要那就可以單用瞜。

**ReplicaSet**
簡單看個 ReplicaSet 的範例配置檔以及應用方法，建置 Pod 的方法也要包含在裡面的原因，是為了在不得不需要建立 Pod 的情況使用，也就是當所有 Pod 都壞光了。

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web
  labels:
	env: dev
	role: web
spec:
  #  要複製幾個，預設是1
  replicas: 4
  selector:
	matchLabels:
  	  role: web
  template:
	metadata:
  	  labels:
    	    role: web
	spec:
  	  containers:
  	  - name: nginx
    	    image: nginx
```

要使用也很簡單，apply 就好

```bash
kubectl apply -f nginx_replicaset.yaml
```

要查看複製的 Pod 也很簡單使用 get 加上 rs 就可以了

```bash
kubectl get rs
```

**搭配 Deployment 擴展**
搭配 Deployment 跨展的方法，就延續前幾天的範例繼續了。
在 Deployment 且 expose 好應用程序之後，就可以使用 scale 指令來擴展我們的 deployment，scale 後面接著的是 deploments type 以及名稱

```bash
kubectl scale deployments/kubernetes-bootcamp --replicas=4
```

查看我們的 pods 可以發現 pods 從一個變四個了

```bash
PS ~\Users> kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-57978f5f5d-pdhn4   1/1     Running   0          2m20s
kubernetes-bootcamp-57978f5f5d-ppdvr   1/1     Running   0          2m20s
kubernetes-bootcamp-57978f5f5d-qq59m   1/1     Running   0          5m47s
kubernetes-bootcamp-57978f5f5d-vnv62   1/1     Running   0          2m20s
```

再 describe 看我們的 service，可以看到 Endpoints 的地方就有了四個擴展 Pod 的 IP address

```
PS ~\Users> kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   app=kubernetes-bootcamp
Annotations:              <none>
Selector:                 app=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.101.106.157
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30384/TCP
Endpoints:                172.18.0.3:8080,172.18.0.4:8080,172.18.0.5:8080 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

## Rolling Update

Rolling Update 的意思就是，像是我們剛剛上面創造了三個 Pod 的複製品，那當今天要更新的時候難道需要一個一個對應 Pod name 來更新嗎？所以就有了 Rolling Update 的方法，只要其中一個 Pod 更新其他的複製品也會隨著一起更新。
使用 Rolling Update 的好處不只是這些，由於 Pod 會依序更新，所以上線中的服務不需要停止也可以直接更新，只需要由 kubernetes 幫我們更換 Pod 就好了，而這些變動使用者都不會發覺，至於更新的條件就是要先有擴展拉。
那如果是負載平衡也不會因此亂套，服務會自動對剩餘可用的 Pod 進行負載平衡。

**實作 Rolling Update**
接下來跟著官方文件將我們的應用程序的 image 更新至 version2 吧，再執行完之後查看 pods 可以看到正在更新的 pod 已經正要被刪除的 pod

```bash
PS ~\Users> kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
deployment.apps/kubernetes-bootcamp image updated
```

```bash
PS ~\Users> kubectl get pods
NAME                                   READY   STATUS              RESTARTS   AGE
kubernetes-bootcamp-57978f5f5d-6qc7h   1/1     Running             0          17m
kubernetes-bootcamp-57978f5f5d-dhgfp   1/1     Running             0          17m
kubernetes-bootcamp-57978f5f5d-mcphx   1/1     Terminating         0          17m
kubernetes-bootcamp-57978f5f5d-qq59m   1/1     Running             0          33m
kubernetes-bootcamp-769746fd4-4glvb    0/1     ContainerCreating   0          2s
kubernetes-bootcamp-769746fd4-fc829    0/1     ContainerCreating   0          2s
```

再查看一次

```bash
PS ~\Users> kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-769746fd4-4glvb   1/1     Running   0          3m59s
kubernetes-bootcamp-769746fd4-548g9   1/1     Running   0          3m37s
kubernetes-bootcamp-769746fd4-fc829   1/1     Running   0          3m59s
kubernetes-bootcamp-769746fd4-s9jkp   1/1     Running   0          3m44s
```

基本上這個時候就更新完成拉，接下來讓我們訪問程序吧，理論上是透過 describe 指令找到 service 的 port，不過我們是 VM 所以要使用 minikube service 建立 VM 與我們的通道，最後使用 curl 訪問應用程序可以看到 response 的 content 已經變成 v=2 就代表升級成功拉。

```bash
PS ~\Users> minikube service kubernetes-bootcamp --url
🏃  Starting tunnel for service kubernetes-bootcamp.
|-----------|---------------------|-------------|------------------------|
| NAMESPACE |        NAME         | TARGET PORT |          URL           |
|-----------|---------------------|-------------|------------------------|
| default   | kubernetes-bootcamp |             | http://127.0.0.1:51818 |
|-----------|---------------------|-------------|------------------------|
http://127.0.0.1:51818
❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.
PS ~\Users> curl "http://localhost:51818"


StatusCode        : 200
StatusDescription : OK
Content           : Hello Kubernetes bootcamp! |
                    Running on: kubernetes-bootcamp-769
                    746fd4-fc829 | v=2

...(省略)
```

**如何退版**
當更新失敗或是錯誤的時候自然需要退回到穩定的版本，接下來實際演練退版的方法
更新至一個會錯誤的版本 v=10

```bash
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10
```

透過查看 deployments 以及 pods 可以發現有些問題，Pod 的狀態提示我們要退回一版

```bash
PS ~\Users> kubectl get deployments              NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   3/4     2            3           57m
PS ~\Users> kubectl get pods
NAME                                  READY   STATUS             RESTARTS   AGE
kubernetes-bootcamp-597654dbd-h9pv6   0/1     ImagePullBackOff   0          80s
kubernetes-bootcamp-597654dbd-pmbfw   0/1     ImagePullBackOff   0          81s
kubernetes-bootcamp-769746fd4-4glvb   1/1     Running            0          24m
kubernetes-bootcamp-769746fd4-fc829   1/1     Running            0          24m
kubernetes-bootcamp-769746fd4-s9jkp   1/1     Running            0          24m
```

既然如此就來退版吧，通過使用 rollout 指令來指定我們的 deloyment undo 回上一版

```bash
PS ~\Users> kubectl rollout undo deployments/kubernetes-bootcamp
deployment.apps/kubernetes-bootcamp rolled back
PS ~\Users> kubectl get pods
NAME                                  READY   STATUS              RESTARTS   AGE
kubernetes-bootcamp-769746fd4-4glvb   1/1     Running             0          30m
kubernetes-bootcamp-769746fd4-fc829   1/1     Running             0          30m
kubernetes-bootcamp-769746fd4-kfxqh   0/1     ContainerCreating   0          2s
kubernetes-bootcamp-769746fd4-s9jkp   1/1     Running             0          29m
kubernetes-bootcamp-597654dbd-h9pv6   0/1     Terminating         0          2m10s
kubernetes-bootcamp-597654dbd-pmbfw   0/1     Terminating         0          2m11s
```

可以看到錯誤的 Pod 正在被移除，而退版也會跟著 Rolling Update 機制將所有 Pod 還原回去

## 總結:

今天很開心又到了下一個階段，因為將 k8s 的官方 basic 教學學完了。接下來為了不脫離主題也算是為之後要實作項目做努力，要先回去鑽研 Docker 了，先來預告明天的目標，明天就要來研究該如何寫 Dockerfile。

參考文獻:
[Kubernetes 官方文件](https://kubernetes.io/docs/home/)
