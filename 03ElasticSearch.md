# ElasticSearch安装文档

添加用户组和用户：
```
$ groupadd elasticsearch
$ useradd -g elasticsearch elasticsearch
```
修改`elasticsearch.yml` 文件
```
$ 
```
修改`jvm.options` 文件，设置jvm参数
```
$ 
```
错误：
```
ERROR: [2] bootstrap checks failed
[1]: max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```
编辑文件
```
vim /etc/sysctl.conf 
vm.max_map_count=655360
```

重启
```
$ sysctl -p
```
错误2：
```
ERROR: [1] bootstrap checks failed
[1]: max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]
```
编辑
cenos7
`/etc/security/limits.d/20-nproc.conf`
cenos6
`/etc/security/limits.d/90-nproc.conf`
添加以下参数：

```
* soft nofile 65536
* hard nofile 65536
* soft nproc 65536
* hard nproc 65536
* soft memlock -1
* hard memlock -1
```










