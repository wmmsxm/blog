---
title: "Tidb探究 Mysql迁入TiDB"
date: 2022-08-01T17:32:01+08:00
tags: ["TiDB"]
categories: ["TiDB"]
draft: false
---
# <center>Tidb探究 Mysql迁入TiDB</center>

TiDB是一个分布式关系型数据库，可以无缝对接 MySQL。考虑到产品数据量大的情况下，单机MySQL可能无法支撑，而无缝切换到TiDB集群也比较方便。  
使用Dumpling工具导出MySQL数据库数据，并使用TiDB Lightning将数据迁移到TiDB集群的流程。

### 1、迁移工具下载
TiDB-community-toolkit软件包可以在[官网](https://pingcap.com/zh/product-community)下载。将下载的tidb-community-toolkit-v6.1.0-linux-amd64.tar.gz放在中控机master的tidb账号下的/home/tidb文件目录。  

````
# 解压文件
tar -xzf tidb-toolkit-v4.0.0-linux-amd64.tar.gz
cd ./tidb-toolkit-v4.0.0-linux-amd64
tar -xzf ./dumpling-v6.1.0-linux-amd64.tar.gz 
````
进入该文件夹，可以看到对应工具![工具](/blog/images/tidb/toolkit1.png)

### 2、MySQL数据备份
Dumpling和tidb-lightning就是一对导出、导入工具，其中Dumpling跟MySQL的mysqldump功能是一样的。  
建议使用Dumpling，原因是用它导出的数据时，会自动创建 xxx-schema-create.sql建库文件，而且建表和插入SQL文件分开，不用额外操作，配套用tidb-lightning执行导入，不容易出错。  
在工具文件夹下，用Dumpling工具连接目标数据库导出，命令如下：
````
./dumpling -h 172.16.1.71 -P 30306 -u root -p 123456 -t 16 -F256MiB -B ry-cloud --filetype sql  -o /tidb-data/mydumpersql/ -r 200000 
# 执行时间过程，请耐心等待
````
参数说明：
主要选项|用途
:--:|:--:
-V 或 --version|输出 Dumpling 版本并直接退出
-B 或 --database|导出指定的数据库
-T 或 --tables-list|导出指定数据表
-f 或 --filter|导出能匹配模式的表
--case-sensitive|table-filter 是否大小写敏感
-t 或 --threads|备份执行的线程数，默认4个线程
-r 或 --rows|将 table 划分成 row 行数据，一般针对大表操作并发生成多个文件
-L 或 --logfile|日志输出地址，为空时会输出到控制台
-F 或 --filesize|将 table 数据划分出来的文件大小，需指明单位（如 128B, 64KiB, 32MiB, 1.5GiB）
-o 或 --output|导出本地文件路径或外部存储 URL
--filetype|导出文件类型（csv/sql）
-p 或 --password|连接的数据库主机的密码
-P 或 --port|连接的数据库主机的端口
  
注意：最后一个 outputdir 的值，后面导入的时候需要使用。


### 3、tidb-lightning导入数据
1. 进入tidb-toolkit-v4.0.0-linux-amd64文件夹，解压TiDB Lightning安装包
````
tar -xzf ./tidb-lightning-v6.1.0-linux-amd64.tar.gz
````
2. 配置 tidb-lightning.toml
在/home/tidb/tidb-toolkit-v4.0.0-linux-amd64文件夹下添加文件tidb-lightning.toml
````
[lightning]
# 日志
level = "info"
file = "tidb-lightning.log"

[tikv-importer]
# 选择使用的导入模式
backend = "local"
# 设置排序的键值对的临时存放地址，目标路径需要是一个空目录，不存在就新建文件夹  
sorted-kv-dir = "/mnt/ssd/sorted-kv-dir"

[mydumper]
# 源数据目录。
data-source-dir = "/tidb-data/mydumpersql"

# 配置通配符规则，默认规则会过滤 mysql、sys、INFORMATION_SCHEMA、PERFORMANCE_SCHEMA、METRICS_SCHEMA、INSPECTION_SCHEMA 系统数据库下的所有表
# 若不配置该项，导入系统表时会出现“找不到 schema”的异常
filter = ['*.*', '!mysql.*', '!sys.*', '!INFORMATION_SCHEMA.*', '!PERFORMANCE_SCHEMA.*', '!METRICS_SCHEMA.*', '!INSPECTION_SCHEMA.*']
[tidb]
# 目标集群的信息
host = "172.16.1.73"
port = 4000
user = "root"
password = "123456"
# 表架构信息在从 TiDB 的“状态端口”获取。
status-port = 10080
# 集群 pd 的地址
pd-addr = "172.16.1.73:2379"
````
3. 开始还原
````
# 在/home/tidb/tidb-toolkit-v4.0.0-linux-amd64目录下执行下面命令：
./tidb-lightning -config tidb-lightning.toml
````
等待一段时间后，输出如下信息即还原完成。如有问题，可在当前目录下查看tidb-lightning.log
![输出](/blog/images/tidb/toolkit2.png)