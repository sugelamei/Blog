---
title: CentOS7安装Docker
date: 2019-08-09 10:05:01
tags: 
    - Docker
    - CentOS7
---


1.官网中文安装参考手册

[https://docs.docker-cn.com/engine/installation/linux/docker-ce/centos/#prerequisites](https://docs.docker-cn.com/engine/installation/linux/docker-ce/centos/#prerequisites "官网中文安装参考手册")

2.确定你是CentOS7及以上版本
	
	cat /etc/redhat-release

3.yum安装gcc相关
<!--more-->
	yum -y install gcc
	yum -y install gcc-c++

4.卸载旧版本

	yum -y remove docker docker-common docker-selinux docker-engine

或者
  
      yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

5.安装需要的软件包

	yum install -y yum-utils device-mapper-persistent-data lvm2

6.设置stable镜像仓库
		
	yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

	
注意：千万不要用官方给出来的镜像仓库，特别慢！特别慢！特别慢！

7.更新yum软件包索引

	yum makecache fast

8.安装DOCKER CE

	yum -y install docker-ce

9.启动docker

	systemctl start docker

10.测试

	docker version


	docker run hello-world
	
11.配置镜像加速
	
	mkdir -p /etc/docker

	vim  /etc/docker/daemon.json

 在daemon.json加入以下内容：

	 
 	#网易云

	{"registry-mirrors": ["http://hub-mirror.c.163.com"] }
 
 
 
 	#阿里云

	{"registry-mirrors": ["https://｛自已的编码｝.mirror.aliyuncs.com"]}


	systemctl daemon-reload


	systemctl restart docker

12.如果需要卸载，请走以下命令
	
	systemctl stop docker 

	yum -y remove docker-ce
	
	rm -rf /var/lib/docker

完美 ！从入门到卸载！
	
	
