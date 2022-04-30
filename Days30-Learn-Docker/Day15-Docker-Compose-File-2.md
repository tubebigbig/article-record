###### tags: `鐵人賽` `docker`

# Day15 Docker Compose file 指令大集(二)

今天繼續 Docker Compose file 的指令學習，由於 docker stack 比較特別可能單獨一篇主要學習他，因此如果有關於 docker stack 的命令我都先跳過。

## 指令介紹

### command

command 指令可以覆蓋默認的執行指令

### credential_specd

可以設置帳戶的憑證的位置，僅能在 window container 使用。

### depends_on

可以配置服務的依賴項，啟動包含依賴的服務時就會先啟動依賴服務，不過啟動依賴不會等待類似 DB、Redis、solr 或是 Elasticsearch 的狀態就緒，如果需要此功能就需要透過三方工具或是寫腳本做連接測試。

### devices

將本機的位置添加到 container

```yaml
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```

### dns

docker compose file 也可以設定自己的 DNS Server

```yaml
dns:
  - 8.8.8.8
  - 9.9.9.9
```

### dns_search

設置 DNS 的搜尋域名

```yaml
dns_search:
  - dc1.example.com
  - dc2.example.com
```

### env_file

包含環境變數的檔案，可以帶入環境變數...

```yaml
services:
  some-service:
    env_file:
      - a.env
      - b.env
```

### environment

在 compose file 中添增環境變數

```yaml
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

### expose

當不建置在主機時，用來設定要公開的端口

```yaml
expose:
  - "3000"
  - "8000"
```

### extra_hosts

能夠用來增加主機名，增加的主機名會儲存在`/etc/hosts`內

```yaml
extra_hosts:
  - "somehost:162.242.195.82"
  - "otherhost:50.31.209.229"
```

### healthcheck

同[Day13 Dockerfile 指令大集(二)](https://ithelp.ithome.com.tw/articles/10244821)提到的 Dockerfile 的健康檢查

### image

標出需要使用的 image 名稱，如果 image 不存在，compose 會幫你建置。

### init

可以在 container 內執行初始化程序(還不知道能幹嘛...

### isolation

主要是 window 在使用，可以配置`default`, `process` and `hyperv`

- process 是共享 kernel 的隔離模式，多個 container 能夠在同個主機上運行。
  ![](https://i.imgur.com/uXI0iy5.png)
  [圖來源](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container)
- hyperv 則是將各個 container 運行在 VM 內，因此都有自己的 kernel，具有較強的安全性以及硬體區別。
  ![](https://i.imgur.com/CnGQ3T5.png)
  [圖來源](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container)

### labels

將配置的值添加至 container

### logging

服務 log 紀錄的路徑配置，可以設置 driver 以及 options。
driver 可以配置`"json-file"`(預設)、`"syslog"`以及`"none"`
options 可以配置 address 以及檔案大小還有數量

```yaml
driver: "syslog"
options:
  syslog-address: "tcp://192.168.0.42:123"
  max-size: "200k"
  max-file: "10"
```

### network_mode

配置網路的模式，包含`"bridge"`、`"host"`、`"none"`、`"service:[service name]"`、`"container:[container name/id]"`

### networks

配置要使用的網路，除此之外也可以設置別名(ALIASES)還有 IPV4 以及 IPV6 的設定

### ports

服務要公開的 port 號
簡單表示方法

```yaml
ports:
  - 3000:3000
```

詳細的表示方法

```yaml
ports:
  - target: 3000
    published: 3000
    protocol: tcp
    mode: host
```

## 總結:

今天超過了昨天訂下的進度，明天就一鼓作氣將全部的 compose file 命令了解完，再來就要來學 docker swarm 以及 docker stack。

參考文獻:
[docker 官方文件](https://docs.docker.com/)
