---
title: "Docker学习(二) Docker安装"
date: 2019-12-03T15:31:56+08:00
tags: ["docker"]
categories: ["docker","learn"]
draft: false
---
## <center>Docker学习(二) Docker安装</center>
### Docker安装前提准备
为了操作方便，玄襄这里直接使用root用户执行。docker支持Centos系统的内核版本要高于3.10，首先检查内核，使用uname -r 命令
![内核版本](/blog/images/docker/docker2-1.png)  
Centos7满足Docker环境需求，可以继续安装Docker(可能无法正常运行18.06.x及以上版本)。

```
  # 更新最新yum内核
  yum update -y

  # 卸载旧版的docker(如果安装过的话)
  yum remove docker-*

  # 安装需要的软件包
  yum install -y yum-utils device-mapper-persistent-data lvm2
  
  # 设置yum源  yum-config-manager 是yum-utils下的  一般出现问题可能yum-utils是没有安装好 
  yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

```

### Docker安装

安装有问题的情况：

        
    1、 yum install docker-ce     安装完之后，启动不起来  failed to start docker application container engine
        各种搜索 说是版本问题，然后卸载安装的docker
    2、 yum install -y docker-ce-18.03.1.ce   依旧启动不起来，同样的错误  
        参照网上的解决方式   使用命令 curl -fsSL https://get.docker.com/ | sh  等待运行完成  问题依旧没有解决


成功历程:

    1、卸载安装的docker  yum remove docker-*
    2、重新更新内核      yum update 
    3、然后重启一下系统  reboot
    4、安装docker       yum install docker
    5、启动docker       systemctl start docker        ---- 很顺利
    6、docker版本       docker version
    7、docker开启启动   systemctl enable docker

![docker版本](/blog/images/docker/docker2-2.png)


### 删除本地文件
注意，docker 的本地文件，包括镜像(images), 容器(containers), 存储卷(volumes)等，都需要手工删除。默认目录存储在 <font color=#DC143C>/var/lib/docker。</font>