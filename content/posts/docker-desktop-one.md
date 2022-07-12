---
title: "Docker_Desktop学习(一) 环境安装"
date: 2021-12-03T10:11:26+08:00
draft: false
tags: ["docker-desktop"]
categories: ["docker","docker-desktop"]
---
## <center>Docker_Desktop学习(一) 环境安装</center>
### windows安装docker_desktop
1、我们先去[官网下载安装包](https://www.docker.com/products/docker-desktop)![dd1](/blog/images/docker_desktop/dd1_1.png)  
2、打开安装包加载一会后一般会弹出两个选项,在较旧的Windows10或之前的系统会出现如下所示的相关提示。我们把第一个选上,第二个根据需求选择即可。![dd2](/blog/images/docker_desktop/dd1_2.png)   
3、如下图,这里推荐使用WSL2。使用WSL2(基于Windows的Linux子系统),如果我们不适用,就会使用Hyper-v虚拟机运行,不过相比于虚拟机,子系统在性能方面更加出色。![dd3](/blog/images/docker_desktop/dd1_3.png)在我们选择使用WSL2之后,并且我们也确定打开了如下图所示的Windows功能(如果没有打开,请先百度如何打开wsl。)![dd5](/blog/images/docker_desktop/dd1_5.png)  
3.1、如果之后安装完成后发生报错可能是WSL2版本比较老,需要更新导致的。![dd4](/blog/images/docker_desktop/dd1_4.png)  需要我们自己手动更新一下,我们根据提示去微软官网下载最新版的wsl2安装后即可正常打开。[更新包下载链接](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)  
### docekr_desktop换源   
在docker_desktop中选择setting,然后选择Docker Engine在其中输入以下源
```
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com",
    "https://cr.console.aliyun.com/"
  ]
}
```

### docker_desktop使用
docker_desktop安装好之后，就可以使用命令窗口操作docker
#### 镜像操作
##### 镜像列表 
```
#所有镜像
docker images
```

| 标签 | 含义  
| ------| ------ | 
| REPOSITORY | 镜像所在的仓库名称 |
| TAG | 镜像标签 |
| IMAGEID | 镜像ID |
| CREATED | 镜像创建日期(不是获取该镜像的日期) |
| SIZE | 镜像大小 |  
  
![dd6](/blog/images/docker_desktop/dd1_6.png)

##### 镜像拉取
```
#镜像拉取
docker pull 镜像名称
比如
docker pull ubuntu
docker pull ubuntu:16.04
#个人镜像
docker pull 仓库名称/镜像名称
docker pull xuanxiang/mysql
#第三方私库
docker pull 第三方仓库地址/仓库名称/镜像名称
docker pull registry.cn-hangzhou.aliyuncs.com/fhzh/system:1.0.0

```

##### 镜像删除
```
docker image rm 镜像名或者镜像id
docker rmi 镜像名或者镜像id
比如
docker image rm mysql
docker rmi e34a19f7b66d
```

##### 加载镜像
```
docker run [可选参数] 镜像名 [向启动容器中传入的命令]
```

| 常用可选参数 | 作用
| ------| ------ |
| -i | 表示以《交互模式》运行容器 |
| -d | 会创建一个守护式容器在后台运行 | 
| -t | 表示容器启动后会进入其命令行。|
| --name | 为创建的容器命名。(默认会随机给名称，不支持中文字符)|
| -v | 表示目录映射关系，即宿主机目录:容器中目录 |
| -p | 表示端口映射，即宿主机端口:容器端口 | 
| --network=host | 表示将主机的网络环境映射到容器中，使容器的网络与主机相同。(windows和mac不生效)|

#### 容器操作
##### 查看容器
```
# 查看当前所有正在运行的容器
docker ps
# 查看当前所有容器
docker ps -a
# 使用过滤器
docker ps -f name=指定的名称
# 显示2个上次创建的容器(2可以改变)
docker ps -n 2
# 显示最新创建的容器(包括所有容器)
docker ps -l
# 仅显示ip
docker ps -q
# 显示容器大小
docker ps -s
```

| 标签 | 含义 | 
| ------ | ------ |
| CONTAINER ID | 容器id |
| IMAGE | 镜像ID |
| COMMAND | 默认启动命令(启动时会自动执行) |
| CREATED | 创建容器的日期 |
| STATUS | 当前的状态(启动了多久,多久之前退出等) |
| PORTS | 映射的端口 |
| NAMES | 容器的名称 |  
  
![dd7](/blog/images/docker_desktop/dd1_7.png)



##### 启动和关闭容器
```
# 停止容器
docker container stop 容器名或者容器id
# 或可简写成
docker stop 容器名或容器id

# 强制关闭容器
docker container kill 容器名或容器id
# 或可简写成
docker kill 容器名或容器id

# 启动容器
docker container start 容器名或容器id
# 或可简写成
docker start 容器名或容器id

# 重启容器
docker restart 容器名或容器id
```


##### 操作后台容器
我们开启容器后,如果需要在容器内执行命令,可以将后台切换到前台,也可能使用docker命令将我们需要执行的命令传入。  
操作方法有很多种,这里我们介绍一些比较常用的方法
```
# 执行单条命令
docker exec -it 容器名或容器id 执行的命令
#比如
docker exec -it mysql whoami
# 进入容器内部
docker exec -it 容器名或容器id /bin/bash
# 比如
docker exec -it mysql /bin/bash
# 除了exec外还有attach可以使用，attach有弊端，多终端启动attach后，都会同步显示。但如果一个窗口阻塞，其他窗口也无法再进行操作。
docker attach 容器名或容器id
docker attach mysql
```

| exec可选参数 | 作用 |
| ------ | ------ |
| -d | 会创建一个守护式容器在后台运行 |
| -e | 设置环境变量 |
| -i | 表示以《交互模式》运行容器 |
| -t | 表示容器启动后会进入其命令行。 |
| -u | 设置用户名和UID。|
| -w | 设置容器内的工作目录。|


##### 删除容器  
需要删除一个容器,首先需要确保这个容器已经停止了,因为正在运行的容器是无法直接删除。
```
docker rm 容器名或者容器id
docker rm mysql
```
