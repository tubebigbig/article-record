###### tags: `鐵人賽` `docker`

# Day23 連接 ubuntu 與 sftp 的橋樑

今天要來嘗試透過網路來當作橋梁，讓 ubuntu 能夠訪問 sftp server。

> 也有透過將 network 設成 host 再將 port export 出來，然後在 host 做之間的連接，不過看起來麻煩多了，而且也不需要將 sftp 公開到 host，因此還是用 overlay 就好。

首先是要先建置 ubuntu，並且在其中安裝好 openssh-client。
在我們昨天建立好的專案建立 interfact 資料夾，用來擺放 ubuntu 的 Dockerfile

ubuntu 的 Dockerfile

```dockerfile=
FROM ubuntu:latest
RUN apt-get update && apt-get install -y openssh-client
```

建置 interfact

```bash=
docker build -t interfact:1.0 ./interfact/
```

接下來修改一下 compose-file 將 interfact 以及 sftp 放在 compose file 底下，使用 Swarm 啟動服務時就會自動建立之間的網路。

```yaml=
version: "3.8"
services:
  first-sftp:
    image: first-sftp:1.0
    build:
      context: ./sftp/
      dockerfile: Dockerfile
    volumes:
      - ./secret/ssh_host_server_ed25519_key:/etc/ssh/ssh_host_ed25519_key
      - d:/share:/home/user/share
    command: user:1111:1001
    # 因為使用密鑰做連結時，他說密鑰too open，還沒辦法解決就先用密碼來連接
  interfact:
    image: interfact:1.0
    build:
      context: ./interfact/
      dockerfile: Dockerfile
    tty: true
```

接下來使用 stack deploy 服務

```bash=
$ docker stack deploy -c docker-compose.yaml sftp-app
Ignoring unsupported options: build

Creating network sftp-app_default
Creating service sftp-app_interfact
Creating service sftp-app_first-sftp
```

接下來我們要尋找 sftp 的位址才能訪問它。我們在啟動服務時就有自動建立網路`sftp-app_default`

```bash=
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
x8vyt8z9atok        sftp-app_default    overlay             swarm
$ docker inspect x8vyt8z9atok
[
    {
        "Name": "sftp-app_default",
        ...
        "Containers": {
            "6f36d50f02ac35c5a00f0c6c3fdb563838e314be6af5768732c1d8584fdfa38e": {
                "Name": "sftp-app_interfact.1.xhqv0t6wmh0ec8bx1dvj8rxva",
                "EndpointID": "e58344715ce3e8e03117fc390135f872f794e04061c2c4f6b5c309935ea583d6",
                "MacAddress": "02:42:0a:00:06:03",
                "IPv4Address": "10.0.6.3/24",
                "IPv6Address": ""
            },
            "b60c57592989e5135489a0a5d4294ead1522a68b8ab40d7058203f965f233b4d": {
                "Name": "sftp-app_first-sftp.1.kjvc21vjr1ttt9lup53dx7du9",
                "EndpointID": "0c77844764b723813ab9dd043816f7da651c9eaf445225b5b9fa2bec8c10b41b",
                "MacAddress": "02:42:0a:00:06:06",
                "IPv4Address": "10.0.6.6/24",
                "IPv6Address": ""
            },
            "lb-sftp-app_default": {
                "Name": "sftp-app_default-endpoint",
                "EndpointID": "f89f87b635ba197de1a7f2fed64fb9eb3ea99cb6a61596e6c1cc3a3915d241dc",
                "MacAddress": "02:42:0a:00:06:04",
                "IPv4Address": "10.0.6.4/24",
                "IPv6Address": ""
            }
        },
        ...
    }
]
```

查看我們 inspect 網路之後的輸出，可以在 Containers 中找到 Name 為 sftp-app_first-sftp 的 container，其中的 IPv4Address 就是 sftp 在`sftp-app_default`這個網路中的位址。

> 可以看到 IPv4Address 為 10.X.X.X 代表其為私有網路

有了 sftp 的位址之後我們就可以使用 exec 連接到 interfact 來做 sftp 訪問

```bash=
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS
NAMES
b60c57592989        first-sftp:1.0      "/entrypoint user:11…"   17 seconds ago      Up 13 seconds       22/tcp
sftp-app_first-sftp.1.kjvc21vjr1ttt9lup53dx7du9
6f36d50f02ac        interfact:1.0       "/bin/bash"              21 seconds ago      Up 18 seconds
sftp-app_interfact.1.xhqv0t6wmh0ec8bx1dvj8rxva

$ docker exec -it 6f36d50f02ac bash
root@6f36d50f02ac:/# sftp user@10.0.6.6:22
user@10.0.6.6's password:
Connected to 10.0.6.6.
sftp>
```

看到`sftp>`就代表連接成功了

## 總結:

接下來我就要試著在 ubuntu 容器內創建金鑰加密，並將 public key 加入 first-sftp 的 container，這些就會在明天的文章內了，那如果這是不可行的就會再想其他方法...
