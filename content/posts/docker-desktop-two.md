---
title: "Docker_Desktop学习(二) Kubernetes"
date: 2021-12-03T14:24:09+08:00
tags: ["docker-desktop"]
categories: ["docker","docker-desktop"]
draft: false
---
## <center>Docker_Desktop学习(二) Kubernetes</center>
### docker_desktop安装Kubernetes  
使用docker desktop安装kubernetes，打开settings，选择Kubernetes, Enable Kubernetes前打钩，然后点击右下角 Apply & Restart,耐心等待即可。![dd2-1](/blog/images/docker_desktop/dd2_1.png)  
等待一段时间后，如果成功即可；未成功则使用下面的方法安装
### 使用k8s-for-docker-desktop方式安装

1、git下载k8s-for-docker-desktop


    git clone https://github.com/AliyunContainerService/k8s-for-docker-desktop.git
    cd k8s-for-docker-desktop


![dd2-2](/blog/images/docker_desktop/dd2_2.png)  
2、 然后cmd命令窗口，执行命令`.\load_images.ps1`， 等待下载镜像  
3、 回到docker desktop的settings中kubernetes，点击"Reset Kubernetes Cluster",清空docker desktop的kubernetes的缓存。  ![dd2-3](/blog/images/docker_desktop/dd2_3.png)  
4、 重新点击Enable Kubernetes前打钩，然后点击右下角 Apply & Restart。等待重启之后即可安装成功。  

### kubectl基础命令  
![dd2-4](/blog/images/docker_desktop/dd2_4.png)  
基础命令包括 create、delete、get、run、expose、set、explain、edit  
#### create 命令：根据文件或者输入来创建资源

    # 创建Deployment和Service资源
    $ kubectl create -f demo-deployment.yaml
    $ kubectl create -f demo-service.yaml   

#### delete 命令：删除资源


    # 根据yaml文件删除对应的资源，但是yaml文件并不会被删除，这样更加高效
    $ kubectl delete -f demo-deployment.yaml 
    $ kubectl delete -f demo-service.yaml

    # 也可以通过具体的资源名称来进行删除，使用这个删除资源，同时删除deployment和service资源

    $ kubectl delete 具体的资源名称

#### get 命令 ：获得资源信息

    # 查看所有的资源信息
    $ kubectl get all
    $ kubectl get --all-namespaces

    # 查看pod列表
    $ kubectl get pod

    # 显示pod节点的标签信息
    $ kubectl get pod --show-labels

    # 根据指定标签匹配到具体的pod
    $ kubectl get pods -l app=example

    # 查看node节点列表
    $ kubectl get node 

    # 显示node节点的标签信息
    $ kubectl get node --show-labels

    # 查看pod详细信息，也就是可以查看pod具体运行在哪个节点上（ip地址信息）
    $ kubectl get pod -o wide

    # 查看服务的详细信息，显示了服务名称，类型，集群ip，端口，时间等信息
    $ kubectl get svc
    $ kubectl get svc -n kube-system

    # 查看命名空间
    $ kubectl get ns
    $ kubectl get namespaces

    # 查看所有pod所属的命名空间
    $ kubectl get pod --all-namespaces

    # 查看所有pod所属的命名空间并且查看都在哪些节点上运行
    $ kubectl get pod --all-namespaces  -o wide

    # 查看目前所有的replica set，显示了所有的pod的副本数，以及他们的可用数量以及状态等信息
    $ kubectl get rs

    # 查看已经部署了的所有应用，可以看到容器，以及容器所用的镜像，标签等信息
    $ kubectl get deploy -o wide
    $ kubectl get deployments -o wide

#### run 命令：在集群中创建并运行一个或多个容器镜像。  
run语法：`run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [--command] -- [COMMAND] [args...]`  

    # 示例，运行一个名称为nginx，副本数为3，标签为app=example，镜像为nginx:1.10，端口为80的容器实例

    $ kubectl run nginx --replicas=3 --labels="app=example" --image=nginx:1.10 --port=80

    # 示例，运行一个名称为nginx，副本数为3，标签为app=example，镜像为nginx:1.10，端口为80的容器实例，并绑定到k8s-node1上
    $ kubectl run nginx --image=nginx:1.10 --replicas=3 --labels="app=example" --port=80 --overrides='{"apiVersion":"apps/v1","spec":{"template":{"spec":{"nodeSelector":{"kubernetes.io/hostname":"k8s-node1"}}}}}'


#### expose 命令：创建一个service服务，并且暴露端口让外部可以访问

    # 创建一个nginx服务并且暴露端口让外界可以访问

    $ kubectl expose deployment nginx --port=88 --type=NodePort --target-port=80 --name=nginx-service

#### set 命令：配置应用的一些特定资源，也可以修改应用已有的资源

    使用 kubectl set --help查看，它的子命令，env，image，resources，selector，serviceaccount，subject。

    语法: resources (-f FILENAME | TYPE NAME) ([--limits=LIMITS & --requests=REQUESTS]


##### kubectl set resources 命令
这个命令用于设置资源的一些范围限制。  
资源对象中的Pod可以指定计算资源需求（CPU-单位m、内存-单位Mi），即使用的最小资源请求（Requests），限制（Limits）的最大资源需求，Pod将保证使用在设置的资源数量范围。  
对于每个Pod资源，如果指定了Limits（限制）值，并省略了Requests（请求），则Requests默认为Limits的值。  
    # 将deployment的nginx容器cpu限制为“200m”，将内存设置为“512Mi”
    $ kubectl set resources deployment nginx -c=nginx --limits=cpu=200m,memory=512Mi

    # 设置所有nginx容器中 Requests和Limits
    $ kubectl set resources deployment nginx --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256Mi

    # 删除nginx中容器的计算资源值
    $ kubectl set resources deployment nginx --limits=cpu=0,memory=0 --requests=cpu=0,memory=0


#### kubectl set selector 命令
设置资源的 selector（选择器）。如果在调用"set selector"命令之前已经存在选择器，则新创建的选择器将覆盖原来的选择器。  
selector必须以字母或数字开头，最多包含63个字符，可使用：字母、数字、连字符" - " 、点"."和下划线" _ "。如果指定了--resource-version，则更新将使用此资源版本，否则将使用现有的资源版本。  
注意：目前selector命令只能用于Service对象。  

    语法：selector (-f FILENAME | TYPE NAME) EXPRESSIONS [--resource-version=version]


#### kubectl set image 命令
​用于更新现有资源的容器镜像。  
可用资源对象包括：pod (po)、replicationcontroller (rc)、deployment (deploy)、daemonset (ds)、job、replicaset (rs)。   

    语法：image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N 

    # 将deployment中的nginx容器镜像设置为“nginx：1.9.1”
    $ kubectl set image deployment/nginx busybox=busybox nginx=nginx:1.9.1

    # 所有deployment和rc的nginx容器镜像更新为“nginx：1.9.1”
    $ kubectl set image deployments,rc nginx=nginx:1.9.1 --all

    # 将daemonset abc的所有容器镜像更新为“nginx：1.9.1”
    $ kubectl set image daemonset abc *=nginx:1.9.1

    # 从本地文件中更新nginx容器镜像
    $ kubectl set image -f path/to/file.yaml nginx=nginx:1.9.1 --local -o yaml

#### explain 命令：用于显示资源文档信息
    kubectl explain rs

#### edit 命令: 用于编辑资源信息
    # 编辑Deployment nginx的一些信息
    $ kubectl edit deployment nginx

    # 编辑service类型的nginx的一些信息
    $ kubectl edit service/nginx

#### logs 命令：输出pod中一个容器的日志

    #查看指定pod日志
    kubectl logs <pod_name>

    #类似tail -f的方式查看
    kubectl logs -f <pod_name>


#### describe 命令：输出指定资源的详细信息

    # 显示node的详细信息
    kubectl describe nodes
    kubectl describe node <node-name>

    # 显示pod的详细信息
    kubectl describe pods
    kubectl describe pod <pod-name>


#### 进入容器内部

    kubectl -n <命名空间> exec -it <pod-name> sh

#### 创建/删除的命名空间

    # 创建
    kubectl create namespace <namespace>
    # 删除
    kubectl delete namespace <namespace>


#### deployment重启

    # 方式1
    kubectl scale deployment {your_deployment_name} --replicas=0 -n {namespace}
    kubectl scale deployment {your_deployment_name} --replicas=1 -n {namespace}

    # 方式2
    kubectl rollout restart {your_deployment_name}




#### pod重启

    kubectl get pod <pod_name> -n <命名空间> -o yaml | kubectl replace --force -f -
