+++
date = "2018-06-25T21:12:23+08:00"
title = "流处理平台 Kakfa"
showonlyimage = false
image = "/img/blog/head-first-kafka/pipe.png"
draft = false
weight = 9999
+++

全新的分布式平台设计思路
<!--more-->

## 为什么快

Kafka 尽量避免随机访问，而采用连续读写模式。利用硬盘在此模式下能达到较高吞吐量的优势，尽量将所有东西全写入磁盘。
Kafka 运行在 JVM 上，如果数据放入内存中作为对象存储，1. 额外开销很很大，占用的空间可能会翻倍；2. 垃圾收集导致 [Heap](https://www.yourkit.com/docs/kb/sizes.jsp)上对象创建/销毁都非常“昂贵”。

## 基本元素

Producer, Consumer, Zookeeper (Boker)
API: Kafka Connect Source and Sink, Kakfa Stream, Mirror Maker
Kafka Registry and Avro and Kafka REST Proxy
GUI: Topics/Schema/Connect UI, Kakfa Manager, Burrow, Exhibitor, Monitor, Tools, Kafkat, JMX Dump and other paid tools

## Schema registry 引入背景

为了应对生产者发送了错误数据，或是随着业务变换，字段更名，格式改变，我们需要数据有自描述的特性，处于系统下游的消息消费方能随着数据进化而不崩溃。

但这部分又因为性能上的考量，不能放入 Broker 中，
- Kakfa 不会去试图解析、读取数据、完全不关心数据是何种形式——也就没有 CPU 占用;
- Kakfa 直接使用字节码作为输入甚至不需要将其加载入内存—— zero copy;
- Kakfa 直接以字节码分发

引入一个单独的组件 schema registry，满足
- 生产方和消费方可以与之通信
- 可以拒绝坏数据
- 采用一种通用的数据格式：支持 schema 定义，可以演进，轻量级

因为引入 Apache Avro 和 Confluent Schema Registry

## Avro

- schema 定义，简单和复杂类型定义
- 创建 Avro 对象，序列化相关操作
- 对 schema 演进的工作流程

数据定义常见形式:
- CSV 易用但对数据完整性和类型都无从保障，需要各种推测
- DDL 能严格保证但应用在无层级关系数据，不同数据库定义各异
- JSON 被广泛接受，可适用于复杂数据结构，但无 schema 约束，体积大
Avro (Protobuf/Thrift/Parquet/ORC ...)

avsc字段：Type, Name, Namespace, Doc, Fields (Name, Doc, Type, Default)
原生类型: null, boolean, int, long, float, double, bytes, string
复合类型: Enums, Arrays, Maps, Unions, Calling other schemas as types

## Avro + Kafka Schema Registry

- 组件处于架构中的位置
- 在 Producer, Consumer, Stream, Connect 中用法
- 向前、向后和完全兼容

## REST Proxy

- 架构和 API
- Producer Consumer 中使用二进制、JSON 或 AVRO 格式

python 支持
https://github.com/dpkp/kafka-python

https://github.com/confluentinc/confluent-kafka-python 官方尚未支持 Windows 平台
https://github.com/confluentinc/confluent-kafka-python/issues/132

### 参考文档

> - Matt Howlett (2017-06-07) [Introduction to Apache Kafka® for Python Programmers](https://www.confluent.io/blog/introduction-to-apache-kafka-for-python-programmers/)
> - Apache Kafka Series [Stephane Maarek](https://www.udemy.com/user/stephane-maarek/)

封面图片来自 [Supply Chain Management Icon](https://dribbble.com/shots/2758377-Supply-Chain-Management-Icon) <a href="https://dribbble.com/isaacanthonyza"><i class="fa fa-dribbble" aria-hidden="true"></i> Isaac</a>
