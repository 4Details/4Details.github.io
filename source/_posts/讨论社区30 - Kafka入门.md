---
title: 讨论社区30  -  Kafka入门
date: 2019-08-03 23:25:32
tags: [Kafka]
category: [讨论社区项目]
---

本项目中仅使用Kafka的消息系统

- Kafka简介
  - Kafka是一个分布式的流媒体平台。
  - 应用：消息系统、日志收集、用户行为追踪、流式处理。
- Kafka特点
  - 高吞吐量、消息持久化、高可靠性、高扩展性。
- Kafka术语
  - Broker、Zookeeper
  - Topic、Partition、Offset
  - Leader Replica 、Follower Replica

下载地址：http://kafka.apache.org/downloads

解压缩之后需要更改一些配置

windows 启动zookeeper和kafka（windows下启动报错，可以尝试直接删除日志文件夹重新启动）

```shell
zookeeper-server-start.bat d:\software\kafka_2.12-2.3.0\config\zookeeper.properties

kafka-server-start.bat d:\software\kafka_2.12-2.3.0\config\server.properties
```

![windows启动zookeeper](https://s1.ax1x.com/2020/09/08/wl1ZIP.jpg)

![window启动kafka](https://s1.ax1x.com/2020/09/08/wl1Vat.jpg)

创建一个test主题 分区

![创建test主题 分区](https://s1.ax1x.com/2020/09/08/wl1iKH.md.png)

查询是否创建成功

```shell
D:\software\kafka_2.12-2.3.0\bin\windows>kafka-topics.bat --list --bootstrap-server localhost:9092
test
```

生产者生产消息（重新开生产者终端）

```shell
D:\software\kafka_2.12-2.3.0\bin\windows>kafka-console-producer.bat --broker-list localhost:9092 --topic test
>hi
>hello world
>
```

消费者消费消息（重新开消费者终端）

```shell
# 从头开始消费
D:\software\kafka_2.12-2.3.0\bin\windows>kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
hi
hello world

```

