###### tags: `鐵人賽` `docker`

# Day24 將 private key 安全地放入容器內

今天來完成昨天未完成的在 ubuntu 中私鑰的使用問題。
今天又回去看了一遍之前的文章，以及翻閱了官網的 Document，看到了 secret 的用法，剛好就適合處理 private key，當初真該早點意識到的。

1.在 compose-file 的檔案中加入 secrets 的建立配置

```yaml=
secrets:
  ubuntu_ed_key:
    file: ./secret/ssh_host_ed25519_key
```

2.再來將要使用 secret 的 APP 標示出來，如我們昨天的例子就是用 interfact

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
      - ./secret/ssh_host_server_ed25519_key:/etc/ssh/ssh_host_ed25519_key
      - d:/share:/home/user/share
    command: user:1111:1001
  interfact:
    image: interfact:1.0
    build:
      context: ./interfact/
      dockerfile: Dockerfile
    ports:
      - 2223:80
    tty: true
    secrets:
      - source: ubuntu_ed_key
        mode: 0400         # mode 設成400只有自己可讀，這樣sftp才不會說你太開放
secrets:
  ubuntu_ed_key:
    file: ./secret/ssh_host_ed25519_key
```

3.使用 stack 執行起來我們的服務，可以看到多建立了 secret 的項目

```bash=
$ docker stack deploy -c docker-compose.yaml sftp-app
Ignoring unsupported options: build

Creating network sftp-app_default
Creating secret sftp-app_ubuntu_ed_key
Creating service sftp-app_interfact
Creating service sftp-app_first-sftp
```

4.查看 secret 檔案。在我們的服務執行起來之後，可以進入我們的 container 查看 secret 檔案，secret 檔案會放在`/run/secrets/`目錄底下。

```bash=
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS
            NAMES
6ac3d6f74ba8        first-sftp:1.0      "/entrypoint user:11…"   27 seconds ago      Up 23 seconds       22/tcp
            sftp-app_first-sftp.1.wes9z5s6ocgs40qoj6g7j9duo
e972c598a9fa        interfact:1.0       "/bin/bash"              30 seconds ago      Up 27 seconds
            sftp-app_interfact.1.0c9ajol7ms0lzir8egq9edxo1
$ docker exec -it e972c598a9fa bash
root@e972c598a9fa:/# cd run/secrets/
root@e972c598a9fa:/run/secrets# ls
ubuntu_ed_key
# 可以看到我們剛剛建立名為ubuntu_ed_key的secret檔案
```

5.尋找 sftp server 的位址

```bash=
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
678l08i7hqmn        sftp-app_default    overlay             swarm
$ docker inspect 678l08i7hqmn
# 找到first-sftp的位址
```

6.接下來就可以使用 sftp 指令連接我們建置的 sftp server，第一次連接都會要我們認證 sftp，之後就不用了。

```bash=
root@d0dfaf2e1e6d:/# sftp -i run/secrets/ubuntu_ed_key user@10.0.2.3
The authenticity of host '10.0.2.3 (10.0.2.3)' can't be established.
RSA key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.2.3' (RSA) to the list of known hosts.
Connected to 10.0.2.3.
sftp>
```

看到`sftp>`就是連接成功了
