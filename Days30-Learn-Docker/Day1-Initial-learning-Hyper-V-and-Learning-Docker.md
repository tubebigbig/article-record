###### tags: `鐵人賽` `docker`
# Day1 Initial learning Hyper-V and Learning Docker

- Hyper-V lets you run multiple operating systems as virtual machines on Windows. 
- Hyper-V provides hardware virtalization,therefore,It can virtualize hard drives and network switches etc.

![](https://petri.com/wp-content/uploads/sites/3/HyperVArchticture-600x379.gif)

如圖所示，虛擬與實體交流的方式便是透過Microsoft Virtual Machine Bus (VMBus)來執行，而VMBus被Data Execution Prevention (DEP)所保護，用意是為了避免buffer overrun attacks

參考文獻:
https://petri.com/hyper-v-hypervisor-architecture
https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/hyper-v-on-windows-server

# Docker concepts
Docker這幾年來一直很熱門，無非是因為他的靈活、輕巧、可擴充性以及跨系統的特性。
    這邊簡單介紹一下Docker，Docker是一個平台，使用者在使用Docker時需要透過Docker image建立container，container顧名思義就是用來裝東西的容器，所以可以盡情的將application裝入container裡，而許多container運行時則會成為所謂的Containerized，就像是容器被裝進箱子一般。
    
![](https://docs.docker.com/images/Container%402x.png)
![](https://docs.docker.com/images/VM%402x.png)
    Docker Containers跟傳統Virtual Machines的區別
    
> 傳統Virtual Machines透過虛擬化，除了虛擬OS也可以模擬出hardware，讓一台主機可以擁有多的OS並各具有獨自的hard driver以及network switches，每個OS上只能執行其對應OS的APP

> Docker則沒有實現獨自OS而是直接實作在Host OS也就是實體OS上，並且存放Application的部分以Containers代替，Containers是位於應用層的抽象(Abstraction)並具有互相依賴的特性，並且多個Containers能夠運行在同個實體上，所以Containers所使用的空間就比VM少在建置時也是比較快。

參考文獻:
https://docs.docker.com/get-started/