---
title: "Centos Python3"
date: 2022-10-22T16:40:33+08:00
draft: false
tags: ["centos"]
categories: ["centos","python"]
---
## <center>Centos环境 python升级python3</center>

#### 查看系统远python版本
```
[root@localhost ~]# python
// 默认是2.7.5

```
#### 下载Python3.8.0
Python3.8.0[下载地址](https://www.python.org/ftp/python/3.8.0/)    

#### 依赖环境gcc、zlib
查看gcc是否安装  
```
whereis gcc  或者
gcc --version

# 没有安装的话需要制作离线安装包
yum install --downloadonly --downloaddir=/root/k8sOfflineSetup/packages \
    gcc-c++
# 将下载的包打包并上传到服务器，并执行下面命令：
rpm -ivh * --nodeps --force

# 如果再make install报错 zlib not available时，安装下面zlib包
yum install --downloadonly --downloaddir=/root/k8sOfflineSetup/packages \
    zlib \
    zlib-devel
yum install -y 
```  

#### 安装python3
安装步骤：
```
1、解压安装宝
tar -zxvf Python-3.8.0.tgz
2、进入解压后的文件夹
./configure --prefix=/usr/local/python3
# 编译
make
# 安装
make install

```

#### 替换默认python2
```
ll /usr/bin/ | grep python
# 可以看到python指向python2

#修改名称再添加python3的软连接
mv /usr/bin/python /usr/bin/old_python
ln -s /usr/local/python3/bin/python3.8 /usr/bin/python

# pip更换
mv /usr/bin/pip /usr/bin/old_pip
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip

# 查看版本
pip --version
```

#### python运行依赖包
```
# 将已安装的依赖包导出
pip freeze > requirements.txt
# 离线文件导出
pip download -r requestments.txt -d ./pip_packages

# 将 whl 包下载后放至某文件夹，如 /root/pip-packages
pip install --no-index --find-links=/root/pip-packages <软件名>
```

#### 升级pip

在离线环境下，pip安装离线包时有时候会报错，首先看下下载依赖包的pip版本和当前服务器版本是否一致  
不一致的情况下，需要更新pip版本。 [版本下载地址](https://pypi.org/project/pip/#files)
```
python -m pip install --upgrade xxx(注意：pip轮子的文件名.whl，一定要有后缀)
```
