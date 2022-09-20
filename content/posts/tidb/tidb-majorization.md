---
title: "Tidb探究 Tidb性能调优"
date: 2022-09-15T15:00:14+08:00
tags: ["TiDB"]
categories: ["TiDB"]
draft: false
---
# <center>Tidb探究 Tidb性能调优</center>

#### 设置 scan 操作的并发度,默认是15
通过navicat连接数据库，使用命令列界面执行下面命令：

```
# 查询session的当前值
select @@tidb_distsql_scan_concurrency;
# 查询global的当前值
select @@global.tidb_distsql_scan_concurrency;
 
# 全局
set global tidb_distsql_scan_concurrency=30;
# session
set session tidb_distsql_scan_concurrency=30;
```

#### 索引添加 

```
# 查询索引
show index from tableName

# 添加索引
CREATE INDEX indeKey ON tableName (字段1, 字段2);
```
单条sql只能使用一个索引，开始索引合并测试不生效。需要后续测试  

使用索引时，需要注意一下情况无法使用索引：
1. 避免总是 SELECT * 查询所有列的语句
2. 查询条件使用 !=，NOT IN 时，无法使用索引。
3. 使用 LIKE 时如果条件是以通配符 % 开头，也无法使用索引。
4. 查询条件使用 IN 表达式时，后面匹配的条件数量建议不要超过 300 个，否则执行效率会较差。


#### 使用TiFlash

```
# count 表示副本数，0 表示删除。
ALTER TABLE table_name SET TIFLASH REPLICA count;


# 查看表同步进度
# AVAILABLE 字段表示该表的 TiFlash 副本是否可用。1 代表可用，0 代表不可用。副本
# PROGRESS 字段代表同步进度，在 0.0~1.0 之间，1 代表至少 1 个副本已经完成同步
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = '<db_name>' and TABLE_NAME = '<table_name>';
```