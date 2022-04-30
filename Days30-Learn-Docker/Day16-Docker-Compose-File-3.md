###### tags: `鐵人賽` `docker`

# Day16 Docker Compose file 指令大集(三)

Docker Compose file 的指令學習 gogogo，再一點就要結束了！！

## 指令介紹

### restart

設置 Docker 重啟後 container 的動作。
| flag | 解說 |
| -------- | -------- |
| `"no"` | 不做任何動作(預設) |
| `always` | 出現錯誤的退出才會重啟 |
| `on-failure` | 始終重啟(如果是手動停止的話直到 Docker 重啟前都不會) |
| `unless-stopped` | 始終重啟(如果是手動停止的話直到自行重啟 container 前都不會) |

### security_opt

此設置會覆蓋所有 image 預設的 label

```yaml
security_opt:
  - label:user:USER
  - label:role:ROLE
```

### stop_grace_period

此配置為，當在發送 SIGKILL 之前，如果 container 無法處理 SIGTERM，停止該容器所指定需要等待的時間。

> 小知識:
> 在電腦內的訊號中，SIGKILL 是用來傳送給一個 process 來停止它們的訊號，此訊號無法被拒絕，因此是在無法處理 SIGTERM 等訊號之後的最終手段。

```yaml
stop_grace_period: 1s
```

### stop_signal

設置停止時發送的訊號，預設為 SIGTERM，電腦訊號如下圖有這麼多...
![](https://i.imgur.com/dXmCBJI.png)
[圖來源](https://zh.wikipedia.org/wiki/SIGKILL)

### sysctls

於 container 中設置的 kernal 參數，只能在一開始設置不能用修改的方式變動。

```yaml
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0
```

### tmpfs

可以在 container 內設置暫時的文件系統

```yaml
tmpfs:
  - /run
  - /tmp
```

### ulimits

ulimits 用來限制 process 的資源利用率，以防止失控的錯誤或安全漏洞導致整個系統癱瘓。

```yaml
ulimits:
  nproc: 65535
  nofile:
    soft: 20000
    hard: 40000
```

### volumes

掛載 volumes，Top level 掛載的 volumes 可以重複使用。
我們從之前的 docker compose 練習的檔案來看，todo-mysql-data 就是可重複使用的 volumes。

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

- external
  當設置為 true 時，volume 會建立在 compose 外，如果外部不存在 volume 則會報錯。
- labels
  為 volume 添加 label
- name
  為 volume 設置名稱

## 總結:

compose file 命令的學習就先到這邊了，明天就要開始來學習跟 kubernetes 很像的 Docker Swarm，看能不能在學習中了解如何結合兩者的優勢來使用。

參考文獻:
[docker 官方文件](https://docs.docker.com/)
