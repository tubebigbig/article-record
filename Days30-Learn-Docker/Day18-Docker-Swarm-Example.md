###### tags: `鐵人賽` `docker`

# Day18 Docker Swarm 實作

感覺昨天講了一堆廢話，沒真正實作的話看不懂，今天將應用 Docker 官方的範例來做 Swarm 的使用並做擴展。

## 實作

### 事前配置

首先先將官方實例 clone 下來再 bulid 就好

```bash
git clone https://github.com/dockersamples/node-bulletin-board
cd node-bulletin-board/bulletin-board-app
docker build --tag bulletinboard:1.0 .
```

接下來再建立 bb-stack.yaml，包含以下的內容(此檔案就如同 compose-file)

```yaml
version: "3.7"

services:
  bb-app:
    image: bulletinboard:1.0
    ports:
      - "8000:8080"
```

在使用之前要先確認 Swarm 功能是有運行的，如果沒有就初始 Swarm，Docker 就會自動建立 Swarm manager 能先體驗一下功能

```bash
docker swarm init
```

### 建立 Swarm service

一切就緒之後就可以 deploy 來使用我們的 stack

```bash
$ docker stack deploy -c bb-stack.yaml mystack
Creating network mystack_default
Creating service mystack_bb-app
```

部屬好我們的 stack 之後可以使用 service 命令來觀察我們建立的 service，可以看到 service ID 還有一些詳細資訊

```bash
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
27g4pcawle5o        mystack_bb-app      replicated          1/1                 bulletinboard:1.0   *:8000->8080/tcp
```

也可以觀察 node 這昨天提過

再來是做擴展的功能，擴展也是使用 service 指令，格式如下

```bash
docker service scale <SERVICE-ID>=<NUMBER-OF-TASKS>
```

再搭配我們剛剛觀察到的 service ID 建立擴展十個，就會增加十個 task

```bash
$ docker service scale 27g4pcawle5o=1027g4pcawle5o scaled to 10
27g4pcawle5o scaled to 10
overall progress: 10 out of 10 tasks
1/10: running   [==================================================>]
2/10: running   [==================================================>]
3/10: running   [==================================================>]
4/10: running   [==================================================>]
5/10: running   [==================================================>]
6/10: running   [==================================================>]
7/10: running   [==================================================>]
8/10: running   [==================================================>]
9/10: running   [==================================================>]
10/10: running   [==================================================>]
verify: Service converged
```

再回去查看我們的 service，REPLICAS 就變成了十個

```bash
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
27g4pcawle5o        mystack_bb-app      replicated          10/10               bulletinboard:1.0   *:8000->8080/tcp
```

也能透過 ps 來查看對應 service 的資訊

```bash
$ docker service ps 27g4pcawle5o
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
qphzkfmn5c7t        mystack_bb-app.1    bulletinboard:1.0   docker-desktop      Running             Running 26 minutes ago
8qna1q65sdsb        mystack_bb-app.2    bulletinboard:1.0   docker-desktop      Running             Running 5 minutes ago
xuqussywhffl        mystack_bb-app.3    bulletinboard:1.0   docker-desktop      Running             Running 5 minutes ago
ex0dg88uu5gy        mystack_bb-app.4    bulletinboard:1.0   docker-desktop      Running             Running 5 minutes ago
2h8prgkgpolm        mystack_bb-app.5    bulletinboard:1.0   docker-desktop      Running             Running 5 minutes ago
7y6wtubgi9om        mystack_bb-app.6    bulletinboard:1.0   docker-desktop      Running             Running 5 minutes ago
6vuncarl1is4        mystack_bb-app.7    bulletinboard:1.0   docker-desktop      Running             Running 5 minutes ago
yjroyvbvkhke        mystack_bb-app.8    bulletinboard:1.0   docker-desktop      Running             Running 5 minutes ago
yru1cx4rbu7z        mystack_bb-app.9    bulletinboard:1.0   docker-desktop      Running             Running 5 minutes ago
zfutz1757xpf        mystack_bb-app.10   bulletinboard:1.0   docker-desktop      Running             Running 5 minutes ago
```

這十個擴展就能提供給 Swarm 自動做負載平衡

### 刪除 Swarm service

刪除的語法則是使用`docker service rm <stack-name>`就好

```bash
$ docker service rm 27g4pcawle5o
27g4pcawle5o
```

## 總結:

看了兩天終於看懂 manager 跟 worker 到底是在說甚麼了，不過由於時間已經太晚就只稍微講一下，manager 跟 workder 都是各自的虛擬機或是實體機拉，所以我每次 swarm join 都在 manager 自然會報錯，真是搞笑，好啦不過我們今天已經將 Swarm 基本功能玩過了。接下來來研究怎麼部署我們的 container image 到 cloud 吧，是跟 docker hub 有關嗎?

參考文獻:
[docker 官方文件](https://docs.docker.com/)
