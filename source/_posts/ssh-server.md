---
title: SSH Server的无密登陆
description: "需要远程登陆ssh server的时候，设置passwordless ssh login是一个必要的步骤。这个文章是我自己在搭建Hadoop cluster的时候配置ssh passwordless的笔记，希望对大家有用。预计用时3分钟"
categories: 
- big data
tags:
- ssh server
- hadoop
---

### 使用场景介绍

我分别启动了三个server, host, IP 和username如下：

| Host Name           | IP Address                    | Username  |
| ------------------- | ----------------------------- | --------- |
| namenode            | 192.168.1.149                 |  user1    |
| resourcemanager     | 192.168.1.150                 |  user2    |
| secnamenode         | 192.168.1.151                 |  user3    |

如果不配置passwordless，登陆方法如下：

``` bash
$ ssh user1@namenode
user1@snamenode's password:
```
**注：** 配置IP和host的映射可以在/etc/hosts里添加如下：

 ```
192.168.1.149       namenode
192.168.1.150       resourcemanager
192.168.1.151       secnamenode
 ```


### 步骤

#### Step 0.（可选）配置IP对应的默认用户名

``` bash
$ sudo vim ~/.ssh/config
```

添加如下：
```
Host namenode
    User user1
Host resourcemanager
    User user2
Host datanode
    User user3
```

#### Step 1. 生成密钥

``` bash
$ ssh-keygen
```
这是互动性的命令，可以直接回车选用默认值。生成完之后，会在`～/.ssh/`中产生两个密钥，其中`id_rsa`是私钥，`id_rsa.pub`是公钥。

#### Step 3. 复制公钥到其他节点

``` bash
$ ssh-copy-id -i ~/.ssh/id_*.pub namenode
$ ssh-copy-id -i ~/.ssh/id_*.pub resourcemanager
$ ssh-copy-id -i ~/.ssh/id_*.pub secnamenode
```

#### Step 4. 测试

``` bash
$ ssh namenode
$ ssh resourcemanager
$ ssh secnamenode
```