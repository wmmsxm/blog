---
title: "K8s离线部署-NFS-Docker-Harbor"
date: 2022-09-22T13:50:26+08:00
draft: false
tags: ["k8s"]
categories: ["k8s"]
---
## <center>K8s离线部署-NFS-Docker-Harbor</center>
#### 站点服务器安装nfs服务
```
# 执行命令准备nfs离线资源
yum -y install nfs-utils --downloadonly --downloaddir /root/nfs
打包nfs.tar并执行下面操作：
1、先将文件copy到服务器root目录下
2、tar -xvf nfs.tar
3、切换到packages包下  先将依赖包全部安装  rpm -ivh * --nodeps --force
4、执行下面步骤：
    #创建nfs目录
    mkdir -p /nfs/data/

    #修改权限
    chmod -R 777 /nfs/data

    #编辑export文件
    vi /etc/exports
    /nfs/data 192.168.0.0/24(rw,no_root_squash,sync)  （“*“代表所有人都能连接，建议换成具体ip或ip段，如192.168.20.0/24）

    #配置生效
    exportfs -r
    #查看生效
    exportfs

    #启动rpcbind、nfs服务
    systemctl restart rpcbind && systemctl enable rpcbind
    systemctl restart nfs && systemctl enable nfs

    #查看 RPC 服务的注册状况
    rpcinfo -p localhost

    #showmount测试
    showmount -e 192.168.92.56
    # 关闭 防火墙  否则其他服务不能访问
    systemctl stop firewalld
    systemctl disable firewalld

    #所有node节点安装客户端 执行1、2、3

5、为申请PV划分磁盘
    #创建pv卷对应的目录
    mkdir -p /nfs/data/pv001
    mkdir -p /nfs/data/pv002

    #配置exportrs
    vi /etc/exports
    /nfs/data 192.168.0.0/24(rw,no_root_squash,sync)
    /nfs/data/pv001 192.168.0.0/24(rw,no_root_squash,sync)
    /nfs/data/pv002 192.168.0.0/24(rw,no_root_squash,sync)

    #配置生效
    exportfs -r
    #重启rpcbind、nfs服务
    systemctl restart rpcbind && systemctl restart nfs
```

#### harbor离线安装
1. 需要提前安装docker。  
docker-19.03.9.tgz离线包下载地址[https://download.docker.com/linux/static/stable/x86_64/](https://download.docker.com/linux/static/stable/x86_64/docker-19.03.9.tgz)  
2. harbor离线包下载地址[https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases)，这里选择的harbor-offline-installer-v1.10.10.tgz为例。  

安装步骤如下：
```
1、先安装docker  tar -zxvf docker-19.03.9.tgz
2、将解压docker文件夹全部移动到/usr/bin  cp -p docker/* /usr/bin
3、将docker注册为系统服务
    （1）在/usr/lib/systemd/system/目录下，创建docker.service文件
    （2）编辑docker.service文件    vi /usr/lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
 
[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
 
[Install]
WantedBy=multi-user.target			

4、重启生效
    systemctl daemon-reload
    systemctl start docker
5、查看docker状态  systemctl status docker
6、设置开机启动   systemctl enable docker   查看版本 docker version
6.1、# vi /etc/docker/daemon.json
    {
    "insecure-registries": ["192.168.229.148:8080"]
    }
	systemctl restart docker
7、安装docker-compose
    (1) mv /root/docker-compose-Linux-x86_64 /usr/bin/docker-compose
    (2) chmod +x /usr/bin/docker-compose
    (3) docker-compose - v
8、安装Harbor
    (1) tar -zxvf harbor-offline-installer-v1.10.10\(最新版2022-02-21\).tgz
    (2) cd /harbor   cp harbor.yml harbor.yml.tmpl
    (3) 编辑harbor.yml   vi ./harbor.yml
    (4) # 修改成你的ip
        hostname: 192.168.211.99
        # 修改端口号
        http:
            port: 8080
        # 如果不需要https，请注释掉https相关
        #https:
        #  port: 443
        #  certificate: /your/certificate/path
        #  private_key: /your/private/key/path
        # 配置密码，将 Harbor12345换成你自己的密码
        harbor_admin_password: Fhzh@zkah!@#123
    （5）执行setenforce 0 
    (6) ./install.sh  即可安装成功  http://192.168.211.99:8080 即可访问
    (7) 常用命令
        # 回到harbor文件夹下 启动harbor  
        setenforce 0
        docker-compose up -d
        # 关闭harbor
        docker-compose down
9、 k8s配置镜像私库密钥
    kubectl create secret docker-registry harbor-secret  --namespace=fhzh --docker-server=20.184.3.15:8080 --docker-username=admin --docker-password=Fhzh@zkah!@#123
```