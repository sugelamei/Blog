---
title: Docker的底层原理
date: 2019-09-18 21:48:51  
tags: 
    - Docker
---

### Docker是怎么工作的 ###

Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上， 
然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。 
容器，是一个运行时环境，就是我们前面说到的集装箱。

![](/image/Docker/docker_container.png)

### 为什么Docker比较比VM快 ###

<!--more-->>>
(1)Docker有着比虚拟机更少的抽象层。由于docker不需要Hypervisor实现硬件资源虚拟化,
运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。
因此在CPU、内存利用率上docker将会在效率上有明显优势。
 
(2)Docker利用的是宿主机的内核,而不需要Guest OS。因此,当新建一个容器时,
docker不需要和虚拟机一样重新加载一个操作系统内核。
仍而避免引寻、加载操作系统内核返个比较费时费资源的过程,
当新建一个虚拟机时,虚拟机软件需要加载Guest OS,
返个新建过程是分钟级别的。
而docker由于直接利用宿主机的操作系统,
则省略了返个过程,因此新建一个docker容器只需要几秒钟。

![](/image/Docker/docker_vm.png)
![](/image/Docker/docker_vm01.png)



 
 
 
 
