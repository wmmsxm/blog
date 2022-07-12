---
title: "redis集群"
date: 2019-10-17T09:02:40+08:00
tags: ["redis"]
categories: ["redis"]
draft: false
---

<meta name="referrer" content="no-referrer" />

### windows搭建redis集群  
    操作系统：win10 64位  
    redis版本：3.2.100 x64  
    ruby版本：2.5.1 x64  
    rubygems版本：2.7.6  

#### 开场白  
    昨天同事需要本地的redis集群环境，在网上找了点资料，历经各种坑，终于搭建完成。今天特地整理一下搭建的流程，加深下印象。
    安装工具在百度网盘下载：   
    链接：https://pan.baidu.com/s/1LxWEfF8bQGIxd2sxN3N3Pw
    提取码：sqmw 

#### 开始搭建
    搭建顺序就是 redis --> 复制多份redis配置 --> 安装ruby --> 安装rubygems --> 进行集群构建脚本

1.  redis安装

    下载的redis文件进行解压，最好是放在新的文件夹中，方便集群使用和管理。  
    ![文件](/blog/images/redis/20191017094637.png)
    
2. 配置三主三从集群

    1. 因为集群最少是6个节点，所以这里配置三主三从，将上面的redis文件夹复制5份，我这边端口修改是6479,6480,6481,6482,6483,6484。如下所示：
    ![三主三从](/blog/images/redis/20191017095130.png)
    其中每一个文件夹都相当于一个redis。

    2. 配置redis
        以6479为例，打开文件夹中的redis.windows.conf文件，分别修改如下数据： 
 
            port 6479     ---> 端口号，与文件夹保持一致  
            cluster-enabled yes  ---> 开始实例的集群模式  
            cluster-config-file nodes-6479.conf  ---> 设定保存节点配置文件的路径，端口与文件保持一致  
            cluster-node-timeout 15000 ---> 创建集群时超时时间 
            appendonly yes ---> 开启后，每次写操作请求都追加到appendonly.aof 文件中  
        **注意，修改时前面不能有空格！！！**  
          
        在每个文件夹中添加一个bat来启动redis，内容如下：

            title redis-6479         //和当前端口保持一致
            redis-server.exe redis.windows.conf
        创建完成之后，分别点击这个bat，启动redis。  
    3. 安装ruby  
        redis的集群需要ruby环境，所以需要使用下载的rebyinstaller，基本上都勾选，然后一路下一步，最后让你选择，没搞懂啥意思，本人选了3。
    4. 安装rubygems  
        解压完成之后，进入文件夹使用powershell，运行如下命令:  
        
            1. ruby setup.rb           // 直接安装即可
            2. gem install redis       // 安装redis插件
    5. redis-trib.rb
        将下载的redis-trib.rb放在redis同级。使用powershell运行如下命令：  

            redis-trib.rb create --replicas 1 127.0.0.1:6479 127.0.0.1:6480 127.0.0.1:6481 127.0.0.1:6482 127.0.0.1:6483 127.0.0.1:6484

            其中会出现Can I set the above configuration? (type 'yes' to accept)，输入yes
            最后出现两个OK的时候差不多就是构建成功了。
        
        注意，如果需要绑定IP地址的话，需要修改redis中的属性 bind 127.0.0.1， 改为bind (你的ip)