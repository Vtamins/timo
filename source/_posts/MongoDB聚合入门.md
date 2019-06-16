---
title: MongoDB聚合入门
date: 2019-06-16 13:27:49
tags:
  - mongo
  - 数据库
---



[TOC]

## Why？

mongoDB的查询非常的方便，但是在开发的过程中总会遇到一些特殊的查询需求。

比如：

### 需求

有1000条访问记录，每一条的数据结构如下：

```json
{
	"ussername": "user1",
	"resource": "api/v1/test1",
	"timestamp": 1560681124,
}
```

查询用户a，最近10条访问记录，要求每个`resource`只取最近访问的一次。

此时，这种复杂的查询就比较适合MongoDB的聚合查询来做。

## What?

官方文档：

```
Aggregation operations process data records and return computed results. Aggregation operations group values from multiple documents together, and can perform a variety of operations on the grouped data to return a single result. MongoDB provides three ways to perform aggregation: the aggregation pipeline, the map-reduce function, and single purpose aggregation methods.
```

大意如下：mongo的聚合从多个文档分组进行处理，最终返回一个结果。并且包括三种：

1. pipeline
2. MapReduce
3. 单一目的聚合操作

### Pipeline

pipeline就是按照用户定义的stage，一个阶段一个阶段的执行，上一个阶段的输出作为下一个阶段的输入。

![image-20190616184434735](http://tuchuang.vtamins.cn/20190616191225)

### MapReduce

MapReduce把数据分为多个过程，这个思想请看Google的MapReduce论文，不多做介绍。

[MapReduce论文](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/mapreduce-osdi04.pdf)

### 单一目的聚合操作

使用单一的聚合函数，如：count、distinct等

## How？

对于以上的需求我们可以直接使用Pipeline来做。

分为三个stage:

```json
stage1: 过滤，去除用户1的访问记录
$match: {"username": "user1"}
stage2: 分组，根据用户resource分组，并取第一个
$group: {"resource":{"$first": "$resource"}}
stage3: 排序,根据时间排序，倒序
$sort: {"timestamp": -1}
```

最终的查询参数:

```json
db.records.aggregate([
{"$match": {"username": "user1"}},
{"$group": {"resource":{"$first": "$resource"}}},
{"$sort": {"timestamp": -1}}
])
```

## 总结

通过以上的例子，相信各位对Mongo的聚合已经有了一个初步的了解，通过mongo聚合能够实现一些较为复杂的查询。后续将继续安排其他更深入的内容，敬请期待。