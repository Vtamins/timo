---
title: Avro模式演化
date: 2019-07-06 20:12:19
tags:
  - Avro
  - 大数据
  - 序列化框架
---

[TOC]

## 背景

Avro作为大数据领域的一个常用的序列化框架之一，具有如下的特点：

1. 高性能：序列化和发序列化的速度快
2. 存储大小优良：序列化后的二进制数据存储空间占用较小
3. 生态良好：作为Hadoop的一个项目，目前得到了大部分的大数据组件的支持，包括但不限于：HDFS、Hive、Flink、Spark、Kafka

同时，Avro在实时流处理领域，几乎成为了一种行业标准。

但是，在实时流处理领域，上游和下游之间对于数据的写和读，往往需要使用对应的Schema来进行，一旦Schema不对，则无法正确的读取数据。但是，上下游之间一旦要进行数据版本的变化，将给系统的稳定性带来挑战。这里我们看一下官方文档的解释：

```
 However, the reader may be programmed to read data into a different schema. For example, if the data was written with a different version of the software than it is read, then fields may have been added or removed from records. This section specifies how such schema differences should be resolved.
```

因此，Avro的Schma演化在实时流处理里面显得非常的重要，这里我们假设你已经有了一定的Avro基础，在这个基础上来介绍Avro的实时流演化规则。

## 演化

### Record

首先看最常用的Record，record支持如下规则：

1. record的名字必须一致；
2. 可以允许fileds的顺序不同，record中的field是按照name进行匹配的；
3. 如果writer的schema中某个field，reader的schema中没有，那么这个field将不会被反序列化出来；
4. 如果reader的schema中包含了一个writer的schema中没有的field，这个field将使用默认值，如果没有默认值将会报错；

### 其他

Arrays要求item的类型一致；

Map要求value的类型一致；

Enums要求name是一致的；

Fixed要求name和size是一致的；

Union要求使用的类型匹配；

### 举例

以Record为例：

Writer的Schema结构如下所示：

```json
{"namespace": "example.avro",
 "type": "record",
 "name": "user",
 "fields": [
     {"name": "name", "type": "string"},
     {"name": "favorite_number",  "type": "int"}
 ]
}
```



Reader的Schema如下：

```json
{"namespace": "example.avro",
 "type": "record",
 "name": "user",
 "fields": [
     {"name": "name", "type": "string"},
     {"name": "age",  "type": "int", "default": 18}
 ]
}
```

数据A被写入是为：`name=avro1,favorrite_number=6`

那么读取出来的数据将会是：`name=avro1,age=18`。

## 总结

Avro Schema演化能够提供有效的数据兼容性，这在流式处理中显得非常有用，不会因为上下游的数据的变化而引起系统崩溃。