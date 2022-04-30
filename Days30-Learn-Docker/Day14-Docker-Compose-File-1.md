###### tags: `鐵人賽` `docker`

# Day14 Docker Compose file 指令大集(一)

今天開始要來學習的是 Docker compose 時使用的 YAML file 的指令介紹跟格式。昨天提到了今天要來實作 Dockerfile，但是發現大多都是單純命令的使用 or 混雜 shell 等的使用，而不是各情境的命令使用方式，因此在了解完之後還是決定改來撰寫學習 compose file 指令的文章。

不過有個可以介紹的是，Dokcer 官方推薦在建立時使用 alpine 的 image，特色在於其 image 大小不到 5MB，因為它移除了大量不必要的套件，而 image 容量小建置速度自然快，不過這邊也有說如果要使用 Python 則不建議，有興趣這邊提供[連結](https://pythonspeed.com/articles/alpine-docker-python/)

## 指令介紹

### build

如名，建置時的配置。
build 包含以下幾種配置

- CONTEXT
  包含 Dockerfile 的路徑或是 URL

```yaml
build:
  context: ./dir
#   Dockerfile在當前路徑的dir資料夾裡
```

- DOCKERFILE
  用來當作備用 Dockerfile 的路徑

- ARGS
  如同 Dockerfile 的 ARG 可以用來當作變數使用
  首先需要在 Dockerfile 做參數使用的設定

```dockerfile
ARG param

RUN echo "ARGS TEST: $param"
```

再來從 compose file 設定數值

```yaml
build:
  context: ./dir
  args:
    param: 1
```

- CACHE_FROM
  用來配置可以暫存的 image，因此可以增加建置速度。任何 image 都可以

```yaml
build:
  context: ./dir
  cache_from:
    - alpine:latest
# 如例cache alpine的建置，如果有重複使用image的Dockerfile就可以更快被建置
```

也可以在 build 的時候用 flag 使用

```shell
$ docker build --cache-from=alpine:latest -t app/test .
```

- LABELS
  跟 Dockerfile 的 LABELS 同功能，標籤會被添加到之後建置完的 image 裡

- NETWORK
  作為最上層的網路配置管理，透過官方提供的例子就能了解，proxy、app 以及 db 都有使用網路配置，而上層的 networks 則是統一管理的概念。

```yaml
version: "3"
services:
  proxy:
    build: ./proxy
    networks:
      - frontend
  app:
    build: ./app
    networks:
      - frontend
      - backend
  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    # Use a custom driver
    driver: custom-driver-1
  backend:
    # Use a custom driver which takes special options
    driver: custom-driver-2
    driver_opts:
      foo: "1"
      bar: "2"
```

- SHM_SIZE
  這裡是用來配置 SHM 的大小。
  先來簡單介紹 SHM，`/dev/shm`是一種共享內存的功能，可以讓 Linux 的程序能夠互相傳遞數據，假設 A 程序創建了一部分的內存，B 程序可以透過共享訪問以增加處裡速度。
  大小配置必須是字串表示的 byte value 或是數值

```yaml
build:
  context: ./dir
  shm_size: '1024kb'
#   or
  shm_size: 10000000

```

- TARGET
  這是包含在之前沒介紹的 multi-stage build 功能的應用，先給連結[multi-stage build 介紹](https://)，這邊是可以填上 Dockerfile 內的定義指定階段的名稱，對其進行相關配置。

```yaml
build:
  context: ./dir
  target: prod
```

Dockerfile 內的 prod 就像這樣

```dockerfile
FROM golang:1.7.3 AS prod
...

FROM alpine:latest
...
COPY --from=prod app/test .
```

## 總結:

今天學習完了實際應用 Dokcerfile 以及 Docker compose file 的 build 功能，看似 compose file 的命令有點多，明天會努力學習希望能寫到一半。

參考文獻:
[docker 官方文件](https://docs.docker.com/)
