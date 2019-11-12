---
title: CentOS7安装Docker
date: 2019-08-09 10:05:01
tags: 
    - Docker
    - CentOS
---


1.官网中文安装参考手册

[https://docs.docker-cn.com/engine/installation/linux/docker-ce/centos/#prerequisites](https://docs.docker-cn.com/engine/installation/linux/docker-ce/centos/#prerequisites "官网中文安装参考手册")

2.确定你是CentOS7及以上版本
	

	cat /etc/redhat-release



3.备份原有的yum源，以防出错时还可以还原。

```
 yum install -y wget
```

```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

4.进入yum.repos.d目录。

```
cd /etc/yum.repos.d
```

5.下载相应的源。(我用的是163yum源)

```
wget  http://mirrors.163.com/.help/CentOS7-Base-163.repo  #centos7系统的
wget  http://mirrors.163.com/.help/CentOS6-Base-163.repo  #centos6系统的
wget  http://mirrors.163.com/.help/CentOS5-Base-163.repo  #centos5系统的（注：如果下载出现错误，可能是变更了下载的地址，可以到：http://mirrors.163.com/.help/centos.html 找新的下载地址。没有安装wget的，要先安装。yum -y install wget）
```

6.更名。

```
mv CentOS-Base-163.repo /etc/yum.repos.d/CentOS-Base.repo
```

7.产生新的缓存。

```
yum clean all
yum makecache
```

至此源更新完成，可以 yum -y update 测试一下更新的速度。



8.yum安装gcc相关
<!--more-->
	yum -y install gcc
	yum -y install gcc-c++

9.卸载旧版本

```shell
yum -y remove docker docker-common docker-selinux docker-engine
```

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

10.安装需要的软件包

	yum install -y yum-utils device-mapper-persistent-data lvm2

11.设置stable镜像仓库
		

	yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
	


注意：千万不要用官方给出来的镜像仓库，特别慢！特别慢！特别慢！

12.更新yum软件包索引

	yum makecache fast

13.安装DOCKER CE

	yum list docker-ce --showduplicates | sort -r 
	yum -y install docker-ce-18.09.5

14.启动docker

	systemctl start docker

15.测试

	docker version


	docker run hello-world

16.配置镜像加速
	

	mkdir -p /etc/docker
	
	vim  /etc/docker/daemon.json

 在daemon.json加入以下内容：


 	#网易云

	{"registry-mirrors": ["http://hub-mirror.c.163.com"] }

 


 	#阿里云

 获取阿里云加速地址，需要注册才能获取，具体怎么操作，可以google

https://cr.console.aliyun.com/undefined/instances/mirrors 

	{"registry-mirrors": ["https://｛自已的编码｝.mirror.aliyuncs.com"]}


	systemctl daemon-reload


	systemctl restart docker

17.如果需要卸载，请走以下命令
	

	systemctl stop docker 
	
	yum -y remove docker-ce
	
	rm -rf /var/lib/docker

完美 ！从入门到卸载！
	
	
