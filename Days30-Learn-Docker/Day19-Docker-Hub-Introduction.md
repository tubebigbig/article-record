###### tags: `鐵人賽` `docker`

# Day19 Docker Hub 學習與實作

今天來了解並實用 Docker hub。Docker Hub 可以說是 Docker 界的 github，能夠輕鬆分享或是取得他人分享的 container image，Dockerfile 內的 FROM 取用的 image 就是來自此。Docker Hub 還可以做為團隊合作的用途，可以選擇公開者以及許多有用的功能。

#### Create Repository

來試著將 bulletin 放上 Docker hub 吧。 1.要使用 Docker hub 第一步當然要先註冊帳號拉，快去辦一個吧[Docker Hub 連結](https://hub.docker.com)。 2.註冊且登入帳號後找到首頁的 Create Repository 點擊來產生一個新的庫，接下來就是填寫這個庫的名稱以及公開不公開的設定，就可以 create 了。

> 在建立的時候右邊還有提示我們怎麼 Push 我們的 APP 到 Repository 呢。
>
> 3.建立好庫之後，該怎麼把我們的 APP Push 上去呢，跟著右側的提示做吧，透過 tag 以及 push 就可以了

```bash
$ docker tag local-image:tagname new-repo:tagname
$ docker push new-repo:tagname
```

那我們要推的是 bulletin 的 APP 所以就先改成

```bash
$ docker tag bulletinboard:1.0 tubebigbig/bulletinboard:1.0
$ docker push tubebigbig/bulletinboard:1.0
```

怎麼無法執行呢，該不會哪裡搞錯了吧？其實是因為還沒登入拉
可以透過 Docker desktop 的 dashboard 來登入，也可以透過指令`docker login`來登入。
登入完再執行一次，就可以摟!!

這時候再來看我們剛剛建立的 Docker hub Repository 是不是多了 1.0 的 tag 了呢

之後要使用我們的 APP 只要透過`docker pull`就可以直接從 Docker hub 拉下來了，整個就方便了許多呢。

#### update Docker hub Repository

接下來肯定會想知道，如果要更新的時候怎麼辦呢，是不是每次都要設定 tag 再 push 上去呢？這樣的確是可以，但是一般都會有 git 來做版控，等於每次都要跟 git 分開推實在是很麻煩呢。
別擔心，Docker hub 跟 github 還有 bitBucket 早就做好連接摟，只要配置好設定跟連結，那我們每次在 push github 或是 bitBucket 的 commit 時就可以 auto build 了。

那該怎麼設置 auto build 呢？

其實在我們 Create Repository 時的最下面就可以設置了呢，那已經建立的 Repository 則是要進入我們的 Repository 再點擊叫做 builds 的 tab 就可以看到了，第一次使用的話就需要我們先授權 github 或是 bitBucket 帳號。

授權完 github 或是 bitBucket 帳號之後，就可以來建立我們的 autobuild 功能了，再回到剛剛的 builds tab 並選擇要建立 autobuilds 的平台，接下來就可以配置包誇 APP 在平台內的 Repository 位置，以及自動測試、建置規則、環境參數等。

來說明建置規則的部分，我們可以配置 auto builds 的規則，如下圖由左至右則是

- Source Type
  選擇要用 Branch 或是 Tag
- Source
  如果 Source Type 是 Branch 則代表 Branch 的分支名，如果是 Tag 則可以用正規表達式
- Docker Tag
  設置在 Docker hub 中要建置的 tag 名稱，如果 Source 是用正則可以用 sourceref 標示其位置`release-{sourceref}`
- Dockerfile location
  Dockerfile 的名稱
- Build Context
  預設是`/`也就是根目錄，可以自行設定到對應 Dockerfile 的路徑
- Autobuild
  是否要 auto build
- Build Caching
  建置緩存，可以減少建置的時間

![](https://i.imgur.com/fSuscl9.png)

設定好之後就可以點選 save，或是 save and build 直接建立。
建立好之後就會看到觀察 auto build 的儀表板，具有 Build Activity(建立的歷程圖)、Automated Builds(建立的當前狀態，可以在此使用 trigger 手動建立)以及 Recent Builds(最近的建置用條列式來表示)。

接著我們就可以試著去修改我們的 APP 並且提交 commit 點到 github 或是 bitBucket，再回來看我們的 builds 頁面，可以看到偵測到提交後正在自動建置。

![](https://i.imgur.com/KkUGUpl.png)

成功後
![](https://i.imgur.com/EBprLU3.png)

這時候再去 tag 頁面就可以看到我們更新成功的 image。
之後每次只要提交寫好的 code 到版控平台就可以自動同步建置到 Docker hub 了。

#### 其他狀態

下圖具有，in progress、success、cancel、fail 等狀態

![](https://i.imgur.com/c2BfoUZ.png)

Build Activity 的畫面

![](https://i.imgur.com/6cCY5ky.png)

參考文獻:
[docker 官方文件](https://docs.docker.com/)
