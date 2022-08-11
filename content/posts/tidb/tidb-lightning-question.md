---
title: "Tidb Lightning Question"
date: 2022-08-11T15:11:07+08:00
draft: false
---
# <center>Tidb探究 迁移后问题</center>
#### 1、The isolation level 'SERIALIZABLE' is not supported. Set tidb\_skip\_isolation\_level\_check=1 to skip this error

通过navicat连接数据库，使用命令列界面执行下面命令：

```纯文本
set global tidb_skip_isolation_level_check=1;
```

TiDB 不支持 SERIALIZABLE 隔离级别，设置 tidb\_skip\_isolation\_level\_check 只是将报错信息变成了 warning ，实际上设置 SERIALIZABLE 还是不生效的。 &#x20;

设置了 tidb\_skip\_isolation\_level\_check=1 还是会报错是因为你是会话级别设置了这个参数，只对当前会话生效。要全局生效的话，可以 set global tidb\_skip\_isolation\_level\_check=1;

#### 2、group by不支持问题

TiDB 支持 SQL 模式 `ONLY_FULL_GROUP_BY`，当启用该模式时，TiDB 拒绝不明确的非聚合列的查询。目前，TiDB 默认开启 SQL 模式 `ONLY_FULL_GROUP_BY`。

通过navicat连接数据库，使用命令列界面执行下面命令：

```纯文本
# 更新全局  对全局作用域变量的更改将作用于集群中的其它服务器，并且重启后更改依然有效。
set @@global.sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
# 更新当前会话
set @@sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'; 

```

例如：

```纯文本
select a, b, sum(c) from t group by a;

```

会报下面的错误：

```纯文本
ERROR 1055 (42000): Expression #2 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'b' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

#### 3、tidb sql兼容适配max_allowed_packet
批量插入数量过多时，会出现下面得错误：
````
com.mysql.cj.jdbc.exceptions.PacketTooBigException: Packet for query is too large (70,199,179 > 67,108,864). You can change this value on the server by setting the ‘max_allowed_packet’ variable.
````
执行下面命令解决：
````
默认 67108864 (64 MB)
set @@global.max_allowed_packet=134217728;   （ 134217728 = 128M ）
````
*<font color=red size=3 >注意：这个虽然是持久化的，但是不会对已连接的会话产生影响，只对新连接的会话产生影响。所以需要重启相应得服务。</font>*