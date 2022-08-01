---
title: "TiDB建库建用户及授权"
date: 2022-08-01T16:01:08+08:00
tags: ["TiDB"]
categories: ["TiDB"]
draft: false
---
# <center>TiDB建库建用户及授权</center>
## TiDB初始化账号
TiDB部署完之后，会生成一个密码，可以用来登录数据库并进行修改![图片](/blog/images/tidb/user_password1.png)。

### 密码验证长度修改
````
set global validate_password_length=3;
````

### root授权远程访问
````
ALTER USER 'root'@'%' IDENTIFIED BY '123456'; 
grant all privileges on *.* to 'root'@'%' identified by '123456';
````

### 创建用户
````
# localhost   密码123456
CREATE USER 'tidb'@'localhost' IDENTIFIED BY '123456'; 

# %   密码123456   可以指定ip
CREATE USER 'tidb'@'%' IDENTIFIED BY '123456'; 
```` 

### 授权
````
# tidb.* 指 数据库tidb; 全部授权使用 *.*
grant all privileges on tidb.* to 'tidb'@'localhost' identified by '123456';


grant all privileges on tidb.* to 'tidb'@'%' identified by '123456';

````