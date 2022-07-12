---
title: "Docker学习(一) Centos7环境"
date: 2019-12-03T10:11:50+08:00
tags: ["docker"]
categories: ["docker", "learn"]
draft: false
---
## <center>Docker学习(一) Centos7环境</center>
### 虚拟机VMware安装
玄襄的电脑是windows系统，所以为了模拟线上环境，使用VMware虚拟机创建Centos系统。 玄襄使用的是VMware15 pro版本  
为了方便下载，在[这里](https://pan.baidu.com/s/1VZ4Vg8b5F3TB8wozQ54j8w)提供了下载链接，网盘密码是【7k58】，有需要的可以下载。
文件如下：
![vmware](/images/docker/docker1.png)

1. 安装vmware，选择安装位置，中间有可能需要重启，一路下一步直到安装完成。
2. 安装完之后，打开注册机KeyGen.exe生成key，然后打开vmware应用，把key复制到需要填写的许可证秘钥即可，点击确定破解完成。

### 创建Centos7虚拟机
玄襄使用的是Centos7，废话不多说，下载地址在[这里](https://pan.baidu.com/s/1GoyC5kRicueZ2x_7VggvHg)，密码【ogv0】。
下载完之后，就可以直接创建虚拟机了。

1. 点击vmware的文件选项，选择【创建虚拟机】![创建虚拟机](/images/docker/docker1-1.png)，弹出如下向导：![安装向导](/images/docker/docker1-2.png)直接选择【典型(推荐)】即可，点击下一步；
2. ![稍后安装系统](/images/docker/docker1-3.png) 此处现在稍后安装系统，点击下一步；
3. 选择【Linux】操作系统，版本选择【Centos7 64位】，点击下一步；
4. 【虚拟机名称】和【位置】  根据自己的情况设置即可，点击下一步；
5. ![磁盘大小](/images/docker/docker1-4.png) 磁盘大小一般设置20GB, 选择【将虚拟磁盘拆分成多个文件】，点击下一步；
6. ![自定义硬盘](/images/docker/docker1-5.png) 选择【自定义硬件】，在【硬件】窗口中选择 【新CD/DVD】,修改右侧的【连接】，使用ISO映像文件，选择本地下载好的镜像文件即可。
![硬件](/images/docker/docker1-6.png)点击关闭， 最后点击完成。

### Centos7系统安装
在上一步创建完虚拟机之后，vmware左侧【我的计算机】菜单会出现创建好的虚拟机。![开始](/images/centos/centos1-1.png)
点击【开启此虚拟机】进行系统安装初始化。

1. 开启之后  首先我们选择第二个选项 检查并安装Centos7，一堆命令行之后，会出现初始化界面![初始化](/images/centos/centos1-2.png)，推荐默认english，直接continue即可；
2. ![安装信息摘要](/images/centos/centos1-3.png) 出现叹号的地方，需要我们点击进行进行确认，然后点击完成即可。![目标位置](/images/centos/centos1-4.png)，点击开始安装；
3. 配置界面需要设置下root密码（简单的密码，需要点击2次完成才能设置成功），安装进度条安装完毕后，点击重启即可。![设置密码](/images/centos/centos1-5.png)

### Centos7网络设置
在新装的Centos系统中，网络基本都是不通的，使用root和密码登录系统之后，使用 ping www.baidu.com会出现如图情况 ![ping](/images/centos/centos1-6.png)
需要我们重新设置网络
1. 使用 ip addr 会发现 eno16777736(文件名可能不一样)，下一步要找到这个文件![eno](/images/centos/centos1-7.png)
2. 使用 cd /etc/sysconfig/network-scripts/  进入文件中， 使用 dir 命令可以看到 ifcfg-eno16777736 文件
3. 使用 vi ifcfg-eno16777736  查看内容 按 【i】 进入编辑模式， 将ONBOOT=no改成ONBOOT=yes， 按【esc】退出编辑模式， 使用【:wq】保存并退出vi。 ![内容](/images/centos/centos1-8.png)
4. 使用 service network restart 重启网络服务即可。

### 配置阿里源

阿里源地址：<http://mirrors.aliyun.com/repo/>
    
1. 下载阿里源配置  
    yum -y install wget  
    mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak  
    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
2. 清除缓存并生成新的缓存  
    yum clean all  
    yum makecache  
3. 验证是否成功，安装net-tools  
    yum -y -install net-tools  
    ifconfig  
    成功后会出现以下结果：![ifconfig](/images/centos/centos1-9.png)
