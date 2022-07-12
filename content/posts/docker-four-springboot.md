---
title: "Docker学习(四) 远程部署springboot项目" 
date: 2019-12-05T09:10:20+08:00
tags: ["docker"]
categories: ["docker", "learn"]
draft: false
---
## <center>Docker学习(四) 远程部署springboot项目</center>
### 远程部署简述
Docker远程部署项目使用的是DockerFile方式，使用IDEA集成Docker进行发布，使用起来比较方便。  
基本过程是：开放远程docker端口，本地连接docker，项目根目录创建DockerFile文件，项目打包，运行dockerFile文件。

### docker端口开放
    # 修改配置文件
    1. cd /usr/lib/systemd/system
    2. vi docker.service
    3. 在ExecStart=/usr/bin/dockerd-current 后面添加 -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock -H tcp://0.0.0.0:7654 
    解析： 2375为主管理端口，unix://var/run/docker.sock用于本地管理，7654是备用端口
  ![开发端口](/blog/images/docker/docker4-1.png)

    # 将管理地址写入/etc/profile
    4. echo 'export DOCKER_HOST=tcp://0.0.0.0:2375' >> /etc/profile
    5. source /etc/profile

    # 重新读取配置文件
    6. systemctl daemon-reload

    # 重启docker服务
    7. systemctl restart docker

### 连接远程docker

1. idea安装dokcer插件  
打开idea的settings，选择Plugins，搜索插件Docker，安装图片中的插件![插件](/blog/images/docker/docker4-2.png)  

2. 连接远程docker地址  
 安装完插件之后，添加远程docker的远程地址![远程](/blog/images/docker/docker4-3.png)  
 在idea的services工具进行连接docker![连接](/blog/images/docker/docker4-4.png)

3. 创建DockerFile文件
  在项目根目录添加dockerFile文件，内容如下：
  ```
    #指定jdk8，注意，改镜像包比较精简，精简掉了如字体等插件  
    #如果需要操作系统字体库，那么就得使用slim版本或者默认版本。需要操作系统字体库的程序例如：图片验证码、PDF导出。  
    #https://segmentfault.com/a/1190000016449865  
    #不涉及到以上问题的，默认请使用openjdk:8-alpine  
    FROM openjdk:8-alpine

    MAINTAINER wmm  
    #tomcat需要  
    VOLUME /tmp   
    #讲jar包加到根目录  
    ADD /target/*.jar app.jar  
    #设置时区  
    RUN echo "Asia/Shanghai" > /etc/timezone  
    RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  
    RUN sh -c 'touch /app.jar'  
    RUN echo $(date) > /image_built_at  
    #执行命令  
    ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar","app.jar"]  
```
4. 镜像启动  
  使用maven打包项目，然后打开dockerfile文件，点击下面图标![图标](/blog/images/docker/docker4-5.png)  
  选择编辑运行环境 如下图![运行环境](/blog/images/docker/docker4-6.png)  
  重新点击上一部的小图标，点击运行即可。  
  此时远程docker可以通过命令 docker ps -s 看到正在运行的项目
  ![启动](/blog/images/docker/docker4-7.png)
