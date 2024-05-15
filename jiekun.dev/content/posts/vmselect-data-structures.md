---
title: "VictoriaMetrics 中的持久化数据结构 (Part 2): vmselect"
date: 2024-05-15T10:41:23+08:00
author: blog@jiekun.dev
type: post
category: 
 - Observability
comments: true
toc: true
---

{{<admonition type=note title="Medium">}}
This blog post is also available in **English**: 
- [Persistent Data Structures in VictoriaMetrics (Part 1): vmselect]()
{{< /admonition >}}

{{<admonition type=info title="Series Introduction">}}
VictoriaMetrics 是一个开源的高性能时序数据库，作为 Prometheus 及其长期存储方案（如：Thanos、Cortex）的平替，有很多企业已经将它大规模部署在生产环境。尽管 VictoriaMetrics 有非常详细的使用文档和示例，关于它的数据是如何存在于磁盘上的讨论却非常少。

鉴于社区中的活跃的成员越来越多，一份介于使用说明和源代码之间的文档可以更好地帮助他们过渡到贡献者。

这个系列旨在让读者了解 VictoriaMetrics 的磁盘数据是如何组织的。读者不需要具备任何 Go 语言的知识，但是最好对 VictoriaMetrics 的组件有简单的了解。

{{< /admonition >}}

## vmselect 简介
**vmselect** 是 VictoriaMetrics 的查询组件，通常位于 Load Balancer 后面，负责向多个 vmstorage 节点发送查询请求，并将数据聚合、缓存、返回给用户。

![](../202405-vm-series/vmselect.png)



## Further Reading
你可以在以下位置找到关键的代码：
