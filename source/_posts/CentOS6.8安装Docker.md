---
title: CentOS6.8安装Docker
date: 2019-08-08 18:05:01
tags: Docker,CentOS6.8
---




1.Docker使用EPEL发布，RHEL系的OS首先要确保已经持有EPEL仓库，否则先检查OS的版本，然后安装相应的EPEL包。

	yum install -y epel-release


2.安装docker

    yum install -y docker-io

3.安装后的配置文件：

	/etc/sysconfig/docker

<!--more-->

4.启动Docker后台服务
	
	service docker start

5.docker version验证

	docker version

6.测试

  	docker run hello-world

7.配置镜像加速

   	mkdir -p /etc/docker
 
	vim  /etc/docker/daemon.json
   
加入以下内容：


     
 #网易云

	{"registry-mirrors": ["http://hub-mirror.c.163.com"] }
 
 
 
 #阿里云

	{"registry-mirrors": ["https://｛自已的编码｝.mirror.aliyuncs.com"]}

8.重新加载文件
     
    systemctl daemon-reload

9.重启docker
	
	systemctl restart docker



----------

说明：本内容整理于尚硅谷周阳老师的思维导图，如有侵权请联系删除！

----------
 
  
	
