---
title: "Docker学习(五) 安装Jenkins实现自动化部署" 
date: 2019-12-11T09:41:13+08:00
tags: ["docker"]
categories: ["docker", "learn"]
draft: false
---
## <center>Docker学习(五) 安装Jenkins实现自动化部署项目</center>
### 安装Jenkins
玄襄这里为了方便省事，直接使用docker安装Jenkins。

    # 下载镜像
    docker pull jenkins/jenkins:latest
![下载镜像](/images/jenkins/jenkins1-1.png)

    # 准备宿主挂载的文件夹
    mkdir /docker
    mkdir /docker/jenkins

    # 此处注意 容器需要宿主挂载文件夹的权限  不设置的话  挂载启动会失败
    chown -R 1000 /docker/jenkins
  ![准备](/images/jenkins/jenkins1-2.png)  
  ![权限](/images/jenkins/jenkins1-2-1.png)

    # 启动容器
    docker run -d --name=jenkins --privileged=true -p 8081:8080 -p 50000:50000 -v /docker/jenkins:/var/jenkins_home jenkins/jenkins

    命令解析：
      -p 8081:8080  指定宿主端口8081映射容器8080
      -p 50000:50000 主站进行通信
      -d 后台运行容器
      -v /docker/jenkins:/var/jenkins_home 宿主文件映射容器文件
      --privileged=true  开启特权模式
  ![启动](/images/jenkins/jenkins1-3.png)

### 初始化Jenkins
jenkins容器已经成功启动了，下面就是初始化jenkins。访问地址 是宿主ip + 8081端口即可访问。
1. 打开网站 需要登录  
![登录](/images/jenkins/jenkins1-4.png)
2. 密码可以根据网页提示找到文件夹，要注意这里应该是查看宿主文件夹   
 输入命令  vi /docker/jenkins/secrets/initialAdminPassword   复制密码即可  
 ![密码](/images/jenkins/jenkins1-5.png)

3. 登录成功之后，选择安装插件，玄襄这个直接选择推荐的。耐心等待安装完成  
![插件](/images/jenkins/jenkins1-6.png)

4. 创建第一个admin用户，也可以直接跳过，继续使用admin进行登录，玄襄这里直接跳过了  
![admin](/images/jenkins/jenkins1-7.png)

5. 访问地址修改，玄襄这里是直接在后面添加 jenkins  此处强烈建议(未来的我)  不要设置jenkins，最好不改url
![url](/images/jenkins/jenkins1-8.png)

6. 初始化基本完成，登录系统之后修改admin账号  
![初始化完成](/images/jenkins/jenkins1-9.png)  
![修改admin密码](/images/jenkins/jenkins1-10.png)

### 添加item项目并运行
#### 安装插件
在运行项目之前，jenkins需要安装相应的插件。在jenkins页面 → 系统管理  →  管理插件 →  可选插件 中搜索  
插件如下：   
    
      Maven Integration       用来支持构建maven项目
      Publish Over SSH        用来把构建好的部署包传送到指定服务器的指定位置
      Git安装
   ![插件](/images/jenkins/jenkins2-2.png)   
    ![插件](/images/jenkins/jenkins2-3.png)  


### 配置Jenkins   
jenkins需要配置jdk环境，根据个人项目需求 还涉及到 maven git等配置。这里需要解释下，因为jenkins是运行
在docker上，所以配置要安装宿主挂碍的文件夹/docker/jenkins下。  

1. 创建全局文件夹  执行命令 mkdir /docker/jenkins/globalEnv   
2. 因为玄襄的centos7没有wget， 所以需要下载   yum y install wget  

#### maven安装
maven选择安装3.6.3的，下载安装完之后  需要设置settings的镜像为阿里镜像，添加路径。具体请跟着玄襄一步步执行：  

    # 下载maven
    wget  http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz

    # 解压文件
    tar -zvxf apache-maven-3.6.3-bin.tar.gz

    # 移动文件夹到全局
    mv /apache-maven-3.6.3 /docker/jenkins/globalEnv

    # 添加阿里镜像
    vi /docker/jenkins/globalEnv/apache-maven-3.6.3/conf/settings.xml
      添加下面命令
    ...
      <mirror>  
          <id>nexus-aliyun</id>  
          <mirrorOf>central</mirrorOf>    
          <name>Nexus aliyun</name>  
          <url>http://maven.aliyun.com/nexus/content/groups/public</url>  
      </mirror>
    ....

    # 添加到路径
    vi /etc/profile 
      添加下面命令
      M2_HOME=/docker/jenkins/globalEnv/apache-maven-3.6.0   
      export PATH=$M2_HOME/bin:$PATH
    
    # 更新配置文件 
    source /etc/profile


#### jdk安装
jdk安装选择是1.8，因为jdk在官网下载需要登录，所以玄襄直接通过浏览器下载好之后，再上传到虚拟机中的。
[jdk下载页面](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 选择相应版本下载即可
![jdk](/images/jenkins/jenkins2-4.png)  

    # 解压文件
    tar -zvxf jdk-8u231-linux-x64.tar.gz
    #移动文件
    mv /jdk-1.8.0_231 /docker/jenkins/globalEnv

    # 添加到路径
    vim /etc/profile 
    export JAVA_HOME=/docker/jenkins/globalEnv/jdk1.8.0_231
    export PATH=$JAVA_HOME/bin:$PATH
    export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH

    # 更新配置文件 
    source /etc/profile
![安装完毕](/images/jenkins/jenkins2-5.png)


#### 安装git
git安装比较简单，执行下面的命令即可

    yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel asciidoc
    yum install gcc perl-ExtUtils-MakeMaker


#### jenkins全局环境配置
1. 首先进行jenkins的系统配置，系统管理  → 系统配置  → 添加Publish over SSH 可参照下图：
![系统配置](/images/jenkins/jenkins2-8.png)  
![ssh](/images/jenkins/jenkins2-9.png)

2. 配置全局工具， 系统管理→ 全局工具配置 → maven配置   → jdk安装  → git配置  → maven安装，具体步骤请看下图：  
![全局工具配置](/images/jenkins/jenkins2-6.png)  
![配置添加](/images/jenkins/jenkins2-7.png)   
![继续配置添加](/images/jenkins/jenkins2-7-1.png) 


#### 远程部署项目
1. 在首页点击“ 新建任务 ” 创建新项目，在弹出框中 输入项目名称，选择构建项目类型，玄襄构建的是maven项目，
然后点击 “ 确定 ”。如下图：  
![创建任务](/images/jenkins/jenkins3-1.png)    
![项目](/images/jenkins/jenkins3-2.png)  
2. 进入项目配置页面，添加源码管理git仓库，输入git地址  选择git仓库的账户密码， 没有的话，添加即可。
然后记得选择分支，玄襄这里执行的是<font color="red">develop</font>分支  
![git](/images/jenkins/jenkins3-3.png)  
![git1](/images/jenkins/jenkins3-3-1.png)  
3. 在Build这里的 Goals and options  添加命令   package compile -U  
![build](/images/jenkins/jenkins3-4.png)
4. 注意这里！！！ 这里是最后一步，也是最重要的一步。在build完成之后，要连接远程服务，将编译好的jar包发送过去，
然后执行远程服务的启动项目命令。  
首先选择远程服务 →  设置传输的员文件  →  移除前缀  →  远程服务的文件夹  → 执行命令   
![远程](/images/jenkins/jenkins3-5.png)  

```
# 切到目录
cd /home/finance
# 显示工作目录    --- 方便查看
pwd
# 检查该项目是否运行， 是则杀掉
ps -ef | grep yobtc-finance-service| grep -v grep | awk '{print $2}' | xargs --no-run-if-empty kill -9
# 休眠10s
sleep 10
# 使环境变量立即生效
source /etc/profile
# 执行项目setsid  实现后台运行
setsid nohup java -jar  -Xmx512m -Xms512m yobtc-finance-service.jar  > yobtc-finance-service.log &
```
5. 保存配置之后，直接点击构建即可。