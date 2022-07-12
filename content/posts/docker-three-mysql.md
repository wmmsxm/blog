---
title: "Docker学习(三) Docker安装mysql"
date: 2019-12-04T11:17:31+08:00
tags: ["docker"]
categories: ["docker", "learn"]
draft: false
---
## <center>Docker学习(三) Docker安装mysql</center>
### 安装mysql简述
  Docker安装mysql比较方便，主要是以下步骤：  
  1. 确认安装mysql的版本 访问Mysql的版本库，点击[这里](https://hub.docker.com/_/mysql?tab=tags) 查看版本
  ![版本](/blog/images/docker/docker3-1.png)
  2. 选择版本后，复制后面的命令  拉取mysql镜像  docker pull mysql:latest
  ![mysql](/blog/images/docker/docker3-2.png)
  3. 根据mysql镜像启动mysql容器  
  docker run --name mysql --privileged=true -v /docker/mysql/data:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql   
  参数说明：  
  run           运行一个容器  
  --name        容器的名称  
  --privileged=true  开启特权模式 
  -v /docker/mysql/data:/var/lib/mysql    将容器中的/var/lib/mysql文件挂载到宿主/docker/mysql/data上，这样删除容器后，数据也不会丢失
  -p 3306:3306  表示这个容器使用3306端口(第二个) 映射到本机的端口号也为3306(第一个)  
  -d            服务后台运行
  4. 外部客户端链接数据库 使用虚拟机的ip + 3306端口  密码是123456即可访问

  ### mysql服务停止和重启
  1. docker查看正在运行的容器   $ docker ps -s                ![正在运行的容器](/blog/images/docker/docker3-3.png)
  2. docker停止运行的容器       $ docker stop 5fa83e71137d        5fa83e71137d 是容器id
  3. docker启动容器            $ docker start 5fa83e71137d      