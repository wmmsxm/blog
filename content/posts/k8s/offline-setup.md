---
title: "K8s离线部署-离线部署"
date: 2022-09-22T11:15:35+08:00
draft: false
tags: ["k8s"]
categories: ["k8s"]
---
## <center>K8s离线部署-离线部署</center>
#### 环境
服务器|IP
|:--:|:--:|
Master|192.168.174.128
Node1|192.168.174.129
Node2|192.168.174.130

将制作的离线安装包上传到各节点,解压到`/root/k8sOfflineSetup`目录,<font color='red'>*解压路径不能修改*</font>

#### Master节点安装
在`master`节点执行下面操作:
```
1、先将文件copy到服务器root目录下
2、tar -xvf k8sOfflineSetup.tar
3、切换到packages包下  先将依赖包全部安装  rpm -ivh * --nodeps --force
4、创建环境变量
    # master节点的主机名
	export HOSTNAME=k8s-master
    # kubernetes apiserver的主机地址
	export APISERVER_NAME=apiserver.k8s.com
    # 集群中master节点的ip地址
	export MASTER_IP=192.168.174.128
    # Pod 使用的网段
	export POD_SUBNET=10.11.10.0/16
5、回到k8sOfflineSetup文件夹
   执行./setup_master.sh
```
执行完毕后，执行下面操作获取加入master的参数
```
# 在 master 节点执行
kubeadm token create --print-join-command

# 得到token和cert，这两个参数在2个小时内可以重复使用，超过以后就得再次生成
kubeadm join apiserver.k8s.com --token mpfjma.4vjjg8flqihor4vt     --discovery-token-ca-cert-hash sha256:6f7a8e40a810323672de5eee6f4d19aa2dbdb38411845a1bf5dd63485c43d303
```

#### Node节点安装
在每个node节点执行下面操作：
```
1、先将文件copy到服务器root目录下
2、tar -xvf k8sOfflineSetup.tar
3、切换到packages包下  先将依赖包全部安装  rpm -ivh * --nodeps --force
4、创建环境变量
    # node节点的主机名
    export HOSTNAME=node1
    # kubernetes apiserver的主机地址
    export APISERVER_NAME=apiserver.k8s.com
    # 集群中master节点的ip地址
    export MASTER_IP=192.168.174.128
    # 加入master的token
    export TOKEN=mpfjma.4vjjg8flqihor4vt
    # 加入master的证书
    export CERT=sha256:6f7a8e40a810323672de5eee6f4d19aa2dbdb38411845a1bf5dd63485c43d303
5、回到k8sOfflineSetup文件夹，执行./setup_worker.sh
```
每个node节点之后完之后，就已经部署完成了。可执行`kubectl get nodes`看所有节点的情况。

#### 访问Kuboard
Kuboard是一个非常方便的web管理界面，安装完以后可以通过[http://任意节点IP:32567/](http://xn--ip-dh3cr99d42rrmy:32567/)访问。   

##### 获取登录token
```
# 在 Master 节点上执行此命令
kubectl -n kube-system get secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}') -o go-template='{{.data.token}}' | base64 -d
```