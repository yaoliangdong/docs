# TiDB 搭建生产集群
## 系统环境 `CentOS 8`
## TiDB `v7.5 LTS`
## HAProxy `2.6.2`
### 部署 TiDB 集群 
https://docs.pingcap.com/zh/tidb/stable/production-deployment-using-tiup

**软硬件环境需求及前置检查**
软硬件环境需求：https://docs.pingcap.com/zh/tidb/stable/hardware-and-software-requirements
环境与系统配置检查：https://docs.pingcap.com/zh/tidb/stable/check-before-deployment

**安装 TiUP 部署工具**
执行如下命令安装 TiUP 工具：

```
$ curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```
按如下步骤设置 TiUP 环境变量：
```
$ source .bash_profile
```
确认 TiUP 工具是否安装：
```
$ which tiup
```
```
/root/.tiup/bin/tiup
```
安装 TiUP cluster 组件：
```
$ tiup cluster
```
如果已经安装，则更新 TiUP cluster 组件至最新版本：
```
$ tiup update --self && tiup update cluster
```
```
预期输出 “Update successfully!” 字样。
```
验证当前 TiUP cluster 版本信息。执行如下命令查看 TiUP cluster 组件版本：
```
$ tiup --binary cluster
```
```
/root/.tiup/components/cluster/v1.14.0/tiup-cluster
```

**使用TiUP部署TiDB集群**
创建`topology.yaml`文件

```
# # Global variables are applied to all deployments and used as the default value of
# # the deployments if a specific deployment value is missing.
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/tidb-deploy"
  data_dir: "/tidb-data"

pd_servers:
  - host: 192.168.154.128

tidb_servers:
  - host: 192.168.154.128

tikv_servers:
  - host: 192.168.154.128

monitoring_servers:
  - host: 192.168.154.128

grafana_servers:
  - host: 192.168.154.128

alertmanager_servers:
  - host: 192.168.154.128
```
执行部署命令

检查集群存在的潜在风险：
```
$ tiup cluster check ./topology.yaml --user root [-p] [-i /home/root/.ssh/gcp_rsa]
```
自动修复集群存在的潜在风险：
```
$ tiup cluster check ./topology.yaml --apply --user root [-p] [-i /home/root/.ssh/gcp_rsa]
```
部署 TiDB 集群（`-p`密码验证方式。`-i`为免密方式部署，详见文档附录）：
```
$ tiup cluster deploy tidb-test v7.5.0 ./topology.yaml --user root [-p] [-i /home/root/.ssh/gcp_rsa]
```
查看 TiUP 管理的集群情况
```
$ tiup cluster list
```
检查部署的 TiDB 集群情况
```
$ tiup cluster display tidb-test
```
启动集群（安全启动）
```
$ tiup cluster start tidb-test --init
```
```
Started cluster `tidb-test` successfully.
The root password of TiDB database has been changed.
The new password is: 'y_+3Hwp=*AWz8971s6'.
Copy and record it to somewhere safe, it is only displayed once, and will not be stored.
The generated password can NOT be got again in future.
```
启动集群（普通启动）
```
# 使用普通启动方式后，可通过无密码的 root 用户登录数据库。
$ tiup cluster start tidb-test
```
验证集群运行状态
```
$ tiup cluster display tidb-test
```
```
tiup is checking updates for component cluster ...
A new version of cluster is available:
   The latest version:         v1.14.1
   Local installed version:    v1.14.0
   Update current component:   tiup update cluster
   Update all components:      tiup update --all

Starting component `cluster`: /root/.tiup/components/cluster/v1.14.0/tiup-cluster display tidb-test
Cluster type:       tidb
Cluster name:       tidb-test
Cluster version:    v7.5.0
Deploy user:        tidb
SSH type:           builtin
Dashboard URL:      http://192.168.154.128:2379/dashboard
Grafana URL:        http://192.168.154.128:3000
ID                     Role          Host             Ports        OS/Arch       Status   Data Dir                      Deploy Dir
--                     ----          ----             -----        -------       ------   --------                      ----------
192.168.154.128:9093   alertmanager  192.168.154.128  9093/9094    linux/x86_64  Up       /tidb-data/alertmanager-9093  /tidb-deploy/alertmanager-9093
192.168.154.128:3000   grafana       192.168.154.128  3000         linux/x86_64  Up       -                             /tidb-deploy/grafana-3000
192.168.154.128:2379   pd            192.168.154.128  2379/2380    linux/x86_64  Up|L|UI  /tidb-data/pd-2379            /tidb-deploy/pd-2379
192.168.154.128:9090   prometheus    192.168.154.128  9090/12020   linux/x86_64  Up       /tidb-data/prometheus-9090    /tidb-deploy/prometheus-9090
192.168.154.128:4100   tidb          192.168.154.128  4100/11080   linux/x86_64  Up       -                             /tidb-deploy/tidb-4100
192.168.154.128:20160  tikv          192.168.154.128  20160/20180  linux/x86_64  Up       /tidb-data/tikv-20160         /tidb-deploy/tikv-20160
192.168.154.129:20160  tikv          192.168.154.129  20160/20180  linux/x86_64  Up       /tidb-data/tikv-20160         /tidb-deploy/tikv-20160
Total nodes: 7
```
TiDB控制台：http://192.168.154.128:2379/dashboard
默认账户密码：
root
密码为空
Grafana控制台：http://192.168.154.128:3000
默认账户密码：
admin
admin

###  对集群进行扩容缩容
**使用 TiUP 扩容缩容 TiDB 集群**https://docs.pingcap.com/zh/tidb/stable/scale-tidb-using-tiup
扩容 TiDB/PD/TiKV 节点
扩容TiKV 示例：

```
$ vim scale-out-kv.yml
```
```
tikv_servers:
  - host: 192.168.154.129
    ssh_port: 22
    port: 20160
    status_port: 20180
    deploy_dir: /tidb-deploy/tikv-20160
    data_dir: /tidb-data/tikv-20160
    log_dir: /tidb-deploy/tikv-20160/log
```
检查集群存在的潜在风险：
```
$ tiup cluster check <cluster-name> scale-out-kv.yml --cluster --user root [-p] [-i /home/root/.ssh/gcp_rsa]
```
自动修复集群存在的潜在风险：
```
$ tiup cluster check <cluster-name> scale-out-kv.yml --cluster --apply --user root [-p] [-i /home/root/.ssh/gcp_rsa]
```
执行扩容命令
```
$ tiup cluster scale-out <cluster-name> scale-out-kv.yml [-p] [-i /home/root/.ssh/gcp_rsa]
```
检查集群状态
```
$ tiup cluster display <cluster-name>
```




###  数据容灾备份与恢复
**TiDB 快照备份**
https://docs.pingcap.com/zh/tidb/stable/br-snapshot-guide
安装br 工具
```
$ tiup install br
```
对集群进行快照备份
使用 br backup full 可以进行一次快照备份：https://docs.pingcap.com/zh/tidb/stable/br-snapshot-guide

```
$ tiup br backup full --pd "192.168.154.128:2379" \
    --backupts '2024-02-05 10:30:00' \
    --storage "s3://tidb-bk/240205/?endpoint=http://192.168.154.128:9000&access-key=nqzBZZhGBYp540MZCBap&secret-access-key=ZFEi9gLosVkprMNvrAxNLmgPF1Hknsok4zlIKF0q" \
    --ratelimit 128 \
```
```
iup is checking updates for component br ...
Starting component `br`: /root/.tiup/components/br/v7.6.0/br backup full --pd 192.168.154.128:2379 --backupts 2024-02-05 10:30:00 --storage s3://tidb-bk/240205/?endpoint=http://192.168.154.128:9000&access-key=nqzBZZhGBYp540MZCBap&secret-access-key=ZFEi9gLosVkprMNvrAxNLmgPF1Hknsok4zlIKF0q --ratelimit 128
Detail BR log in /tmp/br.log.2024-02-04T22.55.05-0500 
[2024/02/04 22:55:05.857 -05:00] [WARN] [backup.go:311] ["setting `--ratelimit` and `--concurrency` at the same time, ignoring `--concurrency`: `--ratelimit` forces sequential (i.e. concurrency = 1) backup"] [ratelimit=134.2MB/s] [concurrency-specified=4]
Full Backup <-----------------------------------------------------------------------------------> 100.00%
Checksum <--------------------------------------------------------------------------------------> 100.00%
[2024/02/04 22:55:12.429 -05:00] [INFO] [collector.go:77] ["Full Backup success summary"] [total-ranges=19] [ranges-succeed=19] [ranges-failed=0] [backup-checksum=168.863382ms] [backup-fast-checksum=58.320307ms] [backup-total-ranges=119] [total-take=6.578538393s] [total-kv=1398] [total-kv-size=408.8kB] [average-speed=62.15kB/s] [backup-data-size(after-compressed)=102.5kB] [Size=102451] [BackupTS=447518343168000000]

```
查询快照备份的时间点信息
```
$ tiup br validate decode --field="end-version" \
--storage "s3://tidb-bk/240205/?endpoint=http://192.168.154.128:9000&access-key=nqzBZZhGBYp540MZCBap&secret-access-key=ZFEi9gLosVkprMNvrAxNLmgPF1Hknsok4zlIKF0q" | tail -n1
```
```
> --storage "s3://tidb-bk/240205/?endpoint=http://192.168.154.128:9000&access-key=nqzBZZhGBYp540MZCBap&secret-access-key=ZFEi9gLosVkprMNvrAxNLmgPF1Hknsok4zlIKF0q" | tail -n1
tiup is checking updates for component br ...
Starting component `br`: /root/.tiup/components/br/v7.6.0/br validate decode --field=end-version --storage s3://tidb-bk/240205/?endpoint=http://192.168.154.128:9000&access-key=nqzBZZhGBYp540MZCBap&secret-access-key=ZFEi9gLosVkprMNvrAxNLmgPF1Hknsok4zlIKF0q
Detail BR log in /tmp/br.log.2024-02-04T22.58.43-0500 
447518343168000000
```
恢复快照备份数据
```
$ tiup br restore full --pd "192.168.154.128:2379" \
--storage "s3://tidb-bk/240205/?endpoint=http://192.168.154.128:9000&access-key=nqzBZZhGBYp540MZCBap&secret-access-key=ZFEi9gLosVkprMNvrAxNLmgPF1Hknsok4zlIKF0q" \
```
```
tiup is checking updates for component br ...
Starting component `br`: /root/.tiup/components/br/v7.6.0/br restore full --pd 192.168.154.128:2379 --storage s3://tidb-bk/240205/?endpoint=http://192.168.154.128:9000&access-key=nqzBZZhGBYp540MZCBap&secret-access-key=ZFEi9gLosVkprMNvrAxNLmgPF1Hknsok4zlIKF0q
Detail BR log in /tmp/br.log.2024-02-05T01.14.12-0500

# 受限于本机资源，没有恢复成功过  -> get timestamp too slow
# 详细日志如下：
[2024/02/05 01:34:45.556 -05:00] [WARN] [pd.go:156] ["get timestamp too slow"] ["cost time"=36.412937ms]
[2024/02/05 01:34:47.544 -05:00] [WARN] [pd.go:156] ["get timestamp too slow"] ["cost time"=32.370948ms]
[2024/02/05 01:36:33.559 -05:00] [WARN] [pd.go:156] ["get timestamp too slow"] ["cost time"=38.628138ms]
[2024/02/05 01:37:02.648 -05:00] [WARN] [pd.go:156] ["get timestamp too slow"] ["cost time"=138.171927ms]
[2024/02/05 01:37:37.551 -05:00] [WARN] [pd.go:156] ["get timestamp too slow"] ["cost time"=40.730184ms]
```
恢复备份数据中指定库表的数据
```
$ tiup br restore db --pd "${PD_IP}:2379" \
--db "test" \
--storage "s3://backup-101/snapshot-202209081330?access-key=${access-key}&secret-access-key=${secret-access-key}"
```
恢复单张表的数据
```
$ tiup br restore table --pd "${PD_IP}:2379" \
--db "test" \
--table "usertable" \
--storage "s3://backup-101/snapshot-202209081330?access-key=${access-key}&secret-access-key=${secret-access-key}"
```
使用表库过滤功能恢复部分数据
```
$ tiup br restore full --pd "${PD_IP}:2379" \
--filter 'db*.tbl*' \
--storage "s3://backup-101/snapshot-202209081330?access-key=${access-key}&secret-access-key=${secret-access-key}"
```
**TiDB 日志备份与 PITR**
https://docs.pingcap.com/zh/tidb/stable/br-pitr-guide
开启日志备份
执行 br log start 命令启动日志备份任务，一个集群只能启动一个日志备份任务
```
$ tiup br log start --task-name=pitr --pd "192.168.154.128:2379" \
--storage 's3://tidb-bk/240205-logbackup/?endpoint=http://192.168.154.128:9000&access-key=nqzBZZhGBYp540MZCBap&secret-access-key=ZFEi9gLosVkprMNvrAxNLmgPF1Hknsok4zlIKF0q'
```
```
tiup is checking updates for component br ...
Starting component `br`: /root/.tiup/components/br/v7.6.0/br log start --task-name=pitr --pd 192.168.154.128:2379 --storage s3://tidb-bk/240205-logbackup/?endpoint=http://192.168.154.128:9000&access-key=nqzBZZhGBYp540MZCBap&secret-access-key=ZFEi9gLosVkprMNvrAxNLmgPF1Hknsok4zlIKF0q
Detail BR log in /tmp/br.log.2024-02-05T02.03.28-0500 
[2024/02/05 02:03:31.923 -05:00] [INFO] [collector.go:77] ["log start"] [streamTaskInfo="{taskName=pitr,startTs=447510377085272066,endTS=999999999999999999,tableFilter=*.*}"] [pausing=false] [rangeCount=2]
[2024/02/05 02:03:35.376 -05:00] [INFO] [collector.go:77] ["log start success summary"] [total-ranges=0] [ranges-succeed=0] [ranges-failed=0] [backup-checksum=31.351165ms] [total-take=6.915016341s]
```
日志备份任务启动后，会在 TiDB 集群后台持续地运行，直到你手动将其暂停。在这过程中，TiDB 变更数据将以小批量的形式定期备份到指定存储中。如果你需要查询日志备份任务当前状态，执行如下命令：
```
$ tiup br log status --task-name=pitr --pd "192.168.154.128:2379"
```
```
tiup is checking updates for component br ...
Starting component `br`: /root/.tiup/components/br/v7.6.0/br log status --task-name=pitr --pd 192.168.154.128:2379
Detail BR log in /tmp/br.log.2024-02-05T02.06.06-0500 
● Total 1 Tasks.
> #1 <
              name: pitr
            status: ● NORMAL
             start: 2024-02-05 02:03:31.806 -0500
               end: 2090-11-18 09:07:45.624 -0500
           storage: s3://tidb-bk/240205-logbackup
       speed(est.): 0.00 ops/s
checkpoint[global]: 2024-02-05 02:03:31.806 -0500; gap=2m37s
```
进行 PITR
如果你想恢复到备份保留期内的任意时间点，可以使用 br restore point 命令。执行该命令时，你需要指定要恢复的时间点、恢复时间点之前最近的快照备份以及日志备份数据。br 命令行工具会自动判断和读取恢复需要的数据，然后将这些数据依次恢复到指定的集群
```
$ tiup br restore point --pd "${PD_IP}:2379" \
--storage='s3://backup-101/logbackup?access-key=${access-key}&secret-access-key=${secret-access-key}' \
--full-backup-storage='s3://backup-101/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}' \
--restored-ts '2022-05-15 18:00:00+0800'
```
**清理过期的日志备份数据**
* 查找备份保留期之外的最近一次全量备份。
*  使用 validate 指令获取该备份对应的时间点。假如需要清理 2022/09/01 之前的备份数据，则应查找该日期之前的最近一次全量备份，且保证它不会被清理。
```
$ FULL_BACKUP_TS=`tiup br validate decode --field="end-version" --storage "s3://backup-101/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}"| tail -n1`
```
* 清理该快照备份 FULL_BACKUP_TS 之前的日志备份数据。
```
$ tiup br log truncate --until=${FULL_BACKUP_TS} --storage='s3://backup-101/logbackup?access-key=${access-key}&secret-access-key=${secret-access-key}'
```
* 清理该快照备份 FULL_BACKUP_TS 之前的快照备份数据。
```
$ rm -rf s3://backup-101/snapshot-${date}
```

###  使用 HAProxy 对TiDB-server进行负载

**修改TiDB集群配置，使集群允许使用 PROXY 协议连接**
https://docs.pingcap.com/zh/tidb/stable/tidb-configuration-file#networks
关于配置修改参考运维操作
https://docs.pingcap.com/zh/tidb/stable/maintain-tidb-using-tiup
修改配置参数
networks配置参考 
配置文件方式：https://docs.pingcap.com/zh/tidb/stable/tidb-configuration-file#networks
命令行方式：https://docs.pingcap.com/zh/tidb/stable/command-line-flags-for-tidb-configuration
官文修改networks不明确。见社区
https://asktug.com/t/topic/1010268
https://asktug.com/t/topic/35838

允许使用 PROXY 协议连接 TiDB 的代理服务器地址列表。
默认值：""
通常情况下，通过反向代理使用 TiDB 时，TiDB 会将反向代理服务器的 IP 地址视为客户端 IP 地址。对于支持 PROXY 协议的反向代理（如 HAProxy），开启 PROXY 协议后能让反向代理透传客户端真实的 IP 地址给 TiDB。
配置该参数后，TiDB 将允许配置的源 IP 地址使用 PROXY 协议连接到 TiDB，且拒绝这些源 IP 地址使用非 PROXY 协议连接。若该参数为空，则任何源 IP 地址都不能使用 PROXY 协议连接到 TiDB。地址可以使用 IP 地址格式 (192.168.1.50) 或者 CIDR 格式 (192.168.1.0/24)，并可用 , 分隔多个地址，或用 * 代表所有 IP 地址。

```
$ tiup cluster edit-config tidb-test
```
server_configs 与 global 同级。（注：附录提供了一个带PROXY 协议的拓扑文件）
拓扑文件参考
https://docs.pingcap.com/zh/tidb/stable/tiup-cluster-topology-reference
https://docs.pingcap.com/zh/tidb/stable/tiup-cluster-topology-reference#server_configs

如果配置的生效范围为该组件全局，则配置到 server_configs
```
server_configs:
  tidb:
    proxy-protocol.networks: 10.30.234.156
```
执行 reload 命令滚动分发配置、重启相应组件
```
$ tiup cluster reload tidb-test
```
如果配置的生效范围为某个节点，则配置到具体节点的 config 中
```
tidb_servers:
- host: 10.0.1.11
  port: 4000
  config:
      log.slow-threshold: 300
```

官文说`*`可以代表所有源 IP 地址，实测试并不行，两个问题：
1、`*`须用引号引起，否则配置无法正常保存
2、即便配置生效了，Navicat连接 报错：1105 - Invalid PROXY Protocol Header

```
server_configs:
  tidb:
    proxy-protocol.networks: '*'
```
一个错误示例：
配置无法正常保存，错误如下：

```
server_configs:
  tidb:
    proxy-protocol.networks: *
```
```
iup is checking updates for component cluster ...
A new version of cluster is available:
   The latest version:         v1.14.1
   Local installed version:    v1.14.0
   Update current component:   tiup update cluster
   Update all components:      tiup update --all

Starting component `cluster`: /root/.tiup/components/cluster/v1.14.0/tiup-cluster edit-config tidb-test
New topology could not be saved: Failed to parse topology file: yaml: line 16: did not find expected alphabetic or numeric character
Do you want to continue editing? [Y/n]: (default=Y)
```


**安装 HAProxy**
https://docs.pingcap.com/zh/tidb/stable/haproxy-best-practices
安装相关依赖包
```
yum -y install epel-release gcc systemd-devel
yum -y install gcc automake autoconf libtool make
```
下载 HAProxy 2.6.2 的源码包：
```
$ wget https://www.haproxy.org/download/2.6/src/haproxy-2.6.2.tar.gz
```

解压源码包：
```
$ tar zxf haproxy-2.6.2.tar.gz
```
从源码编译 HAProxy 应用：
```
cd haproxy-2.6.2
make clean
make -j 8 TARGET=linux-glibc USE_THREAD=1
make PREFIX=/app/haproxy SBINDIR=/app/haproxy/bin install
```
重新配置 profile 文件：
```
$ echo 'export PATH=/app/haproxy/bin:$PATH' >> /etc/profile
. /etc/profile
```

检查 HAProxy 是否安装成功：
```
$ which haproxy
```
```
/app/haproxy/bin/haproxy
```

**配置 HAProxy**
创建HAProxy程序专门的用户与组

```
useradd haproxy
groupadd haproxy
usermod -aG haproxy haproxy
chown -R haproxy:haproxy /var/run
```
```
# 没有就创建目录结构与文件
$ cd /etc/haproxy
$ vim haproxy.cfg
```
```
global                                     # 全局配置。
   log         127.0.0.1 local2            # 定义全局的 syslog 服务器，最多可以定义两个。
   chroot      /var/lib/haproxy            # 更改当前目录并为启动进程设置超级用户权限，从而提高安全性。（软件工作目录）
   pidfile     /var/run/haproxy.pid        # 将 HAProxy 进程的 PID 写入 pidfile。（注：启动进程的用户必须有权限访问此文件 ）
   maxconn     4096                        # 单个 HAProxy 进程可接受的最大并发连接数，等价于命令行参数 "-n"。
   nbthread    48                          # 最大线程数。线程数的上限与 CPU 数量相同。
   user        haproxy                     # 同 UID 参数。
   group       haproxy                     # 同 GID 参数，建议使用专用用户组。
   daemon                                  # 让 HAProxy 以守护进程的方式工作于后台，等同于命令行参数“-D”的功能。当然，也可以在命令行中用“-db”参数将其禁用。
   stats socket /var/lib/haproxy/stats     # 统计信息保存位置。

defaults                                   # 默认配置。
   log global                              # 日志继承全局配置段的设置。
   retries 2                               # 向上游服务器尝试连接的最大次数，超过此值便认为后端服务器不可用。
   timeout connect  2s                     # HAProxy 与后端服务器连接超时时间。如果在同一个局域网内，可设置成较短的时间。
   timeout client 30000s                   # 客户端与 HAProxy 连接后，数据传输完毕，即非活动连接的超时时间。
   timeout server 30000s                   # 服务器端非活动连接的超时时间。

listen admin_stats                         # frontend 和 backend 的组合体，此监控组的名称可按需进行自定义。
   bind 0.0.0.0:8080                       # 监听端口。
   mode http                               # 监控运行的模式，此处为 `http` 模式。
   option httplog                          # 开始启用记录 HTTP 请求的日志功能。
   maxconn 10                              # 最大并发连接数。
   stats refresh 30s                       # 每隔 30 秒自动刷新监控页面。
   stats uri /haproxy                      # 监控页面的 URL。
   stats realm HAProxy                     # 监控页面的提示信息。
   stats auth admin:pingcap123             # 监控页面的用户和密码，可设置多个用户名。
   stats hide-version                      # 隐藏监控页面上的 HAProxy 版本信息。
   stats  admin if TRUE                    # 手工启用或禁用后端服务器（HAProxy 1.4.9 及之后版本开始支持）。

listen tidb-cluster                        # 配置 database 负载均衡。
   bind 0.0.0.0:3390                       # 浮动 IP 和 监听端口。
   mode tcp                                # HAProxy 要使用第 4 层的传输层。
   balance leastconn                       # 连接数最少的服务器优先接收连接。`leastconn` 建议用于长会话服务，例如 LDAP、SQL、TSE 等，而不是短会话协议，如 HTTP。该算法是动态的，对于启动慢的服务器，服务器权重会在运行中作调整。
   server tidb-1 192.168.154.128:4100 check inter 2000 rise 2 fall 3       # 检测 4000 端口，检测频率为每 2000 毫秒一次。如果 2 次检测为成功，则认为服务器可用；如果 3 次检测为失败，则认为服务器不可用。
   # server tidb-2 10.9.39.208:4000 check inter 2000 rise 2 fall 3
   # server tidb-3 10.9.64.166:4000 check inter 2000 rise 2 fall 3

```

启动 HAProxy
```
$ haproxy -f /etc/haproxy/haproxy.cfg
```
```
# 没有任何输出代表成功。ps检查进程
```
HAProxy 控制台
http://192.168.154.128:8080/haproxy
账户密码`haproxy.cfg`文件中配置
admin
pingcap123

通过HAProxy代理访问TiDB集群的地址
jdbc url：192.168.154.128:3390

**验证负载组件**

查看集群状态：配置了两个TiDB role，分别为 192.168.154.129:4000和192.168.154.130:4000，如下：（注：附录提供了一个带PROXY 协议的拓扑文件，两个TiDB节点，请用该文件部署集群）
```
$ tiup cluster display tidb-test
```
```
tiup is checking updates for component cluster ...
A new version of cluster is available:
   The latest version:         v1.14.1
   Local installed version:    v1.14.0
   Update current component:   tiup update cluster
   Update all components:      tiup update --all

Starting component `cluster`: /root/.tiup/components/cluster/v1.14.0/tiup-cluster display tidb-test
Cluster type:       tidb
Cluster name:       tidb-test
Cluster version:    v7.5.0
Deploy user:        tidb
SSH type:           builtin
Dashboard URL:      http://192.168.154.128:2379/dashboard
Grafana URL:        http://192.168.154.128:3000
ID                     Role          Host             Ports        OS/Arch       Status  Data Dir                       Deploy Dir
--                     ----          ----             -----        -------       ------  --------                       ----------
192.168.154.128:9093   alertmanager  192.168.154.128  9093/9094    linux/x86_64  Up      /tidb-data2/alertmanager-9093  /tidb-deploy2/alertmanager-9093
192.168.154.128:3000   grafana       192.168.154.128  3000         linux/x86_64  Up      -                              /tidb-deploy2/grafana-3000
192.168.154.128:2379   pd            192.168.154.128  2379/2380    linux/x86_64  Up|UI   /tidb-data2/pd-2379            /tidb-deploy2/pd-2379
192.168.154.129:2379   pd            192.168.154.129  2379/2380    linux/x86_64  Up|L    /tidb-data2/pd-2379            /tidb-deploy2/pd-2379
192.168.154.130:2379   pd            192.168.154.130  2379/2380    linux/x86_64  Up      /tidb-data2/pd-2379            /tidb-deploy2/pd-2379
192.168.154.128:9090   prometheus    192.168.154.128  9090/12020   linux/x86_64  Up      /tidb-data2/prometheus-9090    /tidb-deploy2/prometheus-9090
192.168.154.129:4000   tidb          192.168.154.129  4000/10080   linux/x86_64  Up      -                              /tidb-deploy2/tidb-4000
192.168.154.130:4000   tidb          192.168.154.130  4000/10080   linux/x86_64  Up      -                              /tidb-deploy2/tidb-4000
192.168.154.128:20160  tikv          192.168.154.128  20160/20180  linux/x86_64  Up      /tidb-data2/tikv-20160         /tidb-deploy2/tikv-20160
192.168.154.129:20160  tikv          192.168.154.129  20160/20180  linux/x86_64  Up      /tidb-data2/tikv-20160         /tidb-deploy2/tikv-20160
192.168.154.130:20160  tikv          192.168.154.130  20160/20180  linux/x86_64  Up      /tidb-data2/tikv-20160         /tidb-deploy2/tikv-20160
Total nodes: 11
```
下线 192.168.154.129:4000 节点
https://docs.pingcap.com/zh/tidb/stable/scale-tidb-using-tiup

```
$ tiup cluster scale-in tidb-test --node 192.168.154.129:4000
```
```
tiup is checking updates for component cluster ...
A new version of cluster is available:
   The latest version:         v1.14.1
   Local installed version:    v1.14.0
   Update current component:   tiup update cluster
   Update all components:      tiup update --all

Starting component `cluster`: /root/.tiup/components/cluster/v1.14.0/tiup-cluster scale-in tidb-test --node 192.168.154.129:4000
This operation will delete the 192.168.154.129:4000 nodes in `tidb-test` and all their data.
Do you want to continue? [y/N]:(default=N) y
Scale-in nodes...
+ [ Serial ] - SSHKeySet: privateKey=/root/.tiup/storage/cluster/clusters/tidb-test/ssh/id_rsa, publicKey=/root/.tiup/storage/cluster/clusters/tidb-test/ssh/id_rsa.pub
+ [Parallel] - UserSSH: user=tidb, host=192.168.154.129
+ [Parallel] - UserSSH: user=tidb, host=192.168.154.130
+ [Parallel] - UserSSH: user=tidb, host=192.168.154.129
+ [Parallel] - UserSSH: user=tidb, host=192.168.154.130
+ [Parallel] - UserSSH: user=tidb, host=192.168.154.128
+ [Parallel] - UserSSH: user=tidb, host=192.168.154.128
+ [Parallel] - UserSSH: user=tidb, host=192.168.154.128
+ [Parallel] - UserSSH: user=tidb, host=192.168.154.128
+ [Parallel] - UserSSH: user=tidb, host=192.168.154.129
+ [Parallel] - UserSSH: user=tidb, host=192.168.154.130
+ [Parallel] - UserSSH: user=tidb, host=192.168.154.128
+ [ Serial ] - ClusterOperate: operation=DestroyOperation, options={Roles:[] Nodes:[192.168.154.129:4000] Force:false SSHTimeout:5 OptTimeout:120 APITimeout:600 IgnoreConfigCheck:false NativeSSH:false SSHType: Concurrency:5 SSHProxyHost: SSHProxyPort:22 SSHProxyUser:root SSHProxyIdentity:/root/.ssh/id_rsa SSHProxyUsePassword:false SSHProxyTimeout:5 SSHCustomScripts:{BeforeRestartInstance:{Raw:} AfterRestartInstance:{Raw:}} CleanupData:false CleanupLog:false CleanupAuditLog:false RetainDataRoles:[] RetainDataNodes:[] DisplayMode:default Operation:StartOperation}
Stopping component tidb
	Stopping instance 192.168.154.129
	Stop tidb 192.168.154.129:4000 success
Destroying component tidb
	Destroying instance 192.168.154.129
Destroy 192.168.154.129 finished
- Destroy tidb paths: [/tidb-deploy2/tidb-4000/log /tidb-deploy2/tidb-4000 /etc/systemd/system/tidb-4000.service]
+ [ Serial ] - UpdateMeta: cluster=tidb-test, deleted=`'192.168.154.129:4000'`
+ [ Serial ] - UpdateTopology: cluster=tidb-test
+ Refresh instance configs
  - Generate config pd -> 192.168.154.128:2379 ... Done
  - Generate config pd -> 192.168.154.129:2379 ... Done
  - Generate config pd -> 192.168.154.130:2379 ... Done
  - Generate config tikv -> 192.168.154.128:20160 ... Done
  - Generate config tikv -> 192.168.154.129:20160 ... Done
  - Generate config tikv -> 192.168.154.130:20160 ... Done
  - Generate config tidb -> 192.168.154.130:4000 ... Done
  - Generate config prometheus -> 192.168.154.128:9090 ... Done
  - Generate config grafana -> 192.168.154.128:3000 ... Done
  - Generate config alertmanager -> 192.168.154.128:9093 ... Done
+ Reload prometheus and grafana
  - Reload prometheus -> 192.168.154.128:9090 ... Done
  - Reload grafana -> 192.168.154.128:3000 ... Done
Scaled cluster `tidb-test` in successfully
```
下线需要一定时间，下线节点的状态变为 Tombstone 就说明下线成功
再次查看集群状态，192.168.154.129:4000 的TiDB role已经成功下线

```
tiup is checking updates for component cluster ...
A new version of cluster is available:
   The latest version:         v1.14.1
   Local installed version:    v1.14.0
   Update current component:   tiup update cluster
   Update all components:      tiup update --all

Starting component `cluster`: /root/.tiup/components/cluster/v1.14.0/tiup-cluster display tidb-test
Cluster type:       tidb
Cluster name:       tidb-test
Cluster version:    v7.5.0
Deploy user:        tidb
SSH type:           builtin
Dashboard URL:      http://192.168.154.128:2379/dashboard
Grafana URL:        http://192.168.154.128:3000
ID                     Role          Host             Ports        OS/Arch       Status  Data Dir                       Deploy Dir
--                     ----          ----             -----        -------       ------  --------                       ----------
192.168.154.128:9093   alertmanager  192.168.154.128  9093/9094    linux/x86_64  Up      /tidb-data2/alertmanager-9093  /tidb-deploy2/alertmanager-9093
192.168.154.128:3000   grafana       192.168.154.128  3000         linux/x86_64  Up      -                              /tidb-deploy2/grafana-3000
192.168.154.128:2379   pd            192.168.154.128  2379/2380    linux/x86_64  Up|UI   /tidb-data2/pd-2379            /tidb-deploy2/pd-2379
192.168.154.129:2379   pd            192.168.154.129  2379/2380    linux/x86_64  Up|L    /tidb-data2/pd-2379            /tidb-deploy2/pd-2379
192.168.154.130:2379   pd            192.168.154.130  2379/2380    linux/x86_64  Up      /tidb-data2/pd-2379            /tidb-deploy2/pd-2379
192.168.154.128:9090   prometheus    192.168.154.128  9090/12020   linux/x86_64  Up      /tidb-data2/prometheus-9090    /tidb-deploy2/prometheus-9090
192.168.154.130:4000   tidb          192.168.154.130  4000/10080   linux/x86_64  Up      -                              /tidb-deploy2/tidb-4000
192.168.154.128:20160  tikv          192.168.154.128  20160/20180  linux/x86_64  Up      /tidb-data2/tikv-20160         /tidb-deploy2/tikv-20160
192.168.154.129:20160  tikv          192.168.154.129  20160/20180  linux/x86_64  Up      /tidb-data2/tikv-20160         /tidb-deploy2/tikv-20160
192.168.154.130:20160  tikv          192.168.154.130  20160/20180  linux/x86_64  Up      /tidb-data2/tikv-20160         /tidb-deploy2/tikv-20160
Total nodes: 10
```

通过 jdbc url：192.168.154.128:3390 任然可访问集群

继续下线 192.168.154.130:4000 节点后
尝试 Navicat 连接报错：
2013 - Lost connection to server at "handshake: reading initial communication packet', system error: 0

没有 TiDB 实例了，集群无法使用



**停止 HAProxy**

```
$ ps -ef | grep haproxy
$ kill -9 ${haproxy.pid}
```


### 集群销毁
重命名集群
```
$ tiup cluster rename ${cluster-name} ${new-name}
```
关闭集群
```
$ tiup cluster stop ${cluster-name}
```
**清除集群数据**
https://docs.pingcap.com/zh/tidb/stable/maintain-tidb-using-tiup
清空集群所有服务的数据，但保留日志：
```
$ tiup cluster clean ${cluster-name} --data
```
清空集群所有服务的日志，但保留数据：
```
$ tiup cluster clean ${cluster-name} --log
```
清空集群所有服务的数据和日志：
```
$ tiup cluster clean ${cluster-name} --all
```
**销毁集群**
```
$ tiup cluster destroy tidb-test
```

```
tiup is checking updates for component cluster ...
A new version of cluster is available:
   The latest version:         v1.14.1
   Local installed version:    v1.14.0
   Update current component:   tiup update cluster
   Update all components:      tiup update --all

Starting component `cluster`: /root/.tiup/components/cluster/v1.14.0/tiup-cluster destroy tidb-test

  ██     ██  █████  ██████  ███    ██ ██ ███    ██  ██████
  ██     ██ ██   ██ ██   ██ ████   ██ ██ ████   ██ ██
  ██  █  ██ ███████ ██████  ██ ██  ██ ██ ██ ██  ██ ██   ███
  ██ ███ ██ ██   ██ ██   ██ ██  ██ ██ ██ ██  ██ ██ ██    ██
   ███ ███  ██   ██ ██   ██ ██   ████ ██ ██   ████  ██████

This operation will destroy tidb v7.5.0 cluster tidb-test and its data.
Are you sure to continue?
(Type "Yes, I know my cluster and data will be deleted." to continue)
: 
```

### 数据迁移
https://docs.pingcap.com/zh/tidb/stable/migrate-small-mysql-to-tidb
**TiDB Data Migration (DM) 工具**
https://docs.pingcap.com/zh/tidb/stable/deploy-a-dm-cluster-using-tiup
https://docs.pingcap.com/zh/tidb/stable/dm-worker-intro
查看DM安装版本

```
$ tiup list dm-master
```
```
Available versions for dm-master:
Version                          Installed  Release                    Platforms
-------                          ---------  -------                    ---------
nightly -> v8.0.0-alpha-nightly             2024-02-07T06:14:32Z       linux/arm64,darwin/amd64,darwin/arm64,linux/amd64
v2.0.0-rc                                   2020-08-21T17:47:06+08:00  linux/arm64,linux/amd64
v2.0.0-rc.2                                 2020-09-01T20:50:20+08:00  linux/arm64,linux/amd64
v2.0.0                                      2020-10-30T16:09:39+08:00  linux/arm64,linux/amd64
v2.0.1                                      2020-12-25T13:21:55+08:00  linux/arm64,linux/amd64
v2.0.3                                      2021-05-11T22:13:57+08:00  linux/arm64,linux/amd64
v2.0.4                                      2021-06-18T16:33:54+08:00  linux/arm64,linux/amd64
v2.0.5                                      2021-07-30T18:45:56+08:00  linux/arm64,linux/amd64
v2.0.6                                      2021-08-13T17:35:32+08:00  linux/arm64,linux/amd64
v2.0.7                                      2021-09-29T16:34:01+08:00  linux/arm64,linux/amd64
v5.3.0                                      2021-11-29T16:49:47+08:00  linux/arm64,linux/amd64
v5.3.1                                      2022-03-07T14:16:16+08:00  linux/arm64,linux/amd64
v5.3.2                                      2022-06-29T10:58:53+08:00  linux/arm64,linux/amd64
v5.3.3                                      2022-09-14T19:13:46+08:00  linux/arm64,linux/amd64
v5.3.4                                      2022-11-24T12:34:19+08:00  linux/arm64,linux/amd64
v5.4.0                                      2022-02-14T10:17:44+08:00  linux/arm64,linux/amd64
v5.4.1                                      2022-05-20T16:28:24+08:00  linux/arm64,linux/amd64
v5.4.2                                      2022-07-08T10:06:45+08:00  linux/arm64,linux/amd64
v5.4.3                                      2022-10-13T22:10:43+08:00  linux/arm64,linux/amd64
v6.0.0                                      2022-04-06T11:32:46+08:00  linux/arm64,linux/amd64
v6.1.0                                      2022-06-13T12:22:46+08:00  linux/arm64,linux/amd64
v6.1.1                                      2022-09-01T12:09:57+08:00  linux/arm64,linux/amd64
v6.1.2                                      2022-10-24T15:14:58+08:00  linux/arm64,linux/amd64
v6.1.3                                      2022-12-05T11:48:02+08:00  linux/arm64,linux/amd64
v6.1.4                                      2023-02-08T11:32:42+08:00  linux/arm64,linux/amd64
v6.1.5                                      2023-02-28T11:22:16+08:00  linux/arm64,linux/amd64
v6.1.6                                      2023-04-12T11:02:29+08:00  linux/arm64,linux/amd64
v6.1.7                                      2023-07-12T11:15:22+08:00  linux/arm64,linux/amd64
v6.2.0                                      2022-08-23T09:13:13+08:00  linux/arm64,linux/amd64
v6.3.0                                      2022-09-30T10:58:08+08:00  linux/arm64,linux/amd64
v6.4.0                                      2022-11-17T11:24:51+08:00  linux/arm64,linux/amd64
v6.5.0                                      2022-12-29T11:30:06+08:00  linux/arm64,linux/amd64
v6.5.1                                      2023-03-10T13:34:24+08:00  linux/arm64,linux/amd64
v6.5.2                                      2023-04-21T10:50:07+08:00  linux/arm64,linux/amd64
v6.5.3                                      2023-06-14T14:28:21+08:00  linux/arm64,linux/amd64
v6.5.4                                      2023-08-28T11:32:13+08:00  linux/arm64,linux/amd64
v6.5.5                                      2023-09-21T11:40:10+08:00  linux/arm64,linux/amd64
v6.5.6                                      2023-12-07T06:57:45Z       linux/arm64,linux/amd64
v6.5.7                                      2024-01-08T04:00:44Z       linux/arm64,linux/amd64
v6.5.8                                      2024-02-02T03:20:15Z       linux/arm64,linux/amd64
v6.6.0                                      2023-02-20T16:41:27+08:00  linux/arm64,linux/amd64
v7.0.0                                      2023-03-30T10:29:26+08:00  linux/arm64,linux/amd64
v7.1.0                                      2023-05-31T14:49:44+08:00  linux/arm64,linux/amd64
v7.1.1                                      2023-07-24T11:36:57+08:00  linux/arm64,linux/amd64
v7.1.2                                      2023-10-25T03:29:13Z       linux/arm64,linux/amd64
v7.1.3                                      2023-12-21T03:37:54Z       linux/arm64,linux/amd64
v7.2.0                                      2023-06-29T11:54:42+08:00  linux/arm64,linux/amd64
v7.3.0                                      2023-08-14T12:34:36+08:00  linux/arm64,linux/amd64
v7.4.0                                      2023-10-12T03:54:40Z       linux/arm64,linux/amd64
v7.5.0                                      2023-12-01T03:43:16Z       linux/arm64,linux/amd64
v7.6.0-alpha-instant                        2024-01-11T07:33:12Z       linux/arm64,darwin/amd64,darwin/arm64,linux/amd64
v7.6.0                                      2024-01-25T04:12:22Z       linux/arm64,linux/amd64
v8.0.0-alpha-instant                        2024-01-24T09:06:04Z       linux/arm64,darwin/amd64,darwin/arm64,linux/amd64
v8.0.0-alpha-nightly                        2024-02-07T06:14:32Z       linux/arm64,darwin/amd64,darwin/arm64,linux/amd64
```

安装 TiUP DM 组件：
```
$ tiup install dm dmctl
```
初始化拓扑文件
```
$ tiup dm template > topology.yaml
```
...未完

**TiDB Lightning 工具**
...未完

**Dumpling 工具**
...未完

**更多 迁移工具**
https://docs.pingcap.com/zh/tidb/stable/migration-tools





结束

----

附录
----
**使用密钥方式部署集群**
SSH免密登录
https://blog.csdn.net/weixin_53287520/article/details/123745026
进入`.SSH`目录

```
$ cd ~/.ssh
# 没有目录，直接创建
$ mkdir ~/.ssh
$ pwd
```
```
/root/.ssh
```
生成公私钥
```
$ ssh-keygen -t rsa 
```
上传公钥到各服务器
```
$ ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.154.128
$ ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.154.129
$ ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.154.130
```
使用`-i`秘钥方式部署集群
```
$ tiup cluster check ./topology.yaml -i ~/.ssh/id_rsa
$ tiup cluster check ./topology.yaml --apply -i ~/.ssh/id_rsa
$ tiup cluster deploy tidb-test v7.5.0 ./topology.yaml -i ~/.ssh/id_rsa
```
```
tiup is checking updates for component cluster ...
A new version of cluster is available:
   The latest version:         v1.14.1
   Local installed version:    v1.14.0
   Update current component:   tiup update cluster
   Update all components:      tiup update --all

Starting component `cluster`: /root/.tiup/components/cluster/v1.14.0/tiup-cluster deploy tidb-test v7.5.0 ./topology.yaml -i /root/.ssh/id_rsa



+ Detect CPU Arch Name
  - Detecting node 192.168.154.128 Arch info ... Done
  - Detecting node 192.168.154.129 Arch info ... Done
  - Detecting node 192.168.154.130 Arch info ... Done



+ Detect CPU OS Name
  - Detecting node 192.168.154.128 OS info ... Done
  - Detecting node 192.168.154.129 OS info ... Done
  - Detecting node 192.168.154.130 OS info ... Done
Please confirm your topology:
Cluster type:    tidb
Cluster name:    tidb-test
Cluster version: v7.5.0
Role          Host             Ports        OS/Arch       Directories
----          ----             -----        -------       -----------
pd            192.168.154.128  2379/2380    linux/x86_64  /tidb-deploy2/pd-2379,/tidb-data2/pd-2379
pd            192.168.154.129  2379/2380    linux/x86_64  /tidb-deploy2/pd-2379,/tidb-data2/pd-2379
pd            192.168.154.130  2379/2380    linux/x86_64  /tidb-deploy2/pd-2379,/tidb-data2/pd-2379
tikv          192.168.154.128  20160/20180  linux/x86_64  /tidb-deploy2/tikv-20160,/tidb-data2/tikv-20160
tikv          192.168.154.129  20160/20180  linux/x86_64  /tidb-deploy2/tikv-20160,/tidb-data2/tikv-20160
tikv          192.168.154.130  20160/20180  linux/x86_64  /tidb-deploy2/tikv-20160,/tidb-data2/tikv-20160
tidb          192.168.154.129  4000/10080   linux/x86_64  /tidb-deploy2/tidb-4000
tidb          192.168.154.130  4000/10080   linux/x86_64  /tidb-deploy2/tidb-4000
prometheus    192.168.154.128  9090/12020   linux/x86_64  /tidb-deploy2/prometheus-9090,/tidb-data2/prometheus-9090
grafana       192.168.154.128  3000         linux/x86_64  /tidb-deploy2/grafana-3000
alertmanager  192.168.154.128  9093/9094    linux/x86_64  /tidb-deploy2/alertmanager-9093,/tidb-data2/alertmanager-9093
Attention:
    1. If the topology is not what you expected, check your yaml file.
    2. Please confirm there is no port/directory conflicts in same host.
Do you want to continue? [y/N]: (default=N) y
+ Generate SSH keys ... Done
+ Download TiDB components
  - Download pd:v7.5.0 (linux/amd64) ... Done
  - Download tikv:v7.5.0 (linux/amd64) ... Done
  - Download tidb:v7.5.0 (linux/amd64) ... Done
  - Download prometheus:v7.5.0 (linux/amd64) ... Done
  - Download grafana:v7.5.0 (linux/amd64) ... Done
  - Download alertmanager: (linux/amd64) ... Done
  - Download node_exporter: (linux/amd64) ... Done
  - Download blackbox_exporter: (linux/amd64) ... Done
+ Initialize target host environments
  - Prepare 192.168.154.130:22 ... Done
  - Prepare 192.168.154.128:22 ... Done
  - Prepare 192.168.154.129:22 ... Done
+ Deploy TiDB instance
  - Copy pd -> 192.168.154.128 ... Done
  - Copy pd -> 192.168.154.129 ... Done
  - Copy pd -> 192.168.154.130 ... Done
  - Copy tikv -> 192.168.154.128 ... Done
  - Copy tikv -> 192.168.154.129 ... Done
  - Copy tikv -> 192.168.154.130 ... Done
  - Copy tidb -> 192.168.154.129 ... Done
  - Copy tidb -> 192.168.154.130 ... Done
  - Copy prometheus -> 192.168.154.128 ... Done
  - Copy grafana -> 192.168.154.128 ... Done
  - Copy alertmanager -> 192.168.154.128 ... Done
  - Deploy node_exporter -> 192.168.154.128 ... Done
  - Deploy node_exporter -> 192.168.154.129 ... Done
  - Deploy node_exporter -> 192.168.154.130 ... Done
  - Deploy blackbox_exporter -> 192.168.154.128 ... Done
  - Deploy blackbox_exporter -> 192.168.154.129 ... Done
  - Deploy blackbox_exporter -> 192.168.154.130 ... Done
+ Copy certificate to remote host
+ Init instance configs
  - Generate config pd -> 192.168.154.128:2379 ... Done
  - Generate config pd -> 192.168.154.129:2379 ... Done
  - Generate config pd -> 192.168.154.130:2379 ... Done
  - Generate config tikv -> 192.168.154.128:20160 ... Done
  - Generate config tikv -> 192.168.154.129:20160 ... Done
  - Generate config tikv -> 192.168.154.130:20160 ... Done
  - Generate config tidb -> 192.168.154.129:4000 ... Done
  - Generate config tidb -> 192.168.154.130:4000 ... Done
  - Generate config prometheus -> 192.168.154.128:9090 ... Done
  - Generate config grafana -> 192.168.154.128:3000 ... Done
  - Generate config alertmanager -> 192.168.154.128:9093 ... Done
+ Init monitor configs
  - Generate config node_exporter -> 192.168.154.128 ... Done
  - Generate config node_exporter -> 192.168.154.129 ... Done
  - Generate config node_exporter -> 192.168.154.130 ... Done
  - Generate config blackbox_exporter -> 192.168.154.128 ... Done
  - Generate config blackbox_exporter -> 192.168.154.129 ... Done
  - Generate config blackbox_exporter -> 192.168.154.130 ... Done
Enabling component pd
	Enabling instance 192.168.154.130:2379
	Enabling instance 192.168.154.128:2379
	Enabling instance 192.168.154.129:2379
	Enable instance 192.168.154.129:2379 success
	Enable instance 192.168.154.128:2379 success
	Enable instance 192.168.154.130:2379 success
Enabling component tikv
	Enabling instance 192.168.154.130:20160
	Enabling instance 192.168.154.128:20160
	Enabling instance 192.168.154.129:20160
	Enable instance 192.168.154.128:20160 success
	Enable instance 192.168.154.129:20160 success
	Enable instance 192.168.154.130:20160 success
Enabling component tidb
	Enabling instance 192.168.154.130:4000
	Enabling instance 192.168.154.129:4000
	Enable instance 192.168.154.129:4000 success
	Enable instance 192.168.154.130:4000 success
Enabling component prometheus
	Enabling instance 192.168.154.128:9090
	Enable instance 192.168.154.128:9090 success
Enabling component grafana
	Enabling instance 192.168.154.128:3000
	Enable instance 192.168.154.128:3000 success
Enabling component alertmanager
	Enabling instance 192.168.154.128:9093
	Enable instance 192.168.154.128:9093 success
Enabling component node_exporter
	Enabling instance 192.168.154.128
	Enabling instance 192.168.154.129
	Enabling instance 192.168.154.130
	Enable 192.168.154.129 success
	Enable 192.168.154.130 success
	Enable 192.168.154.128 success
Enabling component blackbox_exporter
	Enabling instance 192.168.154.128
	Enabling instance 192.168.154.129
	Enabling instance 192.168.154.130
	Enable 192.168.154.129 success
	Enable 192.168.154.130 success
	Enable 192.168.154.128 success
Cluster `tidb-test` deployed successfully, you can start it with command: `tiup cluster start tidb-test --init`
```

**配置了PROXY 协议的`topology.yaml`拓扑文件**
topology.yaml文件
```
# # Global variables are applied to all deployments and used as the default value of
# # the deployments if a specific deployment value is missing.
server_configs:
  tidb:
    proxy-protocol.networks: 10.30.234.156
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/tidb-deploy2"
  data_dir: "/tidb-data2"

pd_servers:
  - host: 192.168.154.128
  - host: 192.168.154.129
  - host: 192.168.154.130

tidb_servers:
  - host: 192.168.154.129
  - host: 192.168.154.130

tikv_servers:
  - host: 192.168.154.128
  - host: 192.168.154.129
  - host: 192.168.154.130

monitoring_servers:
  - host: 192.168.154.128

grafana_servers:
  - host: 192.168.154.128

alertmanager_servers:
  - host: 192.168.154.128
```

