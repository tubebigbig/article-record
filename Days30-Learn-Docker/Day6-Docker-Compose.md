###### tags: `鐵人賽` `docker`

# Day6 Docker Compose 組在一起比較香

這幾天我們學習了如何建立 Container image、建置運行 APP、使用 Volumes、Bind Mounts 以及建立自己的 MySQL，但是每次執行、更新、停止都要一個一個指令實在太花時間了，難道沒有辦法一次就能執行完嗎!?

沒問題！就讓 Docker Compose 來解決你的困擾。

## Docker Compose

Docker Compose 只要一行指令就能執行 multi-container 的全部動作！只需要透過 YAML 定義好 APP 的設置，相等於使用批次檔管理檔案的概念。

接下來讓我們開始來實作 Docker Compose 吧!

首先要先定義好 APP 的設置因此從各個建置 container image 的命令看起吧，昨天建立的 multi-container 共有 APP 跟 MySQL 這兩個 container 因此接下來就根據這兩個來建立。

### 建置 Docker Compose

首先需要在 APP 的 root 底下建立 compose 檔案，因此先建立`docker-compose.yml`，並先寫上要使用的版本。

_docker-compose.yml_

```yaml
version: "3.8"
```

### APP

昨天我們在執行 APP 的時候使用了以下的指令

```bash
docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

根據以上的命令編寫至 compose 檔案裡

_docker-compose.yml_

```yaml
version: "3.8"
services:
  # services 代表需要執行的containers
  app:
    # container 名稱
    image: node:12-alpine
    # image FROM
    command: sh -c "yarn install && yarn run dev"
    # 帶入-c flag的指令
    ports:
      - 3000:3000
    working_dir: /app
    # 帶入-w flag的路徑
    volumes:
      - ./:/app
    # 帶入-v flag的路徑
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
    # 帶入-e flag的內容
```

### MySQL

昨天我們在執行 MySQL 的時候使用了以下的指令

```bash
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```

根據以上的命令編寫至 compose 檔案裡

_docker-compose.yml_

```yaml
version: "3.8"

services:
  app:
    # The app service definition
    # ...
  mysql:
    image: mysql:5.7
    # image FROM
    volumes:
      - todo-mysql-data:/var/lib/mysql
    # 帶入-v flag的路徑
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos
    # 帶入-e flag的內容

volumes:
  # 在這邊指定的volumes會提供給全部的services
  todo-mysql-data:
```

最後 compose 則會呈現這樣

```yaml
version: "3.8"
services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

### 執行 Docker Compose

在我們擁有 compose 檔案之後就可以開始執行!!
執行以下命令

```bash
docker-compose up -d
```

> 還記得我們執行 APP、MySQL 都會使用的 -d flag 嗎，其實不是忘記加在 compose 檔案裡呢，而是在這邊執行就好了。

執行完可以在輸出看到

```bash
Creating network "app_default" with the default driver
Creating volume "app_todo-mysql-data" with default driver
Creating app_app_1   ... done
Creating app_mysql_1 ... done
```

> 可以看到 Docker Compose 很貼心的幫我們建立了網路來連接 volume 了

### 移除 Docker Compose

如果要移除 Docker Compose 也是非常方便
只要執行以下命令，既然開始是 up 那結束自然是 down 瞜

```bash
docker-compose down
```

> 這邊要注意的是，移除 compose 的時候不會幫你移除 volume，要一同移除 volume 則需要加上`--volumes` flag

## 總結:

終於把基礎教學學習完了，也了解了 Image Layering、Layer Caching 以及 Multi-Stage Builds 的概念，只是篇幅會太長就決定不加入此次文章。
這次的挑戰是打算只查看相關的官方文件而不參考他人文章教學的作法，因此有些不太懂得地方會先以主觀認知帶過，用意在於能夠不被他人的主觀意識影響，自行去了解其概念理論。
接下來的預估了幾個目標，例如簡單了解雲端部屬 Docker、K8s 等進階管理使用 Docker 的方法，之後再來詳細研究 Docker 常用的命令及各種 flag，以及 Dockerfile、Docker Compose file 的寫法，預期還會有 Docker api 的使用 就敬請期待了。

參考文獻:
[Docker 官方文件](https://docs.docker.com/)
