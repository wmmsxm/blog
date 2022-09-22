---
title: "K8s离线部署-资源准备"
date: 2022-09-22T09:58:40+08:00
draft: false
tags: ["k8s"]
categories: ["k8s"]
---
## <center>K8s离线部署-资源准备</center>
#### 背景
因当前客户提供服务器均为局域网，不能访问外网，固需要进行离线部署k8s。
#### 版本管理
软件|版本
|:----:|:----:|
Docker|19.03.5
Kubernetes|1.17.1
#### 获取yum的环境依赖
在一个可以联网的系统下载依赖包，注意这个系统要跟服务器的一致，并且是全新安装的，防止出现yum已安装过一些软件，导致依赖包可能不完整的问题。  
* 这里提供阿里源的[CentOS-7-x86_64-DVD-2009]下载地址(http://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-DVD-2009.iso?spm=a2c6h.25603864.0.0.5e776aearg27RG)    

依赖包下载到`/root/k8sOfflineSetup/packages`目录。

```
# 使用阿里云镜像源
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 创建本地仓库包
yum install --downloadonly --downloaddir=/root/k8sOfflineSetup/packages \
    createrepo

# 实用工具
yum install --downloadonly --downloaddir=/root/k8sOfflineSetup/packages \
    yum-utils \
    nfs-utils \
    wget

# docker 依赖包
yum install --downloadonly --downloaddir=/root/k8sOfflineSetup/packages \
    device-mapper-persistent-data \
    lvm2

# 添加阿里云Docker源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# docker
yum install --downloadonly --downloaddir=/root/k8sOfflineSetup/packages \
    docker-ce-19.03.5 \
    docker-ce-cli-19.03.5 \
    containerd.io

# 时间同步
yum install --downloadonly --downloaddir=/root/k8sOfflineSetup/packages \
    chrony

# HAProxy 和 KeepAlived
yum install --downloadonly --downloaddir=/root/k8sOfflineSetup/packages \
    haproxy \
    keepalived

# 配置K8S的yum源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# kubelet
yum install --downloadonly --downloaddir=/root/k8sOfflineSetup/packages \
    kubelet-1.17.1 \
    kubeadm-1.17.1 \
    kubectl-1.17.1

```
#### 获取kubeadm依赖镜像
获取kubeadm用到的镜像列表，这步操作要在一台安装好Docker环境的计算机上进行，拉取完成后打包复制到资源包`/root/k8sOfflineSetup/images`目录中。  
```
kubeadm config images list
```
得到的结果是以下这些镜像，因为镜像的服务器都在国外，下载速度慢或者根本无法下载。所以，我们先在阿里云上获取镜像后再改名放到资源包中。  
* k8s.gcr.io/kube-apiserver:v1.17.1
* k8s.gcr.io/kube-controller-manager:v1.17.1
* k8s.gcr.io/kube-scheduler:v1.17.1
* k8s.gcr.io/kube-proxy:v1.17.1
* k8s.gcr.io/pause:3.1
* k8s.gcr.io/etcd:3.4.3-0
* k8s.gcr.io/coredns:1.6.5
```
# 从阿里云拉取镜像
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.17.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.17.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.17.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.17.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.6.5

# 重新tag镜像
docker images \
    | grep registry.cn-hangzhou.aliyuncs.com/google_containers \
    | sed 's/registry.cn-hangzhou.aliyuncs.com\/google_containers/k8s.gcr.io/' \
    | awk '{print "docker tag " $3 " " $1 ":" $2}' \
    | sh

# 删除旧镜像
docker images \
    | grep registry.cn-hangzhou.aliyuncs.com/google_containers \
    | awk '{print "docker rmi " $1 ":" $2}' \
    | sh

# 在当前目录导出镜像为压缩包
docker save -o kube-controller-manager-v1.17.1.tar k8s.gcr.io/kube-controller-manager:v1.17.1
docker save -o kube-apiserver-v1.17.1.tar k8s.gcr.io/kube-apiserver:v1.17.1
docker save -o kube-scheduler-v1.17.1.tar k8s.gcr.io/kube-scheduler:v1.17.1
docker save -o kube-proxy-v1.17.1.tar k8s.gcr.io/kube-proxy:v1.17.1
docker save -o coredns-1.6.5.tar k8s.gcr.io/coredns:1.6.5
docker save -o etcd-3.4.3-0.tar k8s.gcr.io/etcd:3.4.3-0
docker save -o pause-3.1.tar k8s.gcr.io/pause:3.1
```
#### calico网络插件依赖镜像
从[Quickstart for Calico on Kubernetes](https://docs.projectcalico.org/v3.10/getting-started/kubernetes/)找到calico.yaml文件，命名为`calico-v3.10.3.yaml`保存到资源包`/root/k8sOfflineSetup/plugins`目录中。
```
cat calico-v3.10.3.yaml | grep image: | awk '{print $2}'
```
从文件中搜索`image`关键字找到如下依赖镜像。
* calico/cni:v3.10.3
* calico/pod2daemon-flexvol:v3.10.3
* calico/node:v3.10.3
* calico/kube-controllers:v3.10.3

```
# 拉取全部镜像
cat calico-v3.10.3.yaml \
    | grep image: \
    | awk '{print "docker pull " $2}' \
    | sh

# 在当前目录导出镜像为压缩包
docker save -o calico-cni-v3.10.3.tar calico/cni:v3.10.3
docker save -o calico-pod2daemon-flexvol-v3.10.3.tar calico/pod2daemon-flexvol:v3.10.3
docker save -o calico-node-v3.10.3.tar calico/node:v3.10.3
docker save -o calico-kube-controllers-v3.10.3.tar calico/kube-controllers:v3.10.3
```
同样拉取完成后打包复制到资源包`/root/k8sOfflineSetup/images`目录中。
#### kubernetes dashboard
从GitHub上找到Dashboard最新版本部署文件。
* Dashboard版本发布地址：[https://github.com/kubernetes/dashboard/releases](https://github.com/kubernetes/dashboard/releases)   

下载[v2.0.0-rc5](https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc5/aio/deploy/recommended.yaml)，命名为`dashboard-v2.0.0-rc5.yaml`保存到资源包`/root/k8sOfflineSetup/plugins`目录中。  

修改`dashboard-v2.0.0-rc5.yaml`的imagePullPolicy,默认是`Always`，注释掉。否则即使本地已有镜像dashboard启动时还是要从网上拉取。为了内网能正常运行，所以禁止每次运行都重新pull image。  

1. 找出镜像列表
```
cat dashboard-v2.0.0-rc5.yaml | grep image: | awk '{print $2}'
```
得到着两个镜像名称
* kubernetesui/dashboard:v2.0.0-rc5
* kubernetesui/metrics-scraper:v1.0.3

2. 拉取镜像
```
cat dashboard-v2.0.0-rc5.yaml \
    | grep image: \
    | awk '{print "docker pull " $2}' \
    | sh

# 在当前目录导出镜像为压缩包
docker save -o kubernetesui-dashboard-v2.0.0-rc5.tar kubernetesui/dashboard:v2.0.0-rc5
docker save -o kubernetesui-metrics-scraper-v1.0.3.tar kubernetesui/metrics-scraper:v1.0.3
```
同样拉取完成后打包复制到资源包`/root/k8sOfflineSetup/images`目录中。  

#### Kuboard  
查看Kuboard官网安装文档，从中获取到需要`kuboard.yaml`和`metrics-server.yaml`两个部署文件，中需要使用以下两个镜像，还是按前面到方法拉下来保存起来。  
* eipwork/kuboard:latest
* registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6 

```

# 拉取镜像
docker pull eipwork/kuboard:latest
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6

# 在当前目录导出镜像为压缩包
docker save -o kuboard-latest.tar eipwork/kuboard:latest
docker save -o kuboard-metrics-server-amd64-v0.3.6.tar registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6
```
同样拉取完成后打包复制到资源包`/root/k8sOfflineSetup/images`目录中。  

#### NGINX Ingress Controller  
同样在官方网站找到部署yaml文件，并获取镜像列。
* [https://kubernetes.github.io/ingress-nginx/deploy/](https://kubernetes.github.io/ingress-nginx/deploy/)  

这里下载[ingress-nginx-v0.29.0](https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.29.0/deploy/static/mandatory.yaml)，命名为`ingress-nginx-v0.29.0.yaml`保存到资源包`/root/k8sOfflineSetup/plugins`目录中。  
1. 找出镜像列表
```
cat ingress-nginx-v0.29.0.yaml | grep image: | awk '{print $2}'
```
得到如下镜像  
* quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.29.0

2. 拉取镜像  
```
cat ingress-nginx-v0.29.0.yaml \
    | grep image: \
    | awk '{print "docker pull " $2}' \
    | sh

# 如果网络原因拉取速度太慢可从这儿拉取再重新tag
# docker pull quay.azk8s.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.29.0
# docker tag quay.azk8s.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.29.0 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.29.0
# docker rim quay.azk8s.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.29.0

# 在当前目录导出镜像为压缩包
docker save -o nginx-ingress-controller-0.29.0.tar quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.29.0
```
同样拉取完成后打包复制到资源包`/root/k8sOfflineSetup/images`目录中。

#### 资源包打包  
将下载到`packages`、`images`与[https://github.com/scfido/k8s-offline-setup](https://github.com/scfido/k8s-offline-setup)仓库的文件合并，最后的目录应该是这样的。
```
📁k8sOfflineSetup
├── 📁gpg
├── 📁plugins
├── 📁repos
├── 📁scripts
├── 📁packages
├── 📁images
├── 📃setup_master.sh
└── 📃setup_worker.sh
```
压缩资源包
```
cd /root/k8sOfflineSetup
tar -czf k8sOfflineSetup.tar.gz *
```
现在就完成了可以离线安装到Kubernetes单节点Master集群到资源包。