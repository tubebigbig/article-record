###### tags: `鐵人賽` `docker`

# Day22 sftp APP 建置專案架構

今天來建立檔案傳輸 APP 的架構規劃，以及將 container 的建置設定加入 compose-file，預計未來會有 API、DB、sftp、前端頁面這四個容器。

## 專案架構

我們要來規劃前後端分開的頁面，再包含 DB 以及 sftp，因此總共會分成四個資料夾

```
.
├─Backend                       // 後端程式及配置
├─Database                      // DB配置
├─Frontend                      // 前端程式及配置
├─secret                        // 用來儲存private key，不會儲存在git上
│─sftp                          // sftp配置以及public key
│   │─bash
│   │─Dockerfile
│   │─ssh_host_ed25519_key
│   └─ssh_host_ed25519_key.pub
│─docker-compose.yaml
└─readme.md
```

## Deploy my APP with stack

建立好之後我們開始撰寫`docker-compose.yaml`，之後 sftp 的 port 就不會公開到 host，而是唯一公開給 back-end 使用。

```yaml=
version: "3.8"
services:
  first-sftp:
    image: first-sftp:1.0
    build:
      context: ./sftp/
      dockerfile: Dockerfile
    ports:
      - 2222:22
    volumes:
      - secret/ssh_host_server_ed25519_key:/etc/ssh/ssh_host_ed25519_key
      - d:/share:/home/user/share
    command: user::1001
```

建置好 compose file 之後就先來建置 image，先移至 project 的根目錄再執行 compose build

```bash=
$ docker-compose build
Building first-sftp
Step 1/3 : FROM atmoz/sftp
 ---> bd51ae4c2150
Step 2/3 : RUN mkdir -p /home/user/.ssh/keys
 ---> Using cache
 ---> 78a42e3ffc67
Step 3/3 : COPY ssh_host_ed25519_key.pub /home/user/.ssh/keys/
 ---> bf2425999fc7

Successfully built bf2425999fc7
Successfully tagged first-sftp:1.0
```

接下來就要使用 stack deploy 我們的 service

```bash=
docker stack deploy -c docker-compose.yaml sftp-app
```

deploy 好我們的 service 之後就來查看他們

```bash=
$ docker service ls
ID                  NAME                  MODE                REPLICAS            IMAGE               PORTS
k3djoiknwsnc        sftp-app_first-sftp   replicated          10/10               first-sftp:1.0   *:2222->22/tcp
```

最後就是再使用上我們的擴展功能

```bash=
$ docker service scale k3djoiknwsnc=10
```

## update stack

擴展完之後可以再將我們 compose file 中的 image 版本移除(也就是變成預設的 latest)再來更新 stack

```
$ docker-compose build
$ docker service update --image first-sftp sftp-app_first-sftp
```

之後就可以用 service ps 來查看更新的狀態，會有 shutdown 的狀態是留下來的 log

```bash=
$ docker service ps sftp-app_first-sftp
ID                  NAME                         IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
ani6a9nbkl4c        sftp-app_first-sftp.1        first-sftp:latest   docker-desktop      Running             Running about an hour ago
y633vwvgmqw2         \_ sftp-app_first-sftp.1    first-sftp:1.0      docker-desktop      Shutdown            Shutdown about an hour ago
8ymkmve7ad0n        sftp-app_first-sftp.2        first-sftp:latest   docker-desktop      Running             Running about an hour ago
ibg3f1uu83oj         \_ sftp-app_first-sftp.2    first-sftp:1.0      docker-desktop      Shutdown            Shutdown about an hour ago
rew81kvyexvl        sftp-app_first-sftp.3        first-sftp:latest   docker-desktop      Running             Running about an hour ago
ow966p25qm8g         \_ sftp-app_first-sftp.3    first-sftp:1.0      docker-desktop      Shutdown            Shutdown about an hour ago
fxywdtzw8r3w        sftp-app_first-sftp.4        first-sftp:latest   docker-desktop      Running             Running about an hour ago
tt5sufe48g7z         \_ sftp-app_first-sftp.4    first-sftp:1.0      docker-desktop      Shutdown            Shutdown about an hour ago
7vga0g0xibr3        sftp-app_first-sftp.5        first-sftp:latest   docker-desktop      Running             Running about an hour ago
dbgpcoqvjkxv         \_ sftp-app_first-sftp.5    first-sftp:1.0      docker-desktop      Shutdown            Shutdown about an hour ago
h1drblomg97s        sftp-app_first-sftp.6        first-sftp:latest   docker-desktop      Running             Running about an hour ago
mqqp9ns8ajr1         \_ sftp-app_first-sftp.6    first-sftp:1.0      docker-desktop      Shutdown            Shutdown about an hour ago
lcy1vukqtbev        sftp-app_first-sftp.7        first-sftp:latest   docker-desktop      Running             Running about an hour ago
xw96gzs3wdl8         \_ sftp-app_first-sftp.7    first-sftp:1.0      docker-desktop      Shutdown            Shutdown about an hour ago
tgoui45wxgtq        sftp-app_first-sftp.8        first-sftp:latest   docker-desktop      Running             Running about an hour ago
qzz6b5q1ypxa         \_ sftp-app_first-sftp.8    first-sftp:1.0      docker-desktop      Shutdown            Shutdown about an hour ago
ks8994133h0q        sftp-app_first-sftp.9        first-sftp:latest   docker-desktop      Running             Running about an hour ago
6ra0mehea2as         \_ sftp-app_first-sftp.9    first-sftp:1.0      docker-desktop      Shutdown            Shutdown about an hour ago
lwqg3v1fdhi5        sftp-app_first-sftp.10       first-sftp:latest   docker-desktop      Running             Running about an hour ago
qr7qb6lbs06u         \_ sftp-app_first-sftp.10   first-sftp:1.0      docker-desktop      Shutdown            Shutdown about an hour ago
```

參考文獻:
這次個人鐵人賽的筆記
