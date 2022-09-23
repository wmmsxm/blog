---
title: "Tidb探究 扩容缩容"
date: 2022-09-23T10:03:35+08:00
draft: false
tags: ["TiDB"]
categories: ["TiDB"]
---
# <center>Tidb探究 扩容缩容</center>

### 查看当前tidb配置

```
# 查看tidb信息
tiup cluster list

# 查看cluster的配置
tiup cluster edit-config <cluster-name>

# 查看集群状态
tiup cluster display <cluster-name>
```

#### 扩容TiDB/PD/TiKV节点
添加节点ip地址为192.168.174.128，可按照如下步骤进行操作。
##### 1.编写扩容拓扑配置
> ⚠️**注意**
> * 默认情况下，可以不填写端口以及目录信息。但在单机多实例场景下，则需要分配不同的端口以及目录，如果有端口或目录冲突，会在部署或扩容时提醒。
> * 从 TiUP v1.0.0 开始，扩容配置会继承原集群配置的 global 部分。

创建scale-out.yaml文件添加扩容拓扑配置
```
vi scale-out.yaml
```
TiDB配置文件参考：
```
tidb_servers:
  - host: 192.168.174.128
    ssh_port: 22
    port: 4000
    status_port: 10080
    # 文件存储地址，可参考edit-config
    deploy_dir: /tidb-deploy/tidb-4000
    log_dir: /tidb-deploy/tidb-4000/log
```
TiKV配置文件参考：
```
tikv_servers:
  - host: 192.168.174.128
    ssh_port: 22
    port: 20160
    status_port: 20180
    # 文件存储地址，可参考edit-config
    deploy_dir: /tidb-deploy/tikv-20160
    data_dir: /tidb-data/tikv-20160
    log_dir: /tidb-deploy/tikv-20160/log
    config:
      server.labels:
        host: 192.168.174.128
```
PD配置文件参考：
```
pd_servers:
  - host: 192.168.174.128
    ssh_port: 22
    name: pd-192.168.174.128-2379
    client_port: 2379
    peer_port: 2380
    # 文件存储地址，可参考edit-config
    deploy_dir: /tidb-deploy/pd-2379
    data_dir: /tidb-data/pd-2379
    log_dir: /tidb-deploy/pd-2379/log
```
##### 2.执行扩容命令 
执行`scale-out`命令前，先使用`check`及`check --apply`命令，检查和自动修复集群存在的潜在风险：
```
1.检查集群存在的潜在风险
tiup cluster check <cluster-name> scale-out.yaml --cluster --user tidb [-p] [-i /home/root/.ssh/gcp_rsa]

2.自动修复集群存在的潜在风险
tiup cluster check <cluster-name> scale-out.yaml --cluster --apply --user tidb [-p] [-i /home/root/.ssh/gcp_rsa]

3.执行`scale-out`命令扩容TiDB集群
tiup cluster scale-out <cluster-name> scale-out.yaml [-p] [-i /home/root/.ssh/gcp_rsa]

```
* 扩容配置文件为 scale-out.yaml。
* --user tidb 表示通过 tidb 用户登录到目标主机完成集群部署，该用户需要有 ssh 到目标机器的权限，并且在目标机器有 sudo 权限。也可以用其他有 ssh 和 sudo 权限的用户完成部署。
* [-i] 及 [-p] 为可选项，如果已经配置免密登录目标机，则不需填写。否则选择其一即可，[-i] 为可登录到目标机的 root 用户（或 --user 指定的其他用户）的私钥，也可使用 [-p] 交互式输入该用户的密码。  

预期日志结尾输出 Scaled cluster `<cluster-name>` out successfully 信息，表示扩容操作成功。

#### 缩容TiDB/PD/TiKV节点
添加节点ip地址为192.168.174.128，可按照如下步骤进行操作。  
> ⚠️ **注意**  
> * 移除 TiDB、PD 节点和移除 TiKV 节点的步骤类似。
> * 由于 TiKV、TiFlash 和 TiDB Binlog 组件是异步下线的，且下线过程耗时较长，所以 TiUP 对 TiKV、TiFlash 和 TiDB Binlog 组件做了特殊处理，详情参考[下线特殊处理](https://docs.pingcap.com/zh/tidb/stable/tiup-component-cluster-scale-in#%E4%B8%8B%E7%BA%BF%E7%89%B9%E6%AE%8A%E5%A4%84%E7%90%86)。

> ⚠️ **注意**  
> TiKV 中的 PD Client 会缓存 PD 节点的列表。当前版本的 TiKV 有定期自动更新 PD 节点的机制，可以降低 TiKV 缓存的 PD 节点列表过旧这一问题出现的概率。但你应尽量避免在扩容新 PD 后直接一次性缩容所有扩容前就已经存在的 PD 节点。如果需要，请确保在下线所有之前存在的 PD 节点前将 PD 的 leader 切换至新扩容的 PD 节点。

##### 1.查看节点ID信息
```
tiup cluster display <cluster-name>
```

```
Starting /root/.tiup/components/cluster/v1.10.0/cluster display <cluster-name>

TiDB Cluster: <cluster-name>

TiDB Version: v6.1.0

ID       Role         Host    Ports                            Status  Data Dir        Deploy Dir

--       ----         ----      -----                            ------  --------        ----------

10.0.1.3:8300  cdc          10.0.1.3    8300                            Up      data/cdc-8300      deploy/cdc-8300

10.0.1.4:8300  cdc          10.0.1.4    8300                            Up      data/cdc-8300      deploy/cdc-8300

10.0.1.4:2379  pd           10.0.1.4    2379/2380                        Healthy data/pd-2379      deploy/pd-2379

10.0.1.1:20160 tikv         10.0.1.1    20160/20180                      Up      data/tikv-20160     deploy/tikv-20160

10.0.1.2:20160 tikv         10.0.1.2    20160/20180                      Up      data/tikv-20160     deploy/tikv-20160

10.0.1.5:20160 tikv        10.0.1.5    20160/20180                     Up      data/tikv-20160     deploy/tikv-20160

10.0.1.3:4000  tidb        10.0.1.3    4000/10080                      Up      -                 deploy/tidb-4000

10.0.1.4:4000  tidb        10.0.1.4    4000/10080                      Up      -                 deploy/tidb-4000

10.0.1.5:4000  tidb         10.0.1.5    4000/10080                       Up      -            deploy/tidb-4000

10.0.1.3:9000   tiflash      10.0.1.3    9000/8123/3930/20170/20292/8234  Up      data/tiflash-9000       deploy/tiflash-9000

10.0.1.4:9000   tiflash      10.0.1.4    9000/8123/3930/20170/20292/8234  Up      data/tiflash-9000       deploy/tiflash-9000

10.0.1.5:9090  prometheus   10.0.1.5    9090                             Up      data/prometheus-9090  deploy/prometheus-9090

10.0.1.5:3000  grafana      10.0.1.5    3000                             Up      -            deploy/grafana-3000

10.0.1.5:9093  alertmanager 10.0.1.5    9093/9094                        Up      data/alertmanager-9093 deploy/alertmanager-9093
```

##### 2.执行缩容操作  
```
tiup cluster scale-in <cluster-name> --node 10.0.1.5:20160


# tidb 端口是 4000
# tikv 端口是 20160
# pc 端口是 2379
```
其中 --node 参数为需要下线节点的 ID。  
预期输出 Scaled cluster `<cluster-name>` in successfully 信息，表示缩容操作成功。  
下线需要一定时间，下线节点的状态变为 Tombstone 就说明下线成功。