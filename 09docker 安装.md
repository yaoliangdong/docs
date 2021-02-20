# docker安装
## 系统环境 `CentOS 7`
**基础环境检查**

**安装docker**

1. 卸载旧版本(如果安装过旧版本的话)
```
$ sudo yum remove docker  docker-common docker-selinux docker-engine
```
安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
```
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
设置yum源
```
# 阿里yum源
$ sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
可以查看所有仓库中所有docker版本，并选择特定版本安装
```
$ yum list docker-ce --showduplicates | sort -r
```
安装docker
```
$ sudo yum install docker-ce  #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0
$ sudo yum install <FQPN>  # 例如：sudo yum install docker-ce-17.12.0.ce

$ sudo yum install docker-ce docker-ce-cli containerd.io
$ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```
启动并加入开机启动
```
$ sudo systemctl start docker
$ sudo systemctl enable docker
```
验证安装是否成功(有client和service两部分表示docker安装启动都成功了)
```
$ docker version
```
**docker swarm**
```
# 作为集群的管理
docker swarm
    # 初始化一个swarm
    - docker swarm init
          # 指定初始化ip地址节点
          - docker swarm init --advertise-addr 管理端IP地址
          # 去除本地之外的所有管理器身份
          - docker swarm init --force-new-cluster
    # 将节点加入swarm集群，两种加入模式manager与worker
    - docker swarm join
          # 工作节点加入管理节点需要通过join-token认证
          - docker swarm join-token
          # 重新获取docker获取初始化命令
          - docker swarm join-token worker
    # 离开swarm
    - docker swarm leave
    # 对swarm集群更新配置
    - docker swarm update
```


