---
title: Docker容器数据卷
date: 2019-09-18 22:54:54 
tags: 
    - Docker
---

### 是什么 ###

一句话：有点类似我们Redis里面的rdb和aof文件
 
先来看看Docker的理念：

- 将运用与运行的环境打包形成容器运行 ，运行可以伴随着容器，但是我们对数据的要求希望是持久化的

- 容器之间希望有可能共享数据
 
 
Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据做为镜像的一部分保存下来，
那么当容器删除后，数据自然也就没有了。
 
为了能保存数据在docker中我们使用数据卷。

<!--more-->>>
### 能干嘛 ###
 
 容器的持久化
 容器间继承+共享数据
 
 
 数据卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：数据卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷
 
特点：

- 数据卷可在容器之间共享或重用数据
- 卷中的更改可以直接生效
- 数据卷中的更改不会包含在镜像的更新中
- 数据卷的生命周期一直持续到没有容器使用它为止


### 数据卷 ###

容器内添加

#### 一.直接命令添加 ####
  
1.命令   docker run -it -v /宿主机绝对路径目录:/容器内目录      镜像名

      docker run -it -v /宿主机目录:/容器内目录 centos /bin/bash

2.查看数据卷是否挂载成功

docker inspect 容器ID



![](/image/Docker/docker-inspect.png)

3.容器和宿主机之间数据共享

4.容器停止退出后，主机修改后数据是否同步-->同步

5.命令(带权限)   docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名

#### 二.DockerFile添加 ####

1.根目录下新建mydocker文件夹并进入

2.可在Dockerfile中使用VOLUME指令来给镜像添加一个或多个数据卷

     
VOLUME["/dataVolumeContainer","/dataVolumeContainer2","/dataVolumeContainer3"]
 
说明：
 
出于可移植和分享的考虑，用-v 主机目录:容器目录这种方法不能够直接在Dockerfile中实现。
由于宿主机目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都存在这样的特定目录。


3.File构建

# volume test
FROM centos
VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
CMD echo "finished,--------success1"
CMD /bin/bash


4.build后生成镜像  
 
docker build -f /mydocker/dockerFile2 -t sugelamei/centos

获得一个新镜像sugelamei/centos

5.run容器

6.通过上述步骤，容器内的卷目录地址已经知道对应的主机目录地址哪？？

![](/image/Docker/docker-volume.png)

7.主机对应默认地址

![](/image/Docker/host-volume.png)

注意：Docker挂载主机目录Docker访问出现cannot open directory .: Permission denied
解决办法：在挂载目录后多加一个--privileged=true参数即可

### 数据卷容器 ###

#### 是什么 ####

 
命名的容器挂载数据卷，其它容器通过挂载这个(父容器)实现数据共享，挂载数据卷的容器，称之为数据卷容器

#### 总体介绍 ####

以上一步新建的镜像sugelamei/centos为模板并运行容器dc01/dc02/dc03

它们已经具有容器卷    /dataVolumeContainer1  /dataVolumeContainer2


#### 容器间传递共享(--volumes-from) ####

1.先启动一个父容器dc01  在dataVolumeContainer2新增内容

    docker run -it --name dc01 sugelamei/centos
    touch dc01_add.txt

2.dc02/dc03继承自dc01   --volumes-from

    docker run -it --name dc02 --volumes-from dc01 sugelmei/centos
	touch dc02_add.txt
    touch dc03_add.txt

3.dc02/dc03分别在dataVolumeContainer2各自新增内容

4.回到dc01可以看到02/03各自添加的都能共享了
![](/image/Docker/dc0123.png)

5.删除dc01，dc02修改后dc03可否访问

6.删除dc02后dc03可否访问

7.新建dc04继承dc03后再删除dc03


结论：容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止
   数据卷可以被继承，具有继承关系之间的数据卷共享数据，一个改变，整个都改变，即：一个改变大家都变










 





