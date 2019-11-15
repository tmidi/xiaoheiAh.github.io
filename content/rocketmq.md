---
title: "rocketmq"
date: 2019-08-15T13:19:18+08:00
draft: true
---



# RocketMQ笔记

> 高性能 高吞吐量

## 特性
1. 顺序消息


## 踩坑
官方文档并没有很完善，官方demo中会默认设置brokerIP,在单网卡的情况下没有问题，但是当系统中装有虚拟机或者docker或者像mac一样有很多网卡时启动producer或consumer就会报错。
```java
org.apache.rocketmq.client.exception.MQClientException: Send [3] times, still failed, cost [9199]ms, Topic: TopicTest, BrokersSent: [MacBookPro.local, MacBookPro.local, MacBookPro.local]
See http://rocketmq.apache.org/docs/faq/ for further details.
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.sendDefaultImpl(DefaultMQProducerImpl.java:544)
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.send(DefaultMQProducerImpl.java:1069)
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.send(DefaultMQProducerImpl.java:1023)
	at org.apache.rocketmq.client.producer.DefaultMQProducer.send(DefaultMQProducer.java:214)
	at org.apache.rocketmq.example.quickstart.Producer.main(Producer.java:67)
Caused by: org.apache.rocketmq.remoting.exception.RemotingConnectException: connect to <172.27.224.138:10909> failed
	at org.apache.rocketmq.remoting.netty.NettyRemotingClient.invokeSync(NettyRemotingClient.java:388)
	at org.apache.rocketmq.client.impl.MQClientAPIImpl.sendMessageSync(MQClientAPIImpl.java:351)
	at org.apache.rocketmq.client.impl.MQClientAPIImpl.sendMessage(MQClientAPIImpl.java:335)
	at org.apache.rocketmq.client.impl.MQClientAPIImpl.sendMessage(MQClientAPIImpl.java:298)
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.sendKernelImpl(DefaultMQProducerImpl.java:696)
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.sendDefaultImpl(DefaultMQProducerImpl.java:463)
	... 4 more
```
此时我们需要设置brokerIP为本地的ip即可
在`${rocketmq-home}/distribution/target/apache-rocketmq/conf`下编辑`broker.conf`,追加以下配置:
```
brokerIP1 = 192.168.5.179
```
用该配置重启broker
```bash
bin/mqbroker -n localhost:9876 -c conf/broker.conf
```
