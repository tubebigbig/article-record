###### tags: `鐵人賽` `docker`

# Day3 開始體驗 Docker 吧(二)

昨天成功建立了第一個 container image，今天就來接續著建立出來的教學文件實作吧。

## 建立自己的 image

首先先下載位於教學文件 Our Application 頁上的 todo-list 範例。
![](https://i.imgur.com/S6SykKp.png)

### Dockerfile

接下來就是要撰寫運行 Docker 的靈魂人物 ── Dockerfile
依照官方提供的寫法如下

```dockerfile
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

> - FROM 表示執行的環境，還可以設定 ubuntu。
> - WORKDIR 為 image 設置的工作目錄。
> - COPY
>   格式: COPY <host 路徑> <目標路徑>
>   表示從 host 複製檔案至 container 的系統下。
> - RUN 會在當前 image 的頂端的 new layer 執行，再來才會創建 image，通常用來安裝套件，如上則是代表要使用 yarn 下載 package.json 對映的套件。
> - CMD CMD 的主要目的是為執行中的 container 提供默認指令。在 dockerfile 中只會生效最後一個 CMD 指令。如上則是代表要使用 node.js 執行 src/index.js。

### 建置並運行

如果只是要建置 APP 玩玩，基本上撰寫完 dockerfile 就完成大部分拉。
接下來便可以執行以下兩個命令建置 container image 並執行

```shell
docker build -t getting-started .
docker run -dp 3000:3000 getting-started
```

> -t 代表為建立的 image 命名 getting-started 的 tag，如果不使用就會自動產生隨機值。

在指令跑完之後就能打開 http://localhost:3000 查看建立的 APP 拉。

### 更新 image

接下來可能會想問說，如果修改後的程式該怎麼更新至 image 呢?
是直接執行建置跟運行的指令就可以了嗎? 馬上來試試看吧
執行之後卻發現印出了錯誤！是怎麼回事呢?

```
docker.exe: Error response from daemon: driver r failed programming external connectivity on endpoint gracious_johnson (db545d677ed4b1e336a876ae9c905833bbb4b201cfa09a70314d4e3c66d9396435): Bind for 0.0.0.0:3000 failed: port is aly ready allocated.
```

> 總之呢就是，因為要運行的位置已經有 image 了所以無法運行。
> Docker 不會幫你刪掉舊有的 image (Docker: 自己去刪拉

### 停止/刪除 image

因此我們馬上來學習如何停止並刪除現有的 image。
要停止/刪除現有得 image 需要先執行以下命令，用來查看 image 的 container-id。

```shell
docker ps --all
```

> --all 代表會列出所有啟用及停用的 image

沒意外的話會類似於下方顯示的結果，找到剛剛建置得 getting-start 的 container-id
![](https://i.imgur.com/RketCmd.png)

再來就是停止並移除 image

```shell
docker stop <container-id>
docker rm <container-id>
```

移除完 image 之後再執行

```shell
docker run -dp 3000:3000 getting-started
```

修改過的程式就運行在 contrainer 上了。

明天就學習如何將 image 使用的數據永久的保存吧，也就是 Container Volumes 並搭配 Bind Mounts。
