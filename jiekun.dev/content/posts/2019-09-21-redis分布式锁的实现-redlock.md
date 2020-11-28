---
title: Redis分布式锁的实现——RedLock
author: 小黄鸭
type: post
date: 2019-09-21T10:36:07+00:00
excerpt: '基于单实例Redis使用SETNX的简易分布式锁实现 & 基于Redis集群的分布式锁RedLock实现'
featured_image: /wp-content/uploads/2019/09/redlock.png
rank_math_primary_category:
  - 29
rank_math_seo_score:
  - 17
rank_math_internal_links_processed:
  - 1
categories:
  - 最佳实践
tags:
  - Redis
  - RedLock
  - 分布式
archieved: Arc'd

---
![](../2019/09/redlock-py.png")

文章配图是RedLock Python实现的作者[optimuspaul][1]的头像。

在分布式应用中经常会存在一些并发的问题，当多个请求想要处理同样的资源时，比如某个操作需要读取资源，根据读取结果进行修改，再写入，若这个步骤没有原子性，多个请求同时进行这样的操作，那就会变得非常混乱。通常来说可以依靠Redis来实现简单的分布式锁机制。

## Redis分布式锁SETNX

基于之前的描述，当多个请求需要处理同样的内容时，我们为了确保只有其中一个请求被执行，那么可以借助Redis生成一把锁。并发请求向Redis申请锁，申请成功的人占用本次操作的执行权。因为Redis单线程的特性，一次只处理一个请求，因此后续申请锁的操作都可以被排除。具体代码如下：

```
t resource_name value ex 5 nx</code></pre>

```
意味着：

  * 插入一个名为resource_name的键，它的值为value
  * TTL 5秒
  * 只有这个键不存在的情况下才能插入成功

因为nx参数的存在，过期时间内执行相同的操作不会返回1，意味着插入不成功（没申请到锁）。

### 简易实现的问题

现在来看一下上面的设计有什么问题。

##### 超时

假设现在clinet1拿到了锁，在执行一段时间后超过了设定的ttl，锁过期，client2向Redis执行语句申请锁，因为锁不存在所以client2成功申请到了锁。此时client1仍在继续执行未完成的操作，相当于存在client1和client2共同操作资源的行为。

对于这种问题，当前的锁机制是无法解决的，需要：

  * 避免在分布式中处理超长的任务
  * 适当延长TTL并在执行完后及时对锁DEL
  * 业务上进行处理
  * 取消TTL，改为由client控制锁的DEL

对于最后一种方案，因为client控制锁的归还（del），如果在执行del命令时发生异常，redis服务器没有接收到，或者client出错，没有执行del，将会造成死锁，因为锁会持续存在，其他client不能够正常获取到锁。

##### 锁被其他线程释放

对于上面的设计，不安全的地方在于，若其他线程执行`del resource_name`操作，那么看起来可以立刻获取一把新锁，从而达到无视锁机制的效果。

为此，在锁的设计上，value需要设计成一个unique值，在del操作前，业务上需要确认del的键的值是否匹配，若不匹配，应该取消del操作。

因此，简单的分布式锁的使用应该修改为：

```
t resource_name unique_value px 5000 nx</code></pre>

```
## Redis集群分布式锁RedLock

现在继续来考虑一些简易锁的异常问题：

  * client1申请到了锁，Redis记录了这把锁
  * Redis服务发生异常退出
  * Redis服务恢复，但是丢失数据（假设锁没有及时持久化）
  * client2尝试申请锁，因为Redis没有锁存在，因此申请成功
  * client1、client2一起操作资源

由于服务的不可靠，简易锁的实现在特殊情况下会失效。为此，Redis作者提供了一种基于Redis集群的分布式锁——[RedLock][2]：

<blockquote class="wp-block-quote">
  <p>
    We propose an algorithm, called Redlock, which implements a DLM which we believe to be safer than the vanilla single instance approach. We hope that the community will analyze it, provide feedback, and use it as a starting point for the implementations or more complex or alternative designs.
  </p>
</blockquote>

 [1]: https://github.com/SPSCommerce/redlock-py/commits?author=optimuspaul
 [2]: https://redis.io/topics/distlock