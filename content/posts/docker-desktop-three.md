---
title: "Docker_Desktop学习(三) Kubernetes-Dashboard安装"
date: 2021-12-04T13:35:05+08:00
tags: ["docker-desktop"]
categories: ["docker","docker-desktop"]
draft: false
---
## <center>Docker_Desktop学习(三) Kubernetes-Dashboard安装</center>
### 创建kubernetes-dashboard服务和对应的pod  
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml
如果发现链接失效，可以访问[https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard),查找最新的链接。  
或者将yaml内容存到本地，使用下面命令：    
```
kubectl create -f kubernetes-dashboard.yaml
```

![dd3-1](/blog/images/docker_desktop/dd3_1.png)
### 检查kubernetes-dashboard应用状态
    kubectl get pod -n kubernetes-dashboard  
![dd3-3](/blog/images/docker_desktop/dd3_3.png)
  

### 使用代理访问dashboard  
使用代理命令`kubectl proxy`， 默认端口是8001，可以使用 `-p xxx`指定端口，比如`kubectl proxy -p 8112`,  
![dd3-2](/blog/images/docker_desktop/dd3_2.png)  
然后在通过[http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login)访问dashboard  

### token访问
首次访问时会阻拦，继续访问需要选择验证方式，会有kubeconfig和令牌两种方式，我们选择令牌token。  
```
kubectl -n kube-system describe secret default | awk '$1=="token:"{print $2}'  
```
执行此命令获取token填入即可正常访问。
