---
title: CentOS Kafka 3.2.0 单机集群安装（伪集群）
comments: false
theme: xray
tags: kafka
categories: 中间件
abbrlink: 12140
date: 2022-07-03 21:45:23
---
### 1. 准备工作
由于没有那么多台机器，所以在同一台机器上运行多个Kafka服务，只是端口不同

1) 安装路径： /usr/local/tools ; 服务器IP： 192.168.44.161

2) 基于Kafka单机版安装流程，请查看 [CentOS安装kafka 3.2.0单机版](https://xiaoyuge.work/kafka-install/)

3) 所有Kafka节点连接到相同的ZK（或ZK集群），需要先安装一个ZK，请参考 [CentOS安装Zookeeper 3.7.1单节点](https://xiaoyuge.work/zookeeper-install/) , 在本例中ZK也安装在这台机器上。

注意：单机的kafka和集群的Kafka不要混用一个ZK，否则会出现数据混乱的问题。


### 2. 下载解压kafka
```shell
cd /usr/local/tools
wget https://dlcdn.apache.org/kafka/3.2.0/kafka_2.12-3.2.0.tgz
tar -xzvf kafka_2.12-3.2.0.tgz
cd kafka_2.12-3.2.0
```

### 3. 修改配置文件
复制3个配置文件
```shell
cd config
cp server.properties server1.properties 
cp server.properties server2.properties 
cp server.properties server3.properties 
```
修改配置文件中的broker.id分别为1、2、3

listeners这一行取消注释，端口号分别为9093、9094、9095

log.dirs分别设置为kafka-logs1、kafka-logs2、kafka-logs3（先创建）
```shell
mkdir -p /tmp/kafka-logs1 /tmp/kafka-logs2 /tmp/kafka-logs3
```
- server1.properties 的配置：
    ```shell
    broker.id=1
    listeners=PLAINTEXT://192.168.44.161:9093
    log.dirs=/tmp/kafka-logs1
    ```
  
- server2.properties 的配置:
    ```shell
    broker.id=2
    listeners=PLAINTEXT://192.168.44.161:9094
    log.dirs=/tmp/kafka-logs2
    ```

- server3.properties 的配置：
    ```shell
    broker.id=3
    listeners=PLAINTEXT://192.168.44.161:9095
    log.dirs=/tmp/kafka-logs3
    ```
  
如果listeners取消注释导致topic创建失败，可以修改为
```properties
listeners=PLAINTEXT://:9093
advertised.listeners=PLAINTEXT://10.1.14.159:9093
```
  
### 4. 启动3个服务
1. 启动ZK
   ```shell
    cd /usr/local/tools/apache-zookeeper-3.7.1-bin/bin
    ./zkServer.sh start
    ```
2. 启动Kafka
    ```shell
    cd ../bin
    ./kafka-server-start.sh -daemon ../config/server1.properties
    ./kafka-server-start.sh -daemon ../config/server2.properties
    ./kafka-server-start.sh -daemon ../config/server3.properties
    ```

PS：如果遇到zk node exists的问题，先把brokers节点删掉（临时解决方案）。

### 5. 集群下创建Topic
在bin目录下，创建一个名为ygbtest的`topic`，只有一个服务本一个分区：
```shell
sh kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic ygbtest
```

查看一创建的topic：
```shell
sh kafka-topics.sh -list -zookeeper localhost:2181
```

### 6. 集群下启动Consumer
在一个新的原车鞥窗口中：
```shell
sh kafka-console-consumer.sh --bootstrap-server 192.168.44.161:9093,192.168.44.161:9094,192.168.44.161:9095 --topic ygbtest --from-beginning
```
kafka相关命令可以查看这篇博客 [kafka常用命令](https://xiaoyuge.work/kafka-command/)

### 7. 集群下启动Producer
打开一个新的窗口，在kafka解压目录下：
```shell
sh kafka-console-producer.sh --broker-list 192.168.44.161:9093,192.168.44.161:9094,192.168.44.161:9095 --topic ygbtest
```

### 8. 集群下Producer窗口发送消息
在生产者`Producer`窗口输入hello world 回车



