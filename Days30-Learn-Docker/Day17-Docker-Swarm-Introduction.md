###### tags: `鐵人賽` `docker`

# Day17 Docker Swarm 介紹

Swarm 翻譯為一大群或是密集的東西，在 Docker 中則是指將多個 Docker 主機組在一起的單個虛擬或是物理機，可以管理多個跨主機部署的 container。Swarm 將全部 Docker 主機組在一起後還是能夠對 container 使用 Docker 的命令，但是是經由 Swarm 執行，並且所有動作會由集群的 manager 來控制。

Swarm 與 Kubernetes 確實有許多相似之處，皆具有高可用性、可擴展性、Load balancing、自我修復等特性，兩者的比較網路上也是有許多文章就自行查看了。不過可以說的是 Swarm 是內建在 Docker 可以說是為 Docker 而生的，其在部屬的時候能直接使用 Dockerfile 而不用編譯，並且 Swarm 能夠直接使用 compose-file 的檔案(某些功能可能需要對映做修改像是 deploy)，在平台的統一性以及使用上的簡易性來說是佔有優勢的。

## 簡單創建 Swarm 指令

要創立 Swarm 可以使用 init 就會教導你怎麼創建

```shell
docker swarm init
```

那當我們建立之後當然要設置 worker 以及 manager，所以可以使用 join 加入兩者

- worker

```shell
docker swarm join-token worker
```

- manager

```shell
docker swarm join-token manager
```

## Play with Swarm

透過我們之前遊玩 Docker 建立的 compose-file，改成用 stack 命令來建立 Swarm。
docker-compose 檔案

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

執行 docker stack deploy 來使用 compose file 建置 Swarm

```shell
?$ docker stack deploy --compose-file docker-compose.yml
 mystack
Creating network mystack_default
Creating service mystack_app
Creating service mystack_mysql
```

可以看到我們的 network 以及兩個服務都建立好了，在 Dashboard 裡面則是長這樣
![](https://i.imgur.com/MmNIDEG.png)

這時候就可以訪問 localhost:3000 來運行我們的 APP

## Swarm node

Swarm 的 node 可以有幾種命令來操作，首先是查看方式

```shell
?$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
ue096u6tpyy4gnidykdnn884x *   docker-desktop      Ready               Active              Leader              19.03.12
```

通常 node ls 會列出多個 node 的資訊，因為我們這個 Swarm 只有一個節點所以才顯示一個
如果要檢查單個節點可以使用`docker node inspect <NODE-ID>`，輸出會是 JSON 格式但是會有點亂，因此可以再用`--pretty` flag 來整理成條列式的顯示方式。

```shell
?$ docker node inspect self --pretty
ID:                     ue096u6tpyy4gnidykdnn884x
Hostname:               docker-desktop
Joined at:              2020-09-28 14:39:45.472015233 +0000 utc
Status:
 State:                 Ready
 Availability:          Active
 Address:               192.168.65.3
Manager Status:
 Raft Status:           Reachable
 Leader:                Yes
Platform:
 Operating System:      linux
 Architecture:          x86_64
Resources:
 CPUs:                  2
 Memory:                1.944GiB
Plugins:
 Log:           awslogs, fluentd, gcplogs, gelf, journald, json-file, local, logentries, splunk, syslog
 Network:               bridge, host, ipvlan, macvlan, null, overlay
 Volume:                local
Engine Version:         19.03.12
---(略)
```

## 總結:

今天就先學到這邊了，Swarm 感覺還有很多可以使用的，不過目前只知道可以 deploy compose file 而已

參考文獻:
[docker 官方文件](https://docs.docker.com/)
