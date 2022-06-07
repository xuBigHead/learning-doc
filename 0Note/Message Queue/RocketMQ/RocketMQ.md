# 前言

![u=4174373953,1719844580&fm=15&gp=0](https://raw.githubusercontent.com/xuBigHead/pic/master/img/RocketMQ%E5%9B%BE%E6%A0%87.png)

[Rocket Github地址](https://github.com/apache/rocketmq/tree/master/docs/cn)

# 概念和特性



## 概念[(Concept)](https://github.com/apache/rocketmq/blob/master/docs/cn/concept.md)

介绍RocketMQ的基本概念模型。

## 特性[(Features)](https://github.com/apache/rocketmq/blob/master/docs/cn/features.md)

介绍RocketMQ实现的功能特性。

# 架构设计

## 架构[(Architecture)](https://github.com/apache/rocketmq/blob/master/docs/cn/architecture.md)

介绍RocketMQ部署架构和技术架构。

## 设计[(Design)](https://github.com/apache/rocketmq/blob/master/docs/cn/design.md)

介绍RocketMQ关键机制的设计原理，主要包括消息存储、通信机制、消息过滤、负载均衡、事务消息等。

# 样例

## 样例 [(Example)](https://github.com/apache/rocketmq/blob/master/docs/cn/RocketMQ_Example.md) 

介绍RocketMQ的常见用法，包括基本样例、顺序消息样例、延时消息样例、批量消息样例、过滤消息样例、事务消息样例等。

# 最佳实践

## 最佳实践 [(Best Practice)](https://github.com/apache/rocketmq/blob/master/docs/cn/best_practice.md)

介绍RocketMQ的最佳实践，包括生产者、消费者、Broker以及NameServer的最佳实践，客户端的配置方式以及JVM和linux的最佳参数配置。

## 消息轨迹指南 [(Message Trace)](https://github.com/apache/rocketmq/blob/master/docs/cn/msg_trace/user_guide.md)

介绍RocketMQ消息轨迹的使用方法。

## 权限管理[(Auth Management)](https://github.com/apache/rocketmq/blob/master/docs/cn/acl/user_guide.md)

介绍如何快速部署和使用支持权限控制特性的RocketMQ集群。

## Dledger快速搭建[(Quick Start)](https://github.com/apache/rocketmq/blob/master/docs/cn/dledger/quick_start.md)

介绍Dledger的快速搭建方法。

## 集群部署[(Cluster Deployment)](https://github.com/apache/rocketmq/blob/master/docs/cn/dledger/deploy_guide.md)

介绍Dledger的集群部署方式。



# 运维管理

## 安装RocketMQ

### Docker

 ```shell
# 拉取RocketMQ镜像
$ docker pull rocketmqinc/rocketmq

# 运行RocketMQ的NameServer
$ docker run -d -p 9876:9876 
-v /opt/rocket/data/namesrv/logs:/root/logs 
-v /opt/rocket/data/namesrv/store:/root/store
--name rmqnamesrv 
-e "MAX_POSSIBLE_HEAP=100000000" rocketmqinc/rocketmq sh mqnamesrv

# 使用相同的镜像运行RocketMQ的broker
$ docker run -d -p 10911:10911 -p  10909:10909   
-v   /opt/rocket/data/broker/logs:/root/logs   
-v   /opt/rocket/rocketmq/data/broker/store:/root/store   
-v   /opt/rocket/conf/broker.conf:/opt/rocketmq/conf/broker.conf --name  rmqbroker --link rmqnamesrv:namesrv 
-e "NAMESRV_ADDR=namesrv:9876"   
-e "MAX_POSSIBLE_HEAP=200000000" rocketmqinc/rocketmq sh  mqbroker   
-c /opt/rocketmq/conf/broker.conf  

# 拉取RocketMQ控制台镜像
$ docker pull pangliang/rocketmq-console-ng
$ docker run -e "JAVA_OPTS=-Drocketmq.namesrv.addr=47.101.48.22:9876 
-Dcom.rocketmq.sendMessageWithVIPChannel=false"
-p 19876:19876 
-t pangliang/rocketmq-console-ng
 ```



## 集群部署[(Operation)](https://github.com/apache/rocketmq/blob/master/docs/cn/operation.md)

介绍单Master模式、多Master模式、多Master多slave模式等RocketMQ集群各种形式的部署方法以及运维工具mqadmin的使用方式。

# API Reference（待补充）

[DefaultMQProducer API Reference](https://github.com/apache/rocketmq/blob/master/docs/cn/client/java/API_Reference_DefaultMQProducer.md)