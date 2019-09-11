---
title: Docker常用命令
date: 2019-08-09 11:05:01
tags: 
    - Docker
---

### 帮助命令  ###

 docker version

 docker info	

 docker --help

<!--more-->

### 镜像命令 ###

  1.查看本地镜像

  docker images [OPTIONS] [REPOSITORY[:TAG]]
  
  Options说明:

			-a :列出本地所有的镜像（含中间映像层）

      		-q :只显示镜像ID。
 			
			--digests :显示镜像的摘要信息

			--no-trunc :显示完整的镜像信息

2.在docker仓库中查找镜像

docker search [OPTIONS] TERM

		TERM：镜像名字

3.从仓库中拉镜像

  docker pull [OPTIONS] NAME[:TAG|@DIGEST]
 
   		NAME:镜像名字
   		TAG:具体的版本


4.从本地删除镜像

docker rmi [OPTIONS] IMAGE [IMAGE...]

 Options说明:
            -f：强制
           IMAGE：镜像ID


### 容器命令 ###

   有镜像才能创建容器，这是根本前提(下载一个CentOS镜像演示)

	docker pull centos	

1.新建并启动容器

  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

  OPTIONS说明（常用）：

         有些是一个减号，有些是两个减号
 
		--name="容器新名字": 为容器指定一个名称；
		-d: 后台运行容器，并返回容器ID，也即启动守护式容器；
		-i：以交互模式运行容器，通常与 -t 同时使用；
		-t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；
		-P: 随机端口映射；
		-p: 指定端口映射，有以下四种格式

      			ip:hostPort:containerPort
      			ip::containerPort
      			hostPort:containerPort
      			containerPort

  注意：ip为宿主机IP，即你在哪台电脑上安装的Docker 那么ip就是这    台的ip，hostPort就是这台的端口
  containerPort： 容器的端口	


例如：使用镜像centos:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。

	docker run -it centos /bin/bash 



2.列出当前所有正在运行的容器

	docker ps [OPTIONS]		

OPTIONS说明（常用）：
 
	-a :列出当前所有正在运行的容器+历史上运行过的
	-l :显示最近创建的容器。
	-n：显示最近n个创建的容器。
	-q :静默模式，只显示容器编号。
	--no-trunc :不截断输出。

3.退出容器
   
  容器停止退出 ：exit
  
  容器不停止退出 ：ctrl+P+Q

4.启动容器

		docker start [OPTIONS] CONTAINER

CONTAINER：容器ID或者容器名

5.重启容器

	docker restart [OPTIONS] CONTAINER

CONTAINER：容器ID或者容器名

6.停止容器
    
	docker stop [OPTIONS] CONTAINER

CONTAINER：容器ID或者容器名

7.强制停止容器
		
	docker kill [OPTIONS] CONTAINER

CONTAINER：容器ID或者容器名

8.删除已停止的容器

	docker rm [OPTIONS] CONTAINER

CONTAINER：容器ID或者容器名

9.一次性删除多个容器

	docker rm -f $(docker ps -a -q)
   
    docker ps -a -q | xargs docker rm

10.启动守护式容器

	docker run -d CONTAINER

CONTAINER：容器ID或者容器名

 
11.查看容器日志
  
	docker logs [OPTIONS] CONTAINER

CONTAINER：容器ID或者容器名

OPTIONS说明（常用）：
		
       	-t 是加入时间戳
      	-f 跟随最新的日志打印
	 	--tail 数字 显示最后多少条

12.查看容器内运行的进程

	docker top CONTAINER

CONTAINER：容器ID

13.查看容器内部细节

	docker inspect [OPTIONS] NAME|ID	

NAME|ID:容器ID或者容器名

14.进入正在运行的容器并以命令行交互
	
	docker exec [OPTIONS] CONTAINER COMMAND

CONTAINER:容器ID或者容器名
OPTIONS说明（常用）：

		-i：以交互模式运行容器，通常与 -t 同时使用；
		-t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；

	docker attach [OPTIONS] CONTAINER

CONTAINER:容器ID或者容器名

上述两个区别:

   attach 直接进入容器启动命令的终端，不会启动新的进程

   exec 是在容器中打开新的终端，并且可以启动新的进程

15.从容器内拷贝文件到主机上

    docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
    docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

CONTAINER:容器ID
SRC_PATH:容器内路径 
DEST_PATH:目的主机路径


16.提交镜像

	docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

CONTAINER：容器ID或者容器名
REPOSITORY：到仓库中的名字
TAG：版本

17：使用Dockerfile构建镜像
	
	docker build [OPTIONS] PATH | URL | -


 
