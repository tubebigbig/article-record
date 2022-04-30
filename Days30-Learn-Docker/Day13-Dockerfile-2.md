###### tags: `鐵人賽` `docker`

# Day13 Dockerfile 指令大集(二)

今天頭很痛可能會寫得有點亂，不過還是讓我們繼續 Dockerfile 的指令學習吧。

## 指令介紹

### EXPOSE

expose 有公開的意思，因此其功能類似於使用 run 指令時下的`-p` flag，可以指定的網絡端口(端口有 TCP 以及 UDP)，不過寫在 expose 的 port 不會真的被公開，只是當作一個紀錄而已。

```dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
# 也可以同時設定兩個，那就會對TCP以及UDP各別公開
```

### ADD / COPY

ADD 跟 COPY 就一起介紹吧，因為這兩個功能大同小異，都是能夠將檔案或是路徑從指定路徑`<src>`添加至指定的 Docker 的系統文件中`<dest>`，格式如下

```dockerfile
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
# 要使用ADD就將COPY改成ADD就好
```

剛剛提到 ADD 跟 COPY 之間的小異就是指，ADD 能夠接收的格式有多兩種使用 remote URL 以及 tar 提取，而一般本地端檔案的複製使用還是以 COPY 為主，而 remote URL 其實可以使用 RUN 命令在搭配 curl 來替代效率會更好，因此 ADD 最好的用法就是將本地的 tar 檔案提取到 container

### VOLUME

VOLUME 就是之前提到用來儲存資料的東西，而這邊的指令則是在指定 container 要使用的 VOLUME
格式:

```dockerfile
VOLUME ["/data"]
```

也可以帶入多個參數

```dockerfile
VOLUME ["/var/log/"]
# 如上則會對應VOLUME /var/log 或是 VOLUME /var/log /var/db
```

### WORKDIR

workdir 作為工作目錄，提供給`RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD`指令來使用，沒有建立的話 Docker 會自動建立。
workdir 可以多行使用，當前的 workdir 會相對於前面的 workdir，如例:

```dockerfile
WORKDIR /first
WORKDIR second
WORKDIR third
RUN path
# path 執行的workdir則是 /first/second/third
```

### ONBUILD

ONBUILD 是一個強大的功能，類似於繼承的概念，當有新的 image 需要以此建置當基礎時就可以使用到 ONBUILD。當建立的時候會將 ONBUILD 的命令儲存起來，因此當前指令不會生效。之後當我們使用 FROM 將此 image 作為基礎來建立新的 image 時，builder 就會尋找 ONBUILD 並執行，當 ONBUILD 執行失敗那建置也會中止。
格式:

```dockerfile
ONBUILD <INSTRUCTION>
```

示例

```dockerfile
ONBUILD COPY . /src
ONBUILD RUN APP --dir /src
# 繼承的image就會先將上層的image依照此指令執行。
```

### HEALTHCHECK

HEALTHCHECK 指令用來讓 Docker 了解如何檢查 image 是否健康，透過 HEALTHCHECK 在啟動的時候狀態就會顯示 health 如果一定次數的啟動失敗就會變得 unhealthy
HEALTHCHECK 可以配置四種設定

- `--interval=DURATION` (default: 30s) 檢查的間隔
- `--timeout=DURATION` (default: 30s) 檢查運行的最大時間上限
- `--start-period=DURATION` (default: 0s) 開始時間 提供給需要時間進行引導的容器
- `--retries=N` (default: 3) 重試次數

### SHELL

可以改變 Shell，以 window 來說可以更換 powershell、cmd 以及 sh，更換後的指令就會變為運行在對應的 shell

## 總結:

寫著寫著頭就不痛了，難道這就是知識的力量嗎。
既然已經學完了所有 Dockerfile 指令，接下來就要來練習寫 Dockerfile 了

參考文獻:
[docker 官方文件](https://docs.docker.com/)
