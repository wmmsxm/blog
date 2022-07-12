---
title: "umi搭建dva"
date: 2019-10-22T18:00:35+08:00
tags: ["react", "umi"]
categories: ["react"]
draft: false
---

### umi搭建dva

    使用umiJs搭建后台管理页面，集成dav、ant，一些react需要的插件不需要手动添加，按需加载即可。  
    此处记录安装流程和走过的坑！！

#### 安装步骤

   一、yarn安装  
        国内推荐使用yarn，可以直接在[yarn官网](https://yarn.bootcss.com/docs/install/#windows-stable "yarn官网")下载安装。 不过本人没有成功 (*^▽^*)
        或者使用npm install -g yarn 命令进行安装。
    

   二、下载create-umi
	
   	两种方式  
        1、yarn create umi  
        2、yarn global add create-umi --prefix "D:\Program Files\nodejs\node_global_modules"  

        下载完的create-umi执行失败，系统无法识别，如下所示
![错误](/blog/images/react/20191023141701.png)
        
  	这是因为create-umi.cmd里面的盘符前有符号转义，内容如下：
	@"%~dp0\C:\Users\wmm\AppData\Local\Yarn\Data\global\node_modules\.bin\create-umi.cmd"   %*

    修改为：
    @"C:\Users\wmm\AppData\Local\Yarn\Data\global\node_modules\.bin\create-umi.cmd"   %*

    保存后，重新运行一下 "create-umi"命令，就可以正常运行了。
    

   三、安装

	1、选择project
		? Select the boilerplate type (Use arrow keys)
		  ant-design-pro  - Create project with an layout-only ant-design-pro boilerplate, use together with umi block.
		> app             - Create project with a simple boilerplate, support typescript.
		  block           - Create a umi block.
		  library         - Create a library with umi.
		  plugin          - Create a umi plugin.
		本人选择了app
		
	2、是否使用TypeScript
		? Do you want to use typescript? (y/N)
		y

	3、选择需要的插件功能
		? What functionality do you want to enable? (Press <space> to select, <a> to toggle all, <i> to invert selection)
		>◯ antd
		 ◯ dva
		 ◯ code splitting
		 ◯ dll
		此处本人全选

	4、最后确定，并使用yarn 安装依赖
		顺利的话  在浏览器打开 [地址](http://localhost:8000 "地址") 可看到umi界面