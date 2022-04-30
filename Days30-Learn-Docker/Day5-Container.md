###### tags: `鐵人賽` `docker`

# Day5 container 混戰拉!

昨天學習到了 Volumes 以及 Bind Mounts 的用法，要開發測試選擇 Bind Mounts，要單純存取數據選擇 Volumes 但是今天想要建立自己的 MySQL 呢?

笨阿!摻在一起做 container 阿!

其實沒有這麼簡單唷，由於 Docker 推薦的就是一個 container 就只做一件事，為甚麼呢?

- 首先，資料庫與服務端分開是蠻常見的做法，除了可以使用多個資料庫 Server 來減輕存取負擔，一般也會再多一個備用的環境來當作緊急時的調度使用，可以避免服務被迫終止的情況發生，因此分開可說是好處多多呢。
- 再來就是如果要在一個 container 中擁有多個進程，就需要有個管理進程的做法，在啟動跟關閉的執行上就變得比較複雜了，事情簡單點大家才能早點下班呢。
- 最後就是 APP 版本管理方便等等就不再說明了

## Multi-Container Apps

上面說明了這麼多，接下來就來談談，我們該怎麼建立 MySQL 並與 APP 交流呢。
仔細想想一般 APP 怎麼連接 MySQL?都是需要進行 DB 連接對吧!
那兩個不相關的服務為甚麼可以連接呢?就是因為 ── 網路

> If two containers are on the same network, they can talk to each other. If they aren't, they can't.

需要連接兩者就代表之間必須在同個網路底下，不然是無法進行溝通的。
接下來就讓我們簡單建立本地的 MySQL 來玩玩吧

首先先建立網路

```bash
docker network create todo-app
```

接下來就是建立 MySQL 的 container 並連接至網路

```bash
docker run -d \
    --network todo-app --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=todos \
    mysql:5.7
```

> -d 就是前幾天提到的 detached mode
> --network 就是指定剛剛建立的網路
> --network-alias 代表幫網路取個小名
> -e 代表為 Docker 建立的環境變量
> -v 就是前幾天提到的 Volumes

接下來就能使用指令連接資料庫，進行資料庫的操作。

```bash
docker exec -it <mysql-container-id> mysql -p
```

建立好了網路以及資料庫，接下來就是要連接我們的 APP 與 MySQL 了，我們知道連接就需要 IP 位置，但是該怎麼取得呢，其實可以直接進去 MySQL container 敲敲打打指令就能出來了，但是官方文件推薦我們使用 nicolaka/netshoot 建立 container，除了可以查看 IP 還有許多強大的功能，這些就之後再研究看看瞜...

執行以下命令使用 nicolaka/netshoot 建立 container

```bash
docker run -it --network todo-app nicolaka/netshoot
```

> -it 的意思呢就是--interactive + --tty
> -i or --interactive 意思是保持 STDIN 的開啟
> -t or --tty 是分配一個偽 tty

執行完就會進入 nicolaka/netshoot 建立的 container 的環境，要查詢 IP 位址則需要使用 dig 指令

```bash
dig mysql
```

output

```
; <<>> DiG 9.14.12 <<>> mysql
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24570
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;mysql.                         IN      A

;; ANSWER SECTION:
mysql.                  600     IN      A       172.18.0.2

;; Query time: 26 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Fri Sep 18 15:09:27 UTC 2020
;; MSG SIZE  rcvd: 44
```

> 在 ANSWER SECTION 底下 A 之後的位址就是我們要找的 MySQL IP

現在有 IP 之後我們終於可以連接資料庫了!!

將 APP 連接資料

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

執行完後便可以觀測 container 的 LOGS 直到出現 Connected!才代表連接 MySQL 成功唷

```
$ nodemon src/index.js

[nodemon] 1.19.2

[nodemon] to restart at any time, enter `rs`

[nodemon] watching dir(s): *.*

[nodemon] starting `node src/index.js`

Waiting for mysql:3306.

Connected!

Connected to mysql db at host mysql

Listening on port 3000
```

連接成功之後就能增加幾項 todo item
再使用連接資料庫的指令來查詢看看剛剛新增的項目

```bash
docker exec -ti <mysql-container-id> mysql -p todos
mysql> select * from todo_items;
+--------------------------------------+-----------+-----------+
| id                                   | name      | completed |
+--------------------------------------+-----------+-----------+
| 9b0cbb70-1c2a-4f16-85f3-79784092479b | qweeqeqwe |         0 |
| af930116-e857-4381-9ee3-92229a42b0fc | qqqqq     |         0 |
+--------------------------------------+-----------+-----------+
2 rows in set (0.00 sec)
```

## 總結

今天週末了有點懶散呢，不過還是把目標做到了，學習到了 Multi-Container Apps 的做法，但是如果 container 都是散成一團的話確實有點難管理，因此，明天就要來學習甚麼是 Docker Compose，並且趁著假日訂定之後的目標吧。

參考文獻:
[Docker 官方文件](https://docs.docker.com/)
