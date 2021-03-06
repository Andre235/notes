查看防火墙状态：

```shell
systemctl status firewalld
```

临时关闭防火墙

```shell
systemctl stop firewalld
```

永久关闭防火墙

```shell
systemctl disable firewalld
```

打开防火墙命令

```shell
systemctl enable firewalld
```

#### ES服务配置

**增加Linux系统部署软件的内存和硬盘**

```xml
vim /etc/security/limits.conf
```

在最后一样添加如下配置

```shell
* soft nproc 655350
* soft nofile 655350
* hard nproc 655350
* hard nofile 655350
```

**配置ES最大线程数**

```
vim /etc/sysctl.conf
```

在最后一行添加

```
vm.max_map_count = 262144
```

**配置用户最大的线程数**

```
vim /etc/security/limits.d/90-nproc.conf
```

**使以上所有配置永久生效**

```shell
[root@ES01 opt]# sysctl -p
vm.max_map_count = 262144
```

 **启动ES**

```shell
# 进入bin目录
[root@ES01 bin]# ./elasticsearch
```

第一次启动报错

```java
java.lang.RuntimeException: can not run elasticsearch as root
// 报错原因：因为root用户的权限过大，不能以root用户身份启动ES
```

解决方案：创建用户，给该用户分配在/elasticsearch目录下所有文件的执行权限，然后切换刚才创建的用户，以该用户身份启动ES服务。

启动成功后，如果在虚拟机浏览器可以访问  http://192.168.10.131:9200/，（192.168.10.131是我自己的虚拟机ip地址）但是物理机却不能访问 http://192.168.10.131:9200/的话，一般可以通过关闭虚拟机防火墙或者开放虚拟机90端口就可以解决。（二选一即可，更改为配置后记得**重启虚拟机**）

启动成功后浏览器返回Json字符串

```json
{
  "name" : "node-1",
  "cluster_name" : "my-cluster",
  "cluster_uuid" : "xaAyktXiSU25xWUKoA33CQ",
  "version" : {
    "number" : "7.7.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "81a1e9eda8e6183f5237786246f6dced26a10eaf",
    "build_date" : "2020-05-12T02:01:37.602180Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

**配置ES的yml文件**

```yml
# ES集群名称
cluster.name: my-cluster
# ES节点名称，如果是集群模式，每个节点名称不能重复
node.name: node-1
# ES数据文件存放目录
path.data: /opt/ElasticSearch/elasticsearch-7.7.0/data
# ES日志文件存放目录
path.data: /opt/ElasticSearch/elasticsearch-7.7.0/data
# 释放ES内存锁，让ES拥有最大的内存
bootstrap.memory_lock: false
# 配置允许连接的IP地址，这里配置为0.0.0.0，表示所有的终端都可以连接ES服务
network.host: 0.0.0.0
# ES默认端口号为9200，不建议修改
http.port: 9200
# 识别集群中其他节点的host，如果是单节点只需要配置一个host即可
discovery.seed_hosts: ["192.168.10.131"]
```

### IK分词器

下载IK分词器插件，放到你的ES安装目录下的plugins目录中，然后解压，IK分词器配置完成。

**注意事项：**你的IK分词器版本和ES版本必须保持一致，即IK分词器版本不能高于也不能低于ES版本，否则ES服务启动报版本不匹配错误。

博主ES版本为7.7.0，IK分词器版本也必须为7.7.0。

### Kibana

`介绍：`Kibana是一款开源的数据分析和可视化平台（主要用于操作ES），他可以非常方便的对ES进行添加index，type操作，document操作（数据的增删改查）

`Linux配置Kibana：`

