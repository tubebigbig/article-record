###### tags: `鐵人賽` `docker`

# Day1 Initial learning Hyper-V and Learning Docker

- Hyper-V lets you run multiple operating systems as virtual machines on Windows.
- Hyper-V provides hardware virtalization,therefore,It can virtualize hard drives and network switches etc.

![](https://petri.com/wp-content/uploads/sites/3/HyperVArchticture-600x379.gif)

如圖所示，虛擬與實體交流的方式便是透過 Microsoft Virtual Machine Bus (VMBus)來執行，而 VMBus 被 Data Execution Prevention (DEP)所保護，用意是為了避免 buffer overrun attacks

參考文獻:
https://petri.com/hyper-v-hypervisor-architecture
https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/hyper-v-on-windows-server

# Docker concepts

Docker 這幾年來一直很熱門，無非是因為他的靈活、輕巧、可擴充性以及跨系統的特性。
這邊簡單介紹一下 Docker，Docker 是一個平台，使用者在使用 Docker 時需要透過 Docker image 建立 container，container 顧名思義就是用來裝東西的容器，所以可以盡情的將 application 裝入 container 裡，而許多 container 運行時則會成為所謂的 Containerized，就像是容器被裝進箱子一般。

![](https://docs.docker.com/images/Container%402x.png)
![](https://docs.docker.com/images/VM%402x.png)
Docker Containers 跟傳統 Virtual Machines 的區別

> 傳統 Virtual Machines 透過虛擬化，除了虛擬 OS 也可以模擬出 hardware，讓一台主機可以擁有多的 OS 並各具有獨自的 hard driver 以及 network switches，每個 OS 上只能執行其對應 OS 的 APP

> Docker 則沒有實現獨自 OS 而是直接實作在 Host OS 也就是實體 OS 上，並且存放 Application 的部分以 Containers 代替，Containers 是位於應用層的抽象(Abstraction)並具有互相依賴的特性，並且多個 Containers 能夠運行在同個實體上，所以 Containers 所使用的空間就比 VM 少在建置時也是比較快。

參考文獻:
https://docs.docker.com/get-started/

