---
title: "elasticsearch 安装教程"
date: 2021-01-21T10:56:59+08:00
tags: ["elasticsearch"]
categories: ["elasticsearch"]
draft: false
---
### elasticsearch 安装教程

    elasticsearch安装使用版本是7.8.1, 鉴于封装rest_client的版本最高支持8，向下兼容的； 8之后api接口路径调整,可以选择6,7，考虑到项目稳定和技术支持这里选择版本7.

为了方便下载，可以点击[这里](https://pan.baidu.com/s/1auq9JPJmT3x_yQD0TANgsw)进行下载； 提取码：c5gy  
废话不多说开始进入安装步骤：


前言：  
elasticsearch安装需要配置java环境，篇幅有限，就不介绍怎么安装java了，只需要配置JAVA_HOME环境变量即可。<font color="#dd0000">注意必须是java8以上的版本</font>


1、将下载的文件解压，将里面的elasticsearch-7.8.1文件夹放到相应目录， 注意目录不能有空格例如<font color="#dd0000">d:/Program Files/</font>  ![文件](/blog/images/elasticsearch/esset-1.png)

2、进入elasticsearch/bin目录，双击执行elasticsearch.bat![命令](/blog/images/elasticsearch/esset-2.png)
执行完毕之后，打开浏览器，输入 http://localhost:9200 ，显式以下画面，说明ES安装成功。
![命令](/blog/images/elasticsearch/esset-3.png)


### 安装elasticsearch服务
因为上述方法在关闭命令窗口后，elasticsearch会退出关闭，所以安装服务进行启动更合理  
1、修改elasticsearch-env.bat文件，将内容
```
if "%JAVA_HOME%" == "" (
  set JAVA="%ES_HOME%\jdk\bin\java.exe"
  set JAVA_HOME="%ES_HOME%\jdk"
  set JAVA_TYPE=bundled jdk
) else (
  set JAVA="%JAVA_HOME%\bin\java.exe"
  set JAVA_TYPE=JAVA_HOME
)
```
替换成
``` 
set JAVA="%ES_HOME%\jdk\bin\java.exe"
set JAVA_HOME="%ES_HOME%\jdk"
set JAVA_TYPE=bundled jdk
```

1、打开DOS命令行界面，切换到elasticSearch的bin目录，执行以下命令：  
```.\elasticsearch-service.bat install ```  ![命令](/blog/images/elasticsearch/esset-4.png)

2、打开任务管理器，选择服务,输入e筛选elasticsearch服务![服务](/blog/images/elasticsearch/esset-5.png), 右键启动即可


