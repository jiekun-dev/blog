---
title: Elasticsearch节点选举、分片及Recovery
author: 小黄鸭
type: post
date: 2020-03-14T07:58:10+00:00
excerpt: Elasticsearch通过Bully算法选举临时Master、正式Master；Allocation模块负责分片选举，Master分发Primary Terms和维护in-sync列表；副分片的两阶段Recovery流程，通过_version保证恢复过程中的数据一致性
featured_image: /wp-content/uploads/2020/03/Elasticsearch.png
rank_math_primary_category:
  - 2
rank_math_seo_score:
  - 10
rank_math_internal_links_processed:
  - 1
categories:
  - Elasticsearch
tags:
  - Elasticsearch
  - 分布式
  - 分片
  - 恢复
  - 选举

---
隔了挺长一段时间没有更新，主要是因为近段时间忙于业务和刷题，想来刷题除了Po题解和Explanation也是没有什么特别之处，除非钻研得特别深入，所以（@#$%^&找理由）。

# 关于Elasticsearch

Elasticsearch其实官网的文档特别齐全，所以关于用法没有什么特别好写的，看博客不如RTFM。但是文档特别全的情况下，很多时候又缺少对一些具体细节的描述，一句话说就是不知其所以然。所以今天写的博客内容理应是无关使用的，不涉及命令与操作，大概会更有意义一些吧。

# 概述

以Elasticsearch（下称ES）集群启动过程作为索引来展开，ES想要从Red转为Green，需要经历以下过程：

  * 主节点选举。集群启动需要从已知的活跃机器中选取主节点，因为这是PacificA算法的思想——主从模式，使用Master节点管理元信息，数据则去中心化。这块使用类似Bully的算法。
  * 元信息选举。主节点确认后，需要从各节点的元信息中获取最新版本的元信息。由Gateway模块负责。
  * 主副分片选举。由Allocation模块负责，各分片的多个副本中选出主分片和副分片，记录他们所属的节点，重构内容路由表。
  * 恢复分片数据。因为启动可能包含之前没有来得及刷盘的数据，副分片也可能落后于新选出的主分片。

# Bully算法与主节点选举

## Bully算法

特地查了一下Bully的意思——“仗势欺人者，横行霸道者”，所以这个霸道选举算法如其名，简单暴力地通过**选出ID最大的候选者**来完成。在Bully算法中有几点假设：

  * 系统是处于同步状态的
  * 进程任何时间都可能失效，包括在算法执行过程中
  * 进程失败则停止，并通过重新启动来恢复
  * 有能够进行失败检测的机制
  * 进程间的消息传递是可靠的
  * 每个进程知道自己的ID和地址，以及其他所有的进程ID和地址

它的选举通过以下几类消息：

  * 选举消息：用来声明一次选举
  * 响应消息：响应选举消息
  * 协调消息：胜利者向参与者发送胜利声明

设想以下场景，集群中存在ID为1、2、3的节点，通过Bully算法选举出了3为主节点，此时之前因为网络分区无法联系上的4节点加入，通过Bully算法成了新的主节点，后续失联的5节点加入，同样成为新主节点。这种不稳定的状态在ES中通过优化选举发起的条件来解决，当主节点确定后，在失效前不进行新一轮的选举。另外其他分布式应用一样，ES通过Quorum来解决脑裂的问题。

## Elasticsearch主节点选举

ES的选举与Bully算法有所出入，它选举的是**ID最小**的节点，当然这并没有太大影响。另外目前版本中ES的排序影响因素还有集群状态，对应一个状态版本号，排序中会优先将版本号高的节点放在最前。

在选举过程中有几个概念：

  * 临时Master节点：某个节点认可的Master节点
  * activeMasters列表：不同节点了解到的其他节点范围可能不一样，因此他们可能各自认可不同的Master节点，这些临时Master节点的集合称为activeMasters列表
  * masterCanditates列表：所有满足Master资格（一般不满足例原因如配置了某些节点不能作为主节点）的节点列表
  * 正式Master节点：票数足够时临时Master节点确立为真正Master节点

某个节点ping所有节点，获取一份节点列表，并将自己加入其中。通过这份列表查看当前活跃的Master列表，也就是每个节点认为当前的Master节点，加入**activeMasters列表**中。同样，通过过滤原始列表中不符合Master资格的节点，形成**masterCandidates列表**。

如果activeMasters列表不为空，按照ES的（近似）Bully算法选举自己认为的Master节点；如果activeMasters列表空，从masterCandidates列表中选举，但是此时需要判断当前候选人数是否达到Quorum。ES使用具体的比较Master的逻辑如下：

```
/**
 * compares two candidates to indicate which the a better master.
 * A higher cluster state version is better
 * 比较两个候选节点以得出更适合作为Master的节点。
 * 优先以集群状态版本作为排序
 *
 * @return -1 if c1 is a batter candidate, 1 if c2.
 * @c1更合适则返回-1，c2更合适则返回1
 */
public static int compare(MasterCandidate c1, MasterCandidate c2) {
    // we explicitly swap c1 and c2 here. the code expects "better" is lower in a sorted
    // list, so if c2 has a higher cluster state version, it needs to come first.
    // 先比较版本
    int ret = Long.compare(c2.clusterStateVersion, c1.clusterStateVersion);
    if (ret == 0) {
        // 比较节点
        ret = compareNodes(c1.getNode(), c2.getNode());
    }
    return ret;
}

/** master nodes go before other nodes, with a secondary sort by id **/
 private static int compareNodes(DiscoveryNode o1, DiscoveryNode o2) {
    if (o1.isMasterNode() && !o2.isMasterNode()) {
        // 如果o1是主节点
        return -1;
    }
    if (!o1.isMasterNode() && o2.isMasterNode()) {
        // 如果o2是主节点
        return 1;
    }
    // ID比较
    return o1.getId().compareTo(o2.getId());
}

```
确定之后进行投票，ES的投票是通过发送Join请求进行的，票数即为当前连接数。

如果临时Master为当前节点，则当前节点等待Quorum连接数，若配置时间内不满足，则选举失败，进行新一轮选举；若满足，发布新的clusterState。

如果临时Master节点不是本节点，则向Master发送Join请求，等待回复。Master如果得到足够票数，会先发布状态再确认请求。

# 主副分片选举与Allocation模块

分片的决策由Master节点完成，需要确认的内容包括：

  * 哪些分片应该分配到哪个节点上（平衡）
  * 分片的多个副本中哪个应该成为主分片（数据完整）

## allocators

Allocation模块中，allocators负责对分片作出优先选择，例如：

  * 平衡分片，节点列表按照它们的分片数排序，分片少的靠前，优先将新分片分配至靠前节点
  * 主副分片，按照：节点上如果有完整的分片副本，主分片才能够指定到这个节点；节点上如果有（不一定需要完整）分片副本，副分片可以优先分配在这个节点（然后从主分片恢复数据）。
  * 具体包括：
      * primaryShardAllocator：找到拥有分配最新数据的节点
      * replicaShardAllocator：找到拥有这个分片数据的节点
      * BalancedShardsAllocator：找到拥有最少分片个数的节点

## deciders

作出选择后，需要通过deciders判断分片是否真的可以指定在这个节点，例如：

  * 磁盘空间限制
  * 配置限制
  * 避免主副分片落在同一节点
  * 具体包括：
      * SameShardAllocationDecider：避免同节点
      * AwarenessAllocationDecider：分散存储shard
      * ShardsLimitAllocationDecider：同一节点允许同index的shard数目
      * ThrottlingAllocationDecider：recovery阶段的限速配置影响
      * ConcurrentRebalanceAllocationDecider：重新分片的并发控制
      * DiskThresholdDecider：磁盘空间
      * RebalanceOnlyWhenActiveAllocationDecider：是否所有shard都处于active状态
      * FilterAllocationDecider：接口动态设置的限定参数
      * ReplicaAfterPrimaryActiveAllocationDecider：主分片分配完毕才开始分配副分片
      * ClusterRebalanceAllocationDecider：集群中active的shard的状态

## 主分片选举

分片经过指定节点后有allocation id，并且有inSyncAllocationIds列表记录哪些分片是处于“in-sync”状态的。主分片的选举通过是否处于in-sync列表来进行。

在历史版本中，分片有对应的版本号，但是如果使用版本号进行选举，如果拥有最新数据版本的分片还未启动，那么就会有历史版本的分片被选为主分片，例如只有一个活跃分片时它必定会被选为主分片。

通过将in-sync列表的分片遍历各个decider，如果有任一deny发生，则拒绝本次分配。决策结束之后可能会有多个节点，取第一个节点上的分片作为主分片。

## 分片模型

ES中使用Sequence ID标记写操作，以得到索引操作的顺序。现在考虑这种情况：由于网络原因，主分片产生的SID=145的操作转发到副分片上，但是没有传达成功，此时主分片被另一个副分片取代，也产生了一个SID=145（因为这个副分片最新的SID是144）的操作，转发给其他副分片。转发过程中，原来网络分区的主分片恢复，它的旧SID=145操作继续发送给其他副分片，那么分片副本中就有部分收到了旧主发的145操作，部分收到了新主发的145操作。

因此，除了Sequence ID以外，ES使用Primary Terms来标记主分片，每次新主分片产生时，Primary Terms加1，副分片会拒绝旧的Primary Terms发来的操作。

主节点为分片分配Primary Terms、Allocation ID，其中各个满足in-sync状态的分片的Allocation ID构成inSyncAllocationIds列表；Sequence ID由主分片为写操作分配，副分片拒绝Primary Terms+Sequence ID落后的操作。

# 分片数据Recovery

ES（大致的）存储模型在官网上有描述有图，所以就不多费时间描述了。

## 主分片Recovery

主分片因为处于in-sync list中，需要恢复的数据只有未进行fsync刷盘的部分，也就是refresh之后，变得可被索引，但是没有进行flush生成新的commit point持久化到磁盘的部分。这部分数据在translog中，因此需要将数据从translog进行恢复。

经过一系列的校验（是否主分片、分片状态是否异常等）工作后，从分片读取最后一次提交（commit）的段（segment）信息，获取其中版本号，更新当前索引版本。然后验证元信息中的checksum和实际值是否匹配，避免分片受损。

根据最后一次commit的信息，确认translog中哪些数据需要进行reply，执行具体的写操作，结束后进行refresh，和正常写操作一样，让数据转移到文件系统缓存中，变得可被索引到，但是没有fsync。

最后进行一次refresh更新分片状态，恢复完毕。

## 副分片Recovery

副分片恢复需要根据当前数据状态（进度）决定，如果Sequence ID满足，可以直接从主分片的Translog中恢复缺失部分；如果不满足，需要拉取主分片的Lucene索引和Translog进行恢复。

主分片一般先Recovery，结束后接受新业务的操作，如何保证副分片需要的Translog不清理？在最初的1.x版本中，ES阻止refresh操作保留translog，但是这样会产生很大的translog；在2.0-5.x版本中，引入了translog.view的概念，translog被分为多个文件，维护一个引用文件的列表，同时recovery通过translog.view获取这些文件的引用，因为文件引用的存在translog不能被清理，直到view关闭（没有引用）。6.0版本中引入了TranslogDeletingPolicy概念，维护活跃的translog文件，通过将translog做快照来保持translog不被清理。

副分片的恢复由两个阶段构成：

  * phase1：在主分片上获取translog保留锁，此时translog不会被清理；将Lucene索引做快照，数据复制到副本节点。完成后，副分片可以启动Engine开始接受请求。
  * phase2：对translog做快照，这部分包含了从phase1开始到执行translog快照期间的新增数据，发送到副分片进行reply。

前面提过，如果可以基于SID进行恢复，跳过phase1；如果主副分片有同样的syncid且doc数相同，跳过phase1。

什么是syncid？当分片5分钟（可配置）没有写入操作就会被标记为inactive，执行synced flush，生成一个syncid，相同syncid意味着分片是相同的Lucene索引。

## 恢复过程中的主副分片一致性

恢复时，因为主副分片恢复时间不一致，主分片先进行Recovery，然后副分片才能基于主分片进行Recovery，所以主分片可以工作之后，副分片可能还在恢复中，此时主分片会向副分片发送写请求，因此恢复reply与主分片可能会同时（或者不按发生顺序）对同一个doc进行操作。ES中通过doc的版本号解决这个问题，当收到一个版本号低于doc当前版本号的操作时，会放弃本次操作。对于特定的doc，只有最新一次操作生效。

# 总结

Elasticsearch是个易用又复杂的分布式项目，其中很多分布式相关的设计和思想都值得学习和借鉴。在拉取代码时发现项目体积接近1GB：

```
uck@duck-MS-7A34:~/study/tmp$ du -sh elasticsearch/
949M    elasticsearch/

```
因此其中很多模块都没有了解清楚，希望以后可以保持学习的新鲜感，继续摸索更多的内容。