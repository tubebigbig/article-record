###### tags: `鐵人賽` `docker`

# Day12 Dockerfile 指令大集(一)

接下來兩天要來學習所有 Dockerfile 指令的解釋以及使用方法，雖然直接實做當然是最快的學習方法，但是閱讀完全部的文件再使用才是讓基底穩健的方式。

## Introduction

Dockerfile 是 Docker 在建置 image 時依照的配置檔案，檔案格式為 YAML，由於是使用 YAML 格式，所以註解是使用`#`，而 Docker 會在執行先過濾掉註解。在 Dockerfile 裡面包含了建置的設定也可以加入多個命令指令，建置 image 除了使用原生也可以使用 BuildKit，多了一些優點不過這不是今天的目標所以略過。[BuildKit 連結](https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/experimental.md)

要建立 image 需要透過 build 指令使用

```bash
docker build .
# 於當前路徑下尋找Dockerfile來建置image
```

單純使用 build 不加其他參數，會自動尋找 Dockerfile 名稱的 YAML 來執行，那當然除了叫 Dockerfile 也可以改成其他名字，只是需要加上`-f` flag 讓 Docker 找到正確的路徑，而路徑可以是本機路徑也可以是一段網址

```bash
$ docker build -f /another/way/to/build/Dockerfile .
```

使用`-t` flag 可以為建置的 image 標記名稱

```bash
$ docker build -t buildme/help .
# 如果要多個名稱就繼續加-t下去，例如-t image/one -t image/two
```

## .dockerignore

跟.gitignore 以及其他同類型的檔案一樣，如果有需要排除的檔案像是在 COPY 時不想搬過去的，可以寫在.gitignore 裡面

```
# ignore .git
.git
# ignore all markdown files (md)
*.md
```

---

## 指令介紹

### Parser directives

可以透過在檔案最上方配置 Dockerfile 的解析設置，如果要使用 Parser directives 需要遵守幾個規則，像是最後的指令後面需要空一行，重複的指令視為無效，寫在配置及命令後視為無效，寫在註解後也是無效。
Parser directives 支援`syntax`以及`escape`，詳細就看[Parser directives 說明](https://docs.docker.com/engine/reference/builder/#parser-directives)

### Environment

環境變數就像是程式的變數一樣，可以將值儲存起來使用，關鍵字是`ENV`，使用方法是用`${}`包起來

```dockerfile
FROM node:current-slim
ENV Cool /var
WORKDIR ${Cool}
COPY \$Cool .
```

VSCode 顯示了一些常用的環境變數
![](https://i.imgur.com/Kq9YdOq.png)

### FROM

從`FROM` flag 可以知道新的 build 要建置，一個 Dockerfile 必須至少要有一個 FROM 來開頭，官方文件提供的 FROM 格式有三種

```dockerfile
FROM [--platform=<platform>] <image> [AS <name>]
# or
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
# or
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

`--platform` flag 是在需要使用多平台 image 情況下可以使用來指定 image 的平台，像是 linux/amd64 or windows/amd64 這種的

如上環境變數那段使用的 FROM，代表我們要從公共庫拉出 node 且 tag 是`current-slim`的 image

```dockerfile
FROM node:current-slim
```

### ARG

剛剛雖然說 FROM 必須在最開頭，但其實 ARG 可以是個例外，為甚麼呢，因為如果在 FROM 之前設置的 ARG 是在建置之外使用的，可以想像他被特別抽出了，也代表除了 FROM 之後都不能使用。
ARG 的功用可以當成自定義帶入的參數，能夠在 build 的時候加上`--build-arg <varname>=<value>`的 flag 來帶入。

### RUN

RUN 在[Day3 開始體驗 Docker 吧(二)](https://ithelp.ithome.com.tw/articles/10238119)有簡單解說過
RUN 可以有兩種執行形式

1. `RUN <command>` 使用 command
2. `RUN ["executable", "param1", "param2"]` 使用 JSON 格式的執行表格

RUN 會在當前 image 的頂端的 new layer 執行，再來才會創建 image，通常用來安裝套件。

### CMD

Dockerfile 只會執行一個 CMD 指令，如果有多個就只會執行最後一個。
CMD 共有三種執行格式

1. `CMD ["executable","param1","param2"]` 預設的執行表格
2. `CMD ["param1","param2"]` 提供給 ENTRYPOINT 預設參數
3. `CMD command param1 param2` shell 的執行

雖然看起來 CMD 跟 RUN 看起來很像但是兩個的差距甚大，RUN 就是直接執行指令，而 CMD 則類似將指令提交給 image，所以才會將安裝套件放置 CMD

### LABEL

label 就如同名稱一樣，幫我們的 image 別上標籤，必須依照`<key>=<value>`的方法來定義標籤，有沒有發現跟 Kubernetes 一樣呢。
別上標籤的 image 目前看到是可以使用 filter 用來篩選 image，或是利用第三方套件來使用。

## 總結

透過仔細閱讀指令的方式將 Dockerfile 了解的更透徹了，明天將繼續把剩餘的指令了解完。明天還要補班的各位辛苦拉，小的也是受(害者)眾之一，各位一起努力吧。

參考文獻:
[docker 官方文件](https://docs.docker.com/)
