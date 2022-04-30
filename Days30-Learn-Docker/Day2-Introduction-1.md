###### tags: `鐵人賽` `docker`

# Day2 開始體驗 Docker 吧(一)

## Docker 具有...

一開始要學習一個新技術當然是先看 source code...咳咳先等等，既然不是大神的小弟我還是不要如此逾矩，乖乖地從官方文件開始看起吧。
首先一進來首頁先來了解 Docker 有提供三種功能

### Docker Desktop

Docker Desktop 是最簡單用來本地端開發使用的工具，並本身已經包含了 Docker Engine、Docker CLI client, Docker Compose, Kubernetes etc.，基本上需要使用 Docker 的功能都能從這邊開始。由於是本地端運行，因此很適合快速建置用來測試及除錯 application。

### Docker Hub

Docker Hub 是 Docker 提供的雲端服務，能夠用以共享 container images 且便於團隊合作的服務。
具有以下幾個功能:

- Repositories
  可以想成與 github 以及 gitlab 的 Repositories，就是一個倉庫的概念，能夠上傳以及下載 container images。
- 團隊與組織
  投過團隊及組織讓儲存 image 的倉庫(repository)能夠輕鬆的共享及管理。
- Official/Publisher images
  由於 Docker hub 能夠選擇公開自己的 images，因此可以輕鬆獲得官方以及第三方提供的 images 來使用。
- Automated builds
  Docker hub 可以自動建立來自額外倉庫(EX:github、Bitbucket 等你想得到的地方)的 images，並將建構好的 images 推送至 Docker repository，讓你不需要再將專案搬來搬去輕鬆達成 CI。

### Play with Docker

由 Docker 集成提供的各項 Docker 教學，讓你除了官方的文件外也能來此快速練功升級。

---

## Docker 快速體驗

接下來就讓我們開始來體驗 Docker 吧，首先先從官方下載好 Docker Desktop 並且執行。
等到 Docker 開始運作後便能依照 Docker Desktop DashBoard 的教學開始體驗。

首先先打開 terminal 並執行

```shell
docker run -dp 80:80 docker/getting-started
```

> - -d 代表將 container 以 detached mode 執行在背景(這段看不太懂之後看懂再來補充)註 1。
> - -p 80:80 代表將本機端的 port 映射到 container 的 port
>   (本機:container)

執行完就能看到 Docker DashBoard 上新增了 docker/getting-started 的 container。getting-started 包含了 docker 的教學文章，接下來變讓我們繼續跟著教學文章走下去。

> 註 1:當我們開始一個 container 時可以選擇將其運行在後台的 Detached mode 或是預設的 foreground mode
>
> - Detached mode
>   在 Detached mode 下除非 container 的 root process exit 或是使用--rm 的指令才會 exit。
> - Foreground
>   在 Foreground mode 下可以在 container 裡啟動 process 並將 console 附加到 process 的標準輸入、輸出以及標準錯誤。更可以使用 commad line option 做更多的配置，像是使用-t 將 process 分配成假的 TTY、-i 可以在 STDIN 沒有附加的情況下保持打開

參考文獻:
[docker 官方文件](https://docs.docker.com/)
