# Kafka基础

## 认识Kafka

### 介绍

- 流处理平台
- 基于zookeeper的分布式消息系统 
- 具有高吞吐率、高性能、实时以及高可靠性特点

### 安装常见问题

1. Socket server failed to bind to 1.15.189.113:9092: 无法指定被请求的地址.

   公网ip替换为内网ip

2. 

### 常见命令

1、启动Kafka
bin/kafka-server-start.sh config/server.properties &

2、停止Kafka
bin/kafka-server-stop.sh

3、创建Topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic jiangzh-topic

4、查看已经创建的Topic信息
bin/kafka-topics.sh --list --zookeeper localhost:2181

5、发送消息
bin/kafka-console-producer.sh --broker-list 172.24.0.1:9092 --topic jiangzh-topic

6、接收消息
bin/kafka-console-consumer.sh --bootstrap-server 172.24.0.1:9092 --topic jiangzh-topic --from-beginning
