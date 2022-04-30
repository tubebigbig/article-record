###### tags: `鐵人賽` `docker`

# Day4 了解 Volumes 及 Bind Mounts

昨天順利地將我們的 APP 推至 image 並且學會了更新的技巧了。
但是每次更新的時候 todo-list 的資料也跟著消失不見了，該怎麼辦呢。
這時候肯定就會想到需要一個**儲存資料**的方法，那個方法就是 ── Volumes

# Volumes

Volumes 是由 container 建立，用於保存數據，就是這麼簡單，Volumes 有幾個特點。

- Volumes 易於備份跟遷移。
  > 因為是單獨的 container 想怎麼用就怎麼用
- 新的 Volumes 可以透過 container 預先填入內容。
  > 待補
- Volumes driver 可以將你的 Volumes 儲存至遠端主機及雲服務，可以加密 Volumes 的內容或是使用其他功能。
  > 待補
- Volumes 可以在多個 container 間安全地共享。
  > 待補
- Volumes 在 windows 或是 linux 都可以使用。
- 可以透過 docker CLI 或是 docker API 管理 Volumes。

## 建立 Volumes

要建立 Volumes 的話需要直接搭配 volume 指令就能建立，如下則是建立一個名為 todo-db 的 Volumes。

```shell
docker volume create todo-db
```

接下來就是要連接 APP 與 Volumes，因此記得要先停止 APP 才能成功連接 Volumes。
停止 APP 之後執行以下指令就能連接 APP 與 Volumes。

```shell
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

> 跟一般執行 APP 的指令很像，但是多了-v 的標誌代表要指定 Volumes

接下來就可以在 APP 中加入幾個 todo 事件，或是勾選完成。
之後便可以將 APP 的 image 停止並移除，重新建置再依照上面的指令執行。
執行完成後就會發現剛剛所做的操作以及資料都還留著呢，就是這麼的方便。

# Bind Mounts

Bind Mounts 跟 Volumes 不一樣的地方在於，Volumes 會在 docker 的儲存目錄中建立一個新的目錄，並且由 Docker 管理使用；而 Bind Mounts 則是將主機(host)的目錄安裝置 container 中，所以目錄是由主機上的固定的路徑來使用，雖然 Bind Mounts 效能比較好，但是需要在特定目錄結構的主機文件系統中才能使用且無法使用 Docker CLI 來進行管理。

在開始使用 Bind Mounts 之前需要建立開發模式的 container。

```shell
docker run -dp 3000:3000 \
    -w /app -v "$(pwd):/app" \
    node:12-alpine \
    sh -c "yarn install && yarn run dev"
```

> -w /app 代表設置命令的工作目錄或是當前目錄
> -v "$(pwd):/app" 代表將當前目錄綁定至 container 中的/app 目錄中
> 剩下的就跟 Dockerfile 中的一樣

接下來就可以執行 logs 命令查看檔案變化。

```shell
docker logs -f <container-id>
```

接下來將檔案修改並儲存，例如修改 src/static/js/app.js 中 109 的

```javascript
-                         {submitting ? 'Adding...' : 'Add Item'}
+                         {submitting ? 'Adding...' : 'Add'}
```

儲存之後除了在 logs 印出的資訊裡也可以在 local:3000 的頁面看到變化，對於本地開發測試來說非常方便且實用，不需要安裝所有構建工具和環境。

明天就接著學習如何使用 Multi-Container Apps。

[Docker 官方文件](https://docs.docker.com/)
