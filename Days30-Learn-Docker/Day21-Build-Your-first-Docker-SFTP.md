###### tags: `鐵人賽` `docker`

# Day21 建置自己的第一個 Docker sftp

今天我們就要來建置屬於自己的第一個 Docker sftp，在建立的時候幾乎是處處碰壁，不過因此也是學習了不少知識。

## 事前準備

要建立 Docker sftp 需要一些事前準備檔案或是規劃。

- atmoz/sftp
  因為我們要建立 sftp，而 Docker hub 上就有提供現成的 sftp 映像檔(atmoz/sftp)
- Asymmetric cryptography
  雖然 sftp 可以使用帳號密碼就能登入，但是透過非對稱的認證機制更安全，因此也會實作使用公鑰及私鑰的做法
- FileZilla
  當建置好了 sftp 需要一個連線方式，可以先透過 FileZilla 這個軟體來做測試

## 開始建置

首先我們要先取得 atmoz/sftp

```bash=
docker pull atmoz/sftp
```

再來看一段官方提供的 atmoz/sftp 建立格式

```bash=
docker run \
    -v <host-dir>/upload:/home/<username>/upload \
    -p 2222:22 -d atmoz/sftp \
    <username>:pass:1001
```

> - volumes 的設定是透過共享 host 位置用來儲存檔案，這樣重建容器檔案還會在
> - sftp server 預設的 port 為 22，這邊我們公開到本機的 2222
> - `foo:pass:1001`這邊只有三個，但其實可以設定五個格式是`<user>:<pass>:<UID>:<GID>:<dir>`，最後一個是儲存檔案的子資料夾位置，但因為我們有設定 volumes 所以就不設置他，如果都沒有設置是沒辦法存取檔案在`/HOME`底下的

#### sftp server

接下來我們就可以建立基本的 sftp

```bash=
docker run \
    -v d:/share:/home/user/share \
    -p 2222:22 -d atmoz/sftp \
    user:1111:1001
```

我這邊是共享`d:/share`並將 username 設置為 user，密碼為 1111，執行完就可以看到我們的 sftp 已經建立成功。

#### 安裝 FileZilla

接下來我們要透過 FileZilla 這套軟體來連接我們建立好的 sftp。
進到[FileZilla 官網](https://filezilla-project.org/)並下載並安裝 FileZilla client
![](https://i.imgur.com/IpapfNM.png)
安裝並開啟就可以看到以下畫面
![](https://i.imgur.com/uil8PyN.png)

接下來就要連結我們的 sftp 1.點選 file 開啟下拉選單再點選 Site Manager，並點選 New site 簡單命名這個 site，再來我們要選擇 Protocol 為 SFTP，再填入 host 為你的 Domain host(如例為 localhost)以及公開的 port(如例為 2222)，Login Type 就設定 Normal 並填入 username 以及 password，再按下 Connect 就可以連接成功了。

連接成功
![](https://i.imgur.com/wH1iKeQ.png)

> 這邊要注意，如果嘗試把檔案拉至根目錄底下，根目錄是沒有存取權限的因此傳輸會失敗。

#### sftp server with Asymmetric cryptography

雖然可以設定帳號密碼來登入，但是安全性比較差，尤其是密碼有被破解的可能，因此接下來要來使用公鑰私鑰的方式來登入。

1.首先進到自己喜歡的目錄底下，用來產生並儲存公/私鑰，產生公/私鑰的方法是透過`ssh-keygen`命令，而我們是使用 ed25519 來加密。如下就會產生兩個檔案一個為`ssh_host_ed25519_key.pub`另一個則是`ssh_host_ed25519_key`

```bash=
ssh-keygen -t ed25519 -f ssh_host_ed25519_key
```

接下來我們要將我們的公鑰放置 sftp server 裡面，因此在同個目錄底下建立 Dockerfile 來配置我們的動作

```dockerfile=
FROM atmoz/sftp
RUN mkdir -p /home/user/.ssh/keys
COPY ./ssh_host_ed25519_key.pub /home/user/.ssh/keys/
```

之後建置我們的 sftp

```bash=
docker build -t first-sftp .
```

接下來我們就可以讓我們的 container run 起來

```bash=
docker run \
    -v d:/share:/home/user/share \
    -p 2223:22 \
    -d first-sftp \
    user::1001
```

> 這邊我們不設置 password 就是要讓使用者只能使用 private key 來登入

執行好就可以連接我們的 sftp 1.設置 private key，打開我們的 FileZilla 畫面，點選 Edit 打開下拉選單，點擊 Settings 再選擇 Connection 底下的 SFTP 選擇 Add key file 來添加我們剛剛建立的 private key，添加完成就可以按下 OK。

之後一樣去 Site manager 建立新的 site，配置 host、port、user 而 password 則留空再按下 connect，就可以連接了。

## 總結:

今天在建立的時候一直搞不清楚 sftp 以及 Asymmetric cryptography 如何使用，到最後才知道要將公鑰存在 sftp server

參考文獻:
[Docker hub atmoz/sftp](https://hub.docker.com/r/atmoz/sftp)
