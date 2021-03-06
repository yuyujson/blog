---
title: Redis缓存三大问题
permalink: Redis-san-da-wen-ti
date: 2020-02-15 21:55:32
tags:
categories: redis
---

# 前言

> 使用`redis`做缓存一般使用如下方式:
>
> 1. 后台接受到请求
> 2. 查询`redis`看是否存在,存在则直接返回
> 3. 查询数据库,如果查询到则插入`redis`后返回, 否则返回null
>
> 但是, 在大数据量请求的情况下往往存在以下问题

<!--more-->

# 缓存穿透

> 查询一条不存在的数据
>
> 举例: 黑客攻击
>
> 黑客以一个不存在的订单号不停的对数据进行查询, 这会造成我们查询redis后查询不到结果,然后不停的查询数据库

## 规则校验

订单号中添加几位用于校验的值,在请求过来后先进性校验, 如果不存在则直接返回

> 摸到规律后非常容易破解

## 设置返回空对象

在查询数据库后, 如果数据不存在, 则插入一条过期时间为2秒的空对象, 这样2秒内过来的请求就不会进行数据查询

> 1. 如果黑客进行了并发查询呢? 这会造成同一毫秒内大批量数据进行查询数据库
> 2. 如果黑客每次查询的时候将订单号+1 呢?

## 设置返回空对象+redis锁

1. 查询`redis` 存在则返回
2. 加锁, 没有获取锁则等待
3. 查询`redis` 存在则返回
4. 查询数据库
   1. 存在则加入`redis`并返回
   2. 不存在则加入空对象,时间2s
5. 解锁

**说明:**

1. **逻辑2** 中使用的是`redis`锁, 而不是声明一个`ReentrantLock`, `ReentrantLock`需要放在成员变量中, 这会导致锁的颗粒度过大, 会影响到正常的业务逻辑
2. **逻辑4.1** 中再次查询了一次redis是因为当第一个线程获取锁后, 后续线程处于等待状态 , 当第一个线程释放锁时, 数据已经被放在`redis`中了

> 黑客每次查询的时候将订单号+1 这种情况仍然没有解决

## 使用布隆过滤器

见[布隆过滤器](https://www.chenguanting.top/2020/布隆过滤器)

> 360度无死角

# 缓存击穿

> 一条数据`redis`中没有, 数据库中有. 并发情况下瞬间所有的请求都去查询了数据库
>
> 举例:
>
> 商品设置了过期时间为2秒, 在失效的瞬间, 大量请求过来, 查询`redis`没有,全部查询了数据量

## 设置热点数据永不过期

> 会给redis造成一定压力

## redis锁

具体实现和`缓存穿透-设置返回空对象+redis锁`差不多

> 线程等待可能会比较多, 可能会出现问题

## 查询时刷新时间

每次进行查询时刷新下时间, 保证该数据为热点数据

# 缓存雪崩

> 添加缓存时设置了相同的过期时间，导致大批量数据缓存在某一时刻同时失效，全部请求数据库.造成数据库压力过大雪崩

## 随机数过期时间

在添加缓存时使用一个范围内的随机数的过期时间

## 高可用集群

没有什么事情不是加一台服务器不能解决的, 如果不能,就加两台!