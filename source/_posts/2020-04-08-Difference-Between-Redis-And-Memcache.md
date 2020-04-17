---
title: Difference Between Redis And Memcache
date: 2020-04-08 16:02:02
categories:
  - tech
tags:
  - In-Memory Data Storage Systems
---

这里主要介绍一下Redis和Memcache的一些异同

<!-- more -->

# Redis & Memcache，都是基于内存的存储系统，粗略的说

1. 操作上
  Redis支持服务端的数据操作，（比如incr），但是Memcache不行，需要将值从服务器上取下来，运算，之后再存回Memcache
2. 内存使用率上
  Memcache的内存使用率比Redis更低。（和内存分配策略，和数据结构有关）
3. cpu使用上
  Redis仅支持单核cpu，但是Memcache支持多核cpu

# 再详细点


## 1. Redis使用很多的这个数据集合，比如下图中列的
+ string
+  hash
+  list
+  set
+  sorted set
![RedisDataStructure](/images/2020-04-08-Difference-Between-Redis-And-Memcache/RedisDataStructure.png)

## 2. 内存管理机制
性能：
在redis中，允许存储的内容，比实际的物理内存要更大，当物理内存用完之后，就会使用swap分区。swap操作明显会造成io阻塞[关于VirtualMachine](https://redis.io/topics/virtual-memory)
而memcache不行，仅支持物理内存操作，所以，从纯速度上来说，memcache的性能要比redis要更好。
机制:
memcache的内存分布如下所示，其实类似于linux文件系统，存在一定的内存浪费
![MemcacheMemoryScheme](/images/2020-04-08-Difference-Between-Redis-And-Memcache/MemcacheMemoryScheme.png)

而关于redis的内存分配则是根据基础结构来定的，浪费较少，不过这样的分配容易出现内存碎片问题，官方文档资源则详细得多，我这里直接粘一下好了，一共4点
+ 当key被删除的时候，redis不会立刻释放内存，如果使用了5G内存，然后释放了2G，那么实际上的占用依然会是5G
+ 基于第一点，需要预见到实际的redis占用，如果高峰期可能使用到10G，那么就需要提前预备10G的内存（貌似是废话）
+ 如果申请了5G之后，释放了2G，之后又需要使用内存，那么redis会基本上会重复使用这2G的资源（貌似也是废话）
+ 这样的话，碎片率的当 峰值/平时值 比率较高的时候，碎片率会比较高
参考部分：[memory-optimization](https://redis.io/topics/memory-optimization)

## 3. 数据持久化
memcache简单粗暴，直接不支持
redis支持，两种模式：rdb快照&AOF文件

## 4. 集群化管理
+ memcache本身对于集群化是无感知的，就是说，可以启动多个memcache节点，完了在客户端通过算法，将数据hash到各个节点上，通过client端的处理来做一个集群
![MemcacheCluster](/images/2020-04-08-Difference-Between-Redis-And-Memcache/memcacheCluster.png)
+ redis的集群则可以直接在服务端做处理
![MemcacheCluster](/images/2020-04-08-Difference-Between-Redis-And-Memcache/redisCluster.png)
