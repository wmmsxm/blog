---
title: "Docker学习(六) 自定义tomcat镜像"
date: 2020-12-10T14:09:32+08:00
tags: ["docker"]
categories: ["docker", "learn"]
draft: false
---
## <center>Docker学习(六) 自定义tomcat镜像</center>
### 前期准备
    1、tomcat: tomcat版本选择的是apache-tomcat-8.5.61，
        文件名：apache-tomcat-8.5.61.tar.gz
    2、java: java版本选择的是JDK1.8.0_211，
        文件名：jdk-8u211-linux-x64.tar.gz
以上文件均可在[这里](https://pan.baidu.com/s/14-8cf3OLXi841bc_OkpoGA)下载， 提取码：【tf2u】  
![文件](/blog/images/docker/docker6-1.png)

### Dockerfile文件
    # centos在这里作为tomcat的基础镜像
    FROM centos     
    MAINTAINER wmm
    #（这个环境变量用来表示该镜像模板的最后更新时间）
    ENV REFRESHED_AT 2020-12-10  

    # 切换镜像目录，进入/usr目录
    WORKDIR /usr
    # 在/usr/下创建jdk目录,用来存放jdk文件
    RUN mkdir jdk
    # 在/usr/下创建tomcat目录，用来存放tomcat
    RUN mkdir tomcat

    # 将宿主机的jdk目录下的文件拷至镜像的/usr/jdk目录下
    ADD jdk-8u211-linux-x64.tar.gz /usr/jdk/
    # 将宿主机的tomcat目录下的文件拷至镜像的/usr/tomcat目录下
    ADD apache-tomcat-8.5.61.tar.gz /usr/tomcat/

    # 给权限
    RUN chmod -R 777 /usr/tomcat/
    RUN chmod -R 777 /usr/jdk/

    # 设置环境变量
    ENV JAVA_HOME=/usr/jdk/jdk1.8.0_211
    ENV JRE_HOME=$JAVA_HOME/jre
    ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
    ENV PATH=/sbin:$JAVA_HOME/bin:$PATH

    # 设置时间区域
    RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone

    # 公开端口
    EXPOSE 8080
    # 设置启动命令
    ENTRYPOINT ["/usr/tomcat/apache-tomcat-8.5.61/bin/catalina.sh","run"]

### 打包
    1、将apache-tomcat-8.5.61.tar.gz、jdk-8u211-linux-x64.tar.gz、Dockerfile放在同一级目录
    2、命令进入该目录，执行 docker build -t yuzhi/tomcat .
    3、执行完后，使用命令docker images 查看镜像列表。

### 运行
    docker run -d -p 8080:8080 --name wmm_tomcat yuzhi/tomcat:latest


### 具体执行步骤
    1、进入需要组装镜像的路径
    2、执行命令  docker build -t 镜像名:版本号  .	        # 也可以直接是镜像名  这样版本默认是latest
    3、docker images   							            # 查看镜像   即可以看到创建的镜像
    4、docker run -d -p 映射端口:容器的端口 --name 自定义容器名称 镜像名:版本号   
    例如： docker run -d -p 8080:8080 --name wmm_tomcat yuzhi/tomcat:latest
    5、docker logs -f --tail=200 容器名称                   # 查看日志
    6、docker exec -it 容器名称 /bin/bash			        # 进入容器