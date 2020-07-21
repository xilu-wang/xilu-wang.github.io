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

## 集群部署规划

|       | node-1               | node-2                         | node-3                       |
| ------| :-------------------:  | :-----------------------------:  | :----------------------------: |
| HDFS  | NameNode, DataNode   | DataNode                       | DataNode, SecondaryNameNode  |
| YARN  | NodeManager          | ResourceManager, NodeManager   | Node Manager                 |

## Virtual Machine （VM）准备工作

我用的机器是一台Windows的Desktop。也算是配置一般的十年前的老机器了。原来一直闲置在车库，没想到十年后还能工作的这么好。配置VM我用的是Vagrant和VirtualBox，具体操作如下。

### 启动VM

我已经写好用于vm配置和启动的Vagrantfile和bootstrap.sh，同时在bootstarp.sh里也更改了hadoop集群的一些基本配置。
**关于hadoop集群的配置介绍，可以参考我的另一篇文章：** [如何配置小规模的Hadoop集群](../hadoop-cluster-config)

``` bash
$ git clone https://github.com/xilu-wang/vagrant-hadoop.git
$ cd vagrant-hadoop
$ vagrant up
```

### 更改每个VM的localdomain，以及IP对应的host
### 设置ssh passwordless登陆
### 检查hdfs和yarn的配置
### 启动hdfs和yarn
### 测试hdfs client api和简单map reduce job

## 潜在问题和Debug：

* 如果执行完vagrant up之后，vm无法登入，可以重新进入本地的git repo再执行一次 vagrant reload

