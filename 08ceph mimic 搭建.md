# ceph mimic`13.2.10` 版本搭建
## 系统环境 `CentOS 7`

**基础环境配置：**

1. 查看系统版本（Linux查看操作系统版本的几种方式）
```
$ uname -a
$ lsb_release -a
$ cat /etc/issue
$ cat /proc/version
$ cat /etc/redhat-release
```
```
$ cat /etc/centos-release
```
```
CentOS Linux release 7.8.2003 (Core)
```
2. 更新系统（选中第一个更新即可）
```
# 系统和软件配置不做修改（升级所有包和系统版本，不改变内核,软件和系统设置）
$ yum -y upgrade
# 系统和软件配置文件更新（升级所有包,系统版本和内核，改变软件设置和系统设置）
$ yum -y update
# 重启系统
$ reboot
```

3. 关闭防火墙和禁用selinux
```
$ systemctl stop firewalld
$ systemctl disable firewalld
$ sed -i 's/enforcing/disabled/' /etc/selinux/config
$ setenforce 0
```
4. 配置服务器的NTP时间同步
```
$ yum install ntp ntpdate -y
$ timedatectl set-ntp yes
```
5. 配置所有节点/etc/hosts
```
$ vim /etc/hosts
```
```
192.168.1.170 admin
192.168.1.172 node1
192.168.1.173 node2
192.168.1.174 node3
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```
```
# 设置本机hostname（各自节点执行）
$ sysctl kernel.hostname=admin
$ vim /etc/hostname
```
```
admin
```
6. 配置部署节点到所有节点的无秘钥访问
```
$ ssh-keygen 一直enter键即可
$ ssh-copy-id root@admin
$ ssh-copy-id root@node1
$ ssh-copy-id root@node2
$ ssh-copy-id root@node3
```
**安装ceph[所有节点]**
1. 配置ceph的yum源
```
$ vim /etc/yum.repos.d/ceph.repo
```
```
[ceph]
name=ceph
baseurl=http://mirrors.aliyun.com/ceph/rpm-mimic/el7/x86_64/
gpgcheck=0
gpgkey=http://mirrors.aliyun.com/ceph/keys/release.asc
priority=1

[ceph-noarch]
name=cephnoarch
baseurl=http://mirrors.aliyun.com/ceph/rpm-mimic/el7/noarch/
gpgcheck=0
gpgkey=http://mirrors.aliyun.com/ceph/keys/release.asc
priority=1

[ceph-source]
name=cephsource
baseurl=http://mirrors.aliyun.com/ceph/rpm-mimic/el7/SRPMS/
gpgcheck=0
gpgkey=http://mirrors.aliyun.com/ceph/keys/release.asc
priority=1
```
2. 安装epel-release
```
$ yum install -y epel-release
```
3. 安装ceph
```
$ yum install -y ceph
```
4. 安装ceph-deploy（仅admin节点安装即可）
```
$ yum install -y ceph-deploy
```
5. 查看ceph的版本
```
$ ceph -v
```
```
ceph version 13.2.10 (564bdc4ae87418a232fc901524470e1a0f76d641) mimic (stable)
```
6. 配置ceph.conf
```
$ cd /etc/ceph
$ vim ceph.conf
```
```
public network = 192.168.1.0/24
cluster network = 192.168.1.0/24
```
7. 杀进程（所有节点都要执行）
```
$ netstat  -anp  |grep 6789 
```
```
tcp        0      0 192.168.1.170:60848     192.168.1.172:6789      ESTABLISHED 1025/ceph-mon
```
```
$ pkill ceph-mon
```
**部署ceph的mon服务（部署节点执行即可）**
```
$ cd /etc/ceph
# 这里是创建三个Mon节点
$ ceph-deploy new admin node1 node2 node3
# 这里是执行mon的初始化
$ ceph-deploy mon create-initial
```
**部署ceph的mgr服务**
```
$ cd /etc/ceph
$ ceph-deploy mgr create admin node1 node2 node3
# 这里是同步配置文件到所有集群节点
$ ceph-deploy admin admin node1 node2 node3
```
**部署ceph的osd服务**
```
$ cd /etc/ceph
# 这里是将做OSD的磁盘分区格式化
$ ceph-deploy disk zap node1 /dev/sdb
$ ceph-deploy osd create --data /dev/sdb node1 
```
**查看集群状态**

```
$ ceph -s
$ ceph health
```
**开启ceph的Dashboard**
1. Dashboard的基础设置
```
# 停用dashboard
$ ceph mgr module disable dashboard   
# 开启dashboard
$ ceph mgr module enable dashboard     
# 生成并安装一个 自签名证书
$ ceph dashboard create-self-signed-cert
#创建管理员 用于web控制台的登录账号
$ ceph dashboard set-login-credentials admin admin
# 配置服务地址、端口，默认的端口是8443
$ ceph config set mgr mgr/dashboard/server_addr 192.168.1.170
$ ceph config set mgr mgr/dashboard/server_port 8443
# 确认验证
$ ceph mgr services
```
```
{
    "dashboard": "https://admin:8443/"
}
```
```
# 禁用SSL
$ ceph config set mgr mgr/dashboard/ssl false
```
2. 启用对象网关管理前端（Object Gateway）
文档：https://docs.ceph.com/docs/mimic/mgr/dashboard/#enabling-the-object-gateway-management-frontend
```
# 必须创建系统型用户，否则会报错导致无法使用网关管理 --system
$ radosgw-admin user create --uid=<user_id> --display-name=<display_name> --system
# 查询用户详情
$ radosgw-admin user info --uid=<user_id>
```
```
{
    "user_id": "rgw",
    "display_name": "rgw",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "auid": 0,
    "subusers": [],
    "keys": [
        {
            "user": "rgw",
            "access_key": "3JM68EC66GH184ATTRZM",
            "secret_key": "zvXGwovRufsVi1kYIU2OlMydKgaL1T46aMn0deCA"
        }
    ],
    "swift_keys": [],
    "caps": [],
    "op_mask": "read, write, delete",
    "system": "true",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "temp_url_keys": [],
    "type": "rgw",
    "mfa_ids": []
}
```
```
# 向仪表板模块提供凭据（access_key和secret_key通过用户详情中获取）
$ ceph dashboard set-rgw-api-access-key <access_key>
$ ceph dashboard set-rgw-api-secret-key <secret_key>
# 设置主机和端口
$ ceph dashboard set-rgw-api-host <host>
$ ceph dashboard set-rgw-api-port <port>
# http or https
$ ceph dashboard set-rgw-api-scheme <scheme>

$ ceph dashboard set-rgw-api-admin-resource <admin_resource>
# 绑定访问网关的用户
$ ceph dashboard set-rgw-api-user-id <user_id>
# 禁用证书验证
$ ceph dashboard set-rgw-api-ssl-verify False
# 网关的超时设置，默认值为45秒
$ ceph dashboard set-rest-requests-timeout <seconds>
```
**启动、停止、重启、查看MON进程**
https://blog.csdn.net/don_chiang709/article/details/93620825

```
# 登陆到monitor节点，执行如下命令
# sudo systemctl [start/stop/restart/status] ceph-mon@mon‘sid.service，例如：我的monitor id为node1
$ systemctl status ceph-mon@node1.service
# 查看mon节点上所有启动的ceph服务，命令：systemctl list-units --type=service|grep ceph
$ systemctl list-units --type=service|grep ceph
# 查看节点上所有自动启动的ceph服务，命令：systemctl list-unit-files|grep enabled|grep ceph
$ systemctl list-unit-files|grep enabled|grep ceph
# 设置ceph-mon随Linux 系统自动启动
$ systemctl enable ceph-mon@node1.service
```
**启动、停止、重启、查看OSD所有和单个进程**
```
# 登陆到OSD 节点，对服务器上的所有OSD操作
# sudo systemctl [start/stop/restart/status] ceph-osd@* or eph-osd@osd_id.service
# 如上把@* 替换为OSD的ID 如@0，即可执行对应ID的 OSD操作。这里的osd id的值 是0，不是hostname了，例如：
sudo systemctl [start/stop/restart/status] ceph-osd@0.service
# 设置ceph-osd随Linux 系统自动启动
$ systemctl enable ceph-osd@0.service

```

结束

----

附录
----

**命令集合**
文档：https://ceph.readthedocs.io/en/latest/man/8/radosgw-admin/
1. 操作用户
```
# 查看用户信息：
$ radosgw-admin user info --uid=admin
# 创建用户（指定access_key和secret）：
$ radosgw-admin user create --uid=admin --display-name=admin --access_key=admin --secret=admin
# 创建系统用户：
$ radosgw-admin user create --uid=system --display-name=system --system
# 查看用户列表：
$ radosgw-admin user list

```
2. 操作bucket
```
# 查看bucket列表
$ radosgw-admin bucket list
# 把桶关联到指定用户
$ radosgw-admin bucket link
# 取消指定用户和桶的关联
$ radosgw-admin bucket unlink
# 返回桶的统计信息
$ radosgw-admin bucket stats
$ radosgw-admin bucket stats --bucket=ccs
# 删除一个桶。
$ radosgw-admin bucket rm --bucket=ccs
# 检查桶的索引信息
$ radosgw-admin bucket check --bucket=ccs
```

3. s3cmd常用命令
文档：
[1]https://blog.51cto.com/11433696/1854502
[2]https://blog.csdn.net/baidu_26495369/article/details/81535209
```
# 安装
$ yum install -y s3cmd
# 配置
$ vim /root/.s3cfg
```
```
[default]
access_key = 1WO8NCINBUDE02WD2Y5U
secret_key = mWVndEB7epF0UUrsoPciG5QcPpqNoDSENvmItnBA
host_base = 192.168.1.170:7480
host_bucket = 192.168.1.170/test
use_https = False
```
```
# 列举所有buckets
$ s3cmd ls
# 创建bucket
$ s3cmd mb s3://{$BUCKETNAME}
# 删除空bucket
$ s3cmd rb s3://{$BUCKETNAME}
# 上传某个文件到bucket
$ s3cmd put {$FILENAME}t s3://{$BUCKETNAME}
# 列举bucket中的内容
$ s3cmd ls s3://{$BUCKETNAME}
# 下载文件
$ s3cmd get s3://{路径+文件名}
# 删除文件
$ s3cmd del/rm s3://{路径+文件名}
# 获取对应的bucket所占用的的空间大小
$ s3cmd du -H s3://{目录}
# 查看更多关于bucket和文件的信息
$ s3cmd info s3://BUCKET[/OBJECT]
```

4. 磁盘挂载
```
# 扫描磁盘使挂载盘出现在列表
$ echo "- - -" > /sys/class/scsi_host/host0/scan
# 查看磁盘分区列表
$ fdisk -l
# 列出所有可用块设备的信息
$ lsblk
```







