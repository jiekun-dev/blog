---
title: 短链生成系统设计——Counter+ZooKeeper+Base62
author: 小黄鸭
type: post
date: 2020-02-15T05:02:40+00:00
excerpt: 解决分布式的短链ID生成问题，系统设计方向的第一次设计尝试。附带号段生成和取号的简单实现，因为取号会有并发读写的问题，顺便试用了一下ZooKeeper的锁。
featured_image: /wp-content/uploads/2020/02/URL-Shorten-System-Design.png
rank_math_seo_score:
  - 10
rank_math_primary_category:
  - 46
rank_math_internal_links_processed:
  - 1
categories:
  - 基础架构
tags:
  - System Design
  - 分布式
  - 短链

---
我们最后设计出的系统架构如图所示，如果想了解最后的结论可以跳到最后一小节。这个系统理论上可以支持大流量的生成请求，分布式部署便于扩展。当然在使用的存储方案性能上没有过多的讨论，因为这次的重点是解决“唯一”、“分布式”的ID问题。

## 准备设计

设计一个系统之前，我们应该对系统的需求有所了解。对于短链系统，首先应该有以下思考：

  * 我们需要什么样的短链接，具体是多短？
  * 短链系统的请求量有多大？
  * 这是个单实例还是分布式系统？

首先我们可以做一些假设，例如参考Twitter有3亿访问/月，我们假设有它的10%，也就是3千万/月，平均每日100万。

然后再来假设生成的短链，一般格式为`domain/unique_id`，例如`s-url.com/D28CZ63`，我们假设Unique ID的长度最多为7位。

下面我们根据这些假设条件来完成这个系统的设计。

## 数据量计算

根据上面的假设，首先每个原始URL可以按照2KB估算（2048字符），而短URL可以按照17Byte估算；我们可能还需要记录创建时间和过期时间，分别是7Byte。因此可以大致估算每行记录的大小应该为2.031KB。

我们一共有30M月访问，`30M * 2.031KB = 60.7GB`，**每月约60GB数据**，因此一年内估算为**0.7TB**，**5年3.6TB**数据量。

## 唯一ID算法

我们需要的是一个短的（7位）唯一ID生成方案。考虑Base62和MD5，Base62即使用0-9A-Za-z一共62个字符，MD5使用0-9a-z，一般输出长度为32的字符串。

使用MD5的话，因为输出长度固定，我们可能需要截取其前7位来作为唯一ID，这种情况下，首先不同的输入可能会输出相同的MD5，其次，不同的MD5的前7位也可能是相同的。这样的话会产生不少的Collision，需要业务上进行保障。而使用MD5的好处，也恰恰是如果不同用户提交相同输入，那么可以得到相同的ID而不需要重复生成新的短链ID，但是同样需要业务进行处理和保证。

对于Base62，每一位有62个可能字符串，7位则是`62^7=3521614606208`种组合，每秒产生1000个ID的话也足够使用110年。同时在短URL的要求上，Base62接受输入，产生的输出长度会根据输入变化，因此不需要进行截取，而只需要想办法将7位ID的所有情况消耗完毕就可以满足大部分场景的要求。Base62伪代码如下：

```
f base62_encode(deci):
    s = '0-9A-Za-z'
    hash_str = ''
    while deci > 0:
        hash_str = s[deci % 62] + hash_str
        deci /= 62
    return hash_str

```
## 存储选择

一般我们会考虑使用RDBMS比如MySQl，或者NoSQL比如Redis。在关系型数据库中，横向扩展会比较麻烦，例如MySQL进行分表和分库，我们可能需要多个实例，而扩展需要一开始就设想好，但是这一点在NoSQL中会相对比较容易，例如使用一个Redis的Cluster，或许向里面添加节点会相对容易一点。而使用NoSQL我们可能需要考虑数据的最终一致性，还有数据的持久化等问题。

同时根据业务场景，从性能上考虑如果在高峰期有大量短链生成请求需要写入到MySQL或许表现会比Redis差一些。

对于将“长URL-短URL”的映射关系写入数据库的步骤，重点是确保这个短URL没有被其他长URL使用过。如果使用过，那么你需要想办法使用新的字符串生成这个短URL。

先来想一下，这是一个两步操作，首先查询是否存在，然后写入。如果这是个串行，那么是可行的。如果这是一个并行操作，很显然，你可能查询的时候发现没有存在这个短URL，而其他Session也查到了同样的结果，最后大家都认为可以写入，然后写入过程中晚写的一方就会出问题。

在RDBMS中我们可能可以通过一些提供的方法来解决这个问题，例如`INSERT_IF_NOT_EXISTS`，但是在NoSQL中是没有这些方法的，因为它的设计是要实现最终一致性，所以不会提供这种支持。

## 基于以上分析和假设的方案

我们需要确定的内容主要是：

  * 生成算法
  * 存储选择

目前罗列出来的方案主要包括：MD5，Base62，以及MySQL和Redis。

如果使用MD5的话，需要使用能够解决哈希冲突的RDBMS，因为这个步骤在NoSQL上处理比较麻烦，所以会有MD5+MySQL的组合。这套组合实际上性能并不太满足需求，并且在扩展上会相对另外一组组合难度大些。

那么另外一种就是我们打算使用的方案：Base62+Redis。如何将7位Base62的所有情况都用尽，我们可以采用一个计数器，从0-62^7的数字转为Base62，作为短链ID使用。这个方案在单实例上是很容易的，并且可以保证冲突问题。那么如何实现它的可扩展性呢？

在接入大流量的情况下，我们必然需要部署多点的ID生成服务，那么根据思路，我们需要对应的计数来转换成Base62的唯一ID，如果不同的服务拿到了同样的计数，那么就会生成相同的ID，造成冲突，且因为分布式的部署，仍然能够正常写入。

因此现在问题转化为如何让不同的服务拿到正确的计数。因为总的数字段是已知的（0-62^7），一个很简单的方法就是我们提前将这些数字进行分段，每个ID生成服务都拿到不同段的数字本地使用。例如：

```
0-100000
100001-200000
200001-300000
300001-400000
400001-500000
500001-600000
...

```
当服务的计数消耗完毕后，继续向计数分配的服务请求下一段可用的数字。例如目前有3个ID生成服务A、B、C，在最初的分配中A拿到了0-100000号码段，B拿到了100001-200000，C拿到了200001-300000。当A使用完之后，询问分配服务，拿到下一段300001-400000。如此即可解决计数器分布式部署的问题。

在数字段分配的业务场景，很容易想到使用ZooKeeper实现，因为ZooKeeper是分布式架构，保障单点故障时仍然可以正确地分配计数号码，这样我们不用重复造轮子实现自己的高可用的分发服务。

## 2020-03-03：号段生成与获取

补充这一段主要因为很想尝试一下，ZK对于自己来说还是太过陌生，于是就花了点时间测试。

### 初始化

```
import json
from kazoo.client import KazooClient

zk = KazooClient(hosts='127.0.0.1:2181')
zk.start()
exist = zk.exists("/url-shortener")
if exist:
    quit()

# Node not exist. Initializing with counter range
zk.create("/url-shortener")

uid_length = 7
start = 62 ** uid_length
end = start + 100000
data = json.dumps({
    "start": start,
    "end": end
}).encode("utf-8")
zk.create("/url-shortener/range-", value=data, sequence=True)

children = zk.get_children("/url-shortener")
int(children)

```
连接上本地的ZK，如果目录已经存在，说明已经完成过初始化了，直接结束。

初始化过程也就是根据配置参数初始化对应的起始数值，因为我需要一个7位长度的ID，对应base62的数值起始位置是`62 ** 7`。号段长度可以分配，号段太长的话，如果有应用的节点挂掉，则有可能浪费掉一段；号段太短，则应用节点需要频繁向ZK询问取号。这里设置为100000。

拿到号段始末后，初始化一个顺序的znode，顺序是因为我可以提前生成一堆znode保存数据，Client直接取即可。不过这里测试过62^7-62^8的范围实在是太大了，按照区间长度为100000来分配的话需要预先初始化`(62^8 - 62^7) / 100000 = 2148184909`个节点，完全没有必要。先初始化1个节点，然后让Client取的时候自动生成下一段即可。

### 取号

```
import json
import os
import time
import random

from kazoo.client import KazooClient
zk = KazooClient(hosts='127.0.0.1:2181')
zk.start()
lock = zk.Lock("/url-shortener-lock", "get_range")

def get_range():
    with lock:
        exist = zk.exists("/url-shortener")
        if not exist:
            print("ZK path not exist!")

        children = zk.get_children("/url-shortener/")
        if children and len(children) == 1:
            node_path = "/url-shortener/%s" % children[0]
            data, stat = zk.get(node_path)
            seq_range = json.loads(data.decode("utf-8"))
            start = seq_range["start"]
            end = seq_range["end"]
            print("Process %d gets range: %d-%d" % (os.getpid(), start, end))
            # generating next range
            start = end + 1
            end = start + 100000
            data = json.dumps({
                "start": start,
                "end": end
            }).encode("utf-8")
            zk.create("/url-shortener/range-", value=data, sequence=True)
            zk.delete(node_path)
            children = zk.get_children("/url-shortener")
        else:
            print("Running out of range!")
    return

if __name__ == '__main__':
    get_range()

```
取号逻辑也很简单，暂时还没有完善异常处理。检查指定的Path下是否有znode可用，如果没有的话可能是异常或号段用完，对应处理一下即可。然后获取znode的值，作为自己目前的号段范围。

生成下一个号段，那么就用上一段的末尾加上固定长度就行，同样创建一个znode，删除旧的节点。

原来确实是这么考虑的，不过发现，如果有多个Client同时尝试获取range，因为读znode和写znode和删除znode并不是原子操作，多个节点读到znode信息后，用作自己的range（已重复），创建新znode（创建了多个顺序的，内容相同的znode），删除znode（只有1个会操作成功，其余出错），所以数据都是错的。

既然用到了ZK，那么顺便来用一下ZK的分布式锁，原理就是创建对应的用作锁的顺序znode，序号靠后的client需要注册对应的watcher等待前一把锁释放（删除），当前面的锁释放（或挂掉，临时znode自动删除）触发watcher后，后面的client获得锁。kazoo已经为我们实现好了对应的逻辑，在执行过程中，可以看到`/url-shortener-lock`路径下的znode：

```
[zk: localhost:2181(CONNECTED) 121] ls /url-shortener-lock
[1f5cda205022418c8b7db432a667dfd5__lock__0000000342, 9dffb44d40a74bbc969c4c64616b77d2__lock__0000000344, a829fbf9835c46bbaa4af4dda02aa9c1__lock__0000000343]

```
我们运行了3个并行的进程，因此有3个znode被创建，只有其中1个能执行临界区的代码。执行结果就是各自获取到了不同的range，符合期望：

```
(zk) duck@duck-MS-7A34:~/src/apache-zookeeper-3.5.7-bin/script$ ./run_consume.sh 
Process 14188 gets range: 3521648606548-3521648706548
Process 14186 gets range: 3521648706549-3521648806549
Process 14187 gets range: 3521648806550-3521648906550
All done
```