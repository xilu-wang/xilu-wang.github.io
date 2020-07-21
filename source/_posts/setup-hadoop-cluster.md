---
title: Hadoop集群快速搭建
date: 2020-07-01 17:36:01
categories: 
- hadoop
tags: 
- hadoop
- hdfs
- yarn
description: "在这个无聊的周末，我用了一个闲置很久的老电脑搭了一个hadoop的开发环境，这篇文章先介绍hdfs和yarn的搭建。预计用时15分钟。"
---

# Hadoop Cluster的搭建

## 1. 集群部署规划

|       | node-1               | node-2                         | node-3                       |
| ------| :-------------------:  | :-----------------------------:  | :----------------------------: |
| HDFS  | NameNode, DataNode   | DataNode                       | DataNode, SecondaryNameNode  |
| YARN  | NodeManager          | ResourceManager, NodeManager   | Node Manager                 |

## 2. Virtual Machine （VM）准备工作

我用的机器是一台Windows的Desktop。也算是配置一般的十年前的老机器了。原来一直闲置在车库，没想到十年后还能工作的这么好。配置VM我用的是Vagrant和VirtualBox，我已经写好用于vm配置和启动的Vagrantfile和bootstrap.sh，同时在bootstarp.sh里也更改了hadoop集群的一些基本配置。

``` bash
$ git clone https://github.com/xilu-wang/vagrant-hadoop.git
$ cd vagrant-hadoop
$ vagrant up
```
*注：可以在VM跑起来之后删除.vagrant文件夹，重新执行vagrant up，就会新建新的VM。如此重复三次。*

## 3. 登陆VM并更新host

**login:** admin
**password:** admin

server-1:
``` bash
$ sudo hostnamectl set-hostname namenode
```
server-2:
``` bash
$ sudo hostnamectl set-hostname resourcemanager
```
server-3:
``` bash
$ sudo hostnamectl set-hostname secnamenode
```


同时，可以通过ifconfig拿到server IP，用来更新host的映射。

结果示例：
``` bash
$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::a00:27ff:fe15:602e  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:15:60:2e  txqueuelen 1000  (Ethernet)
        RX packets 240668  bytes 210747618 (210.7 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 108428  bytes 22769110 (22.7 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.150  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 2600:1700:1151:b370::26  prefixlen 128  scopeid 0x0<global>
        inet6 fe80::a00:27ff:fe22:12d0  prefixlen 64  scopeid 0x20<link>
        inet6 2600:1700:1151:b370:a00:27ff:fe22:12d0  prefixlen 64  scopeid 0x0<global>
        ether 08:00:27:22:12:d0  txqueuelen 1000  (Ethernet)
        RX packets 1797145  bytes 433847872 (433.8 MB)
        RX errors 0  dropped 550377  overruns 0  frame 0
        TX packets 570469  bytes 88389007 (88.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 584917  bytes 81921922 (81.9 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 584917  bytes 81921922 (81.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
在这里，eth1下面的`inet 192.168.1.150`既是本机IP。

拿到三个VM的IP后，依次在每个VM的`/etc/hosts`文件里加入:
 ```
192.168.1.149       namenode
192.168.1.150       resourcemanager
192.168.1.151       secnamenode
 ```

## 4. 设置ssh passwordless

``` bash
$ ssh-keygen
$ ssh-copy-id -i ~/.ssh/id_*.pub namenode
$ ssh-copy-id -i ~/.ssh/id_*.pub resourcemanager
$ ssh-copy-id -i ~/.ssh/id_*.pub secnamenode
```
细节可以参考我的另外一篇博客：**[SSH server的无密登陆](../../misc/ssh-server)**

## 5. 检查hdfs和yarn的配置
Hadoop的配置已经在bootstrap vm的时候已经更新了。

需要检查的文件分别是：
1. core-site.xml * 
2. hdfs-site.xml *
3. yarn-site.xml *
4. mapred-site.xml
5. hadoop-env.sh
6. slaves *

*如果在host里面没有添加IP的映射，需要将文件中对应的hostname更新成原IP。

关于hadoop集群的配置介绍，可以参考我的另一篇文章：**[如何配置小规模的Hadoop集群](../hadoop-cluster-config)**

## 6. 启动hdfs和yarn

```bash
$ cd /home/admin/hadoop/hadoop-2.7.2/
```

#### 第一步：格式化NameNode

0. 在node-1（namenode）进行格式化

```bash
$ hadoop namenode -format
```

#### 第二步：（选择一，不推荐）单点启动/停止

1. 在node-1（namenode）启动namenode
```bash
$ sbin/hadoop-daemon.sh start namenode
```

2. 在node-2（resourcemanager）启动yarn
```bash
$ sbin/yarn-daemon.sh start resourcemanager
```

3. 在node-3（secnamenode）启动
```bash
$ sbin/hadoop-daemon.sh start secondarynamenode
```

4. 在node-1， node-2， node-3启动datanode
```bash
$ sbin/hadoop-daemon.sh start datanode
```

5. 在node-1， node-2， node-3启动nodemanager
```bash
$ sbin/hadoop-daemon.sh start nodemanager
```

#### 第二步：（选择二，推荐）集群启动/停止

1. 在namenode节点启动/停止：
```bash
$ sbin/start-dfs.sh
```
```bash
$ sbin/stop-dfs.sh
```

2. 在resourcemanager节点启动/停止：
```bash
$ sbin/start-yarn.sh
```
```bash
$ sbin/stop-yarn.sh
```

## 7. 测试hdfs client api和简单map reduce job

我是自己写了一个hdfs的client api，上传/下载文件test.txt
示例client api代码：
```bash
$ git clone https://github.com/xilu-wang/hdfs-client-java.git
```
测试一下mapreduce
```bash
$ cd /home/admin/hadoop/hadoop-2.7.2
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount hdfs:/test.txt hdfs:/output
```

## 8. UI
**Yarn:** http://resourcemanager:8088/
**Hadoop overview:** http://secnamenode:50090/status.html

# 潜在问题和Debug：

* 如果执行完vagrant up之后，vm无法登入，可以重新进入本地的git repo再执行一次 vagrant reload
* 如果集群集体启动的时候没有远程启动datanode/nodemanager，可以删掉slaves，新建slaves文件，重新格式化namenode
* 重新格式化namenode的时候，一定要把name & data文件夹全都删除，位置在/home/admin/hadoop下
* 如果mapreduce job起来的时候报错connection confused，可能是好几个vm之间的localhost问题，需要注释掉`/etc/hosts/`里面的所有有关127.0.0.1和127.0.1.1的映射hostname
