###### tags: `鐵人賽` `docker`

# Day20 MongoDB with Docker

今天來學習如何在 Docker 內使用 MongoDB。

## 建立 MongoDB

要使用 Docker 建立 MongoDB 首先要取得映像檔，直接去 Docker hub 查看映像檔的版本，因為只是要學習直接用 latest 版本

```bash
docker pull mongo
```

取得映像檔之後就可以開始建置，但是在此之前先建立一個儲存資料的地方，因此先建立一個資料夾。

```bash
mkdir mongodb
```

接下來就能建置 mongoDB(mongodb 預設的 port 為 27017)，如果不需要公開可以將`-p 27017:27017`移除

```bash
docker run -it -v mongodb:/data/db -p 27017:27017 --name mongodb -d mongo
```

建立好之後可以使用 logs 查看 container 的 log，有新刪修也會顯示在上面

```bash
docker logs mongodb
```

建立好 mongodb 後我們可以使用 exec 連接到 container 以使用 mongodb 的指令

```bash
$ docker exec -it mongodb bash
root@<container ID>:/#
```

## Docker secrets

既然學到了 MongoDB 就一起來學 Docker secrets，當我們在 Docker swarm 建立 MongoDB 的時候需要配置 username 跟 password，但是總不可能直接將其寫在配置檔中，其他機敏資料還包含 SSH 私鑰、證書等等，因此 Docker 有一套存放機敏資料的方法。這些機敏資料存在 Docker Swarm 處於靜止時會是加密狀態，只有被授權的服務且處於運行中才可以訪問。

加密的文件預設會儲存在`/run/secrets/<secret_name>`

接下來讓我們來學習如何使用 Docker secrets
如官方的範例，我們先用`secret`指令建立 my_secret_data

```bash
docker secret create my_secret_data -
```

再使用`--secret`flag 指定要使用的 secret 檔案

```bash
docker service  create --name redis --secret my_secret_data redis:alpine
```

建置好 redis 也可以透過 exec 查看使用的 secret，顯示就會是你剛剛設定的 secret 內容

```bash
docker container exec $(docker ps --filter name=redis -q) cat /run/secrets/my_secret_data
```

接下來讓我們實用 secret 至 mongoDB

## MongoDB with secret

首先建立 mongodb_root_username，並使用 vim 編輯器輸入 username

```bash
$ touch mongodb_root_username
$ vim mongodb_root_username
$ docker secret create mongodb_root_username ./mongodb_root_username
```

再建立 mongodb_root_password，並使用 vim 編輯器輸入 password

```bash
$ touch mongodb_root_password
$ vim mongodb_root_password
$ docker secret create mongodb_root_password ./mongodb_root_password
```

建立好之後可以使用 ls 查看所有的 secret 設置

```bash
$ docker secret ls
ID                          NAME                    DRIVER              CREATED              UPDATED
0r8bsldt0uh0tf5lczgh6k8ky   mongodb_root_password                       About a minute ago   About a minute ago
ufbpe94tzxpixpm6fhhdd9ctm   mongodb_root_username                       About a minute ago   About a minute ago

```

最後配置好 swarm YAML 檔，就可以使用拉。

```yaml
version: "3.8"
services:
  mongodb:
    build: ./mongodb
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_PASSWORD_FILE: /run/secrets/mongodb_root_password
      MONGO_INITDB_ROOT_USERNAME_FILE: /run/secrets/mongodb_root_username
      MONGO_INITDB_DATABASE: database01
      MONGO_DATABASE: todolist
    secrets:
      - mongodb_root_password
      - mongodb_root_username
secrets:
  mongodb_root_password:
    external: true
  mongodb_root_username:
    external: true
```

參考文獻:
[Manage sensitive data with Docker secrets](https://docs.docker.com/engine/swarm/secrets/)
[Docker hub MongoDB](https://hub.docker.com/_/mongo)
