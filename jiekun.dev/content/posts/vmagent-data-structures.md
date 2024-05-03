---
title: "Victoriametrics 中的持久化数据结构 (Part 1): vmagent"
date: 2024-05-03T11:33:36+08:00
author: blog@jiekun.dev
type: post
category: 
 - Observability
comments: true
toc: true
---

{{<admonition type=note title="Medium">}}
This blog post is also available in **English**: 
- [Persistent Data Structures in Victoriametrics (Part 1): vmagent]()
{{< /admonition >}}

{{<admonition type=info title="Series Introduction">}}
VictoriaMetrics 是一个开源的高性能时序数据库，作为 Prometheus 及其长期存储方案（如：Thanos、Cortex）的平替，有很多企业已经将它大规模部署在生产环境。尽管 VictoriaMetrics 有非常详细的使用文档和示例，关于它的数据是如何存在于磁盘上的讨论却非常少。

鉴于社区中的活跃的成员越来越多，一份介于使用说明和源代码之间的文档可以更好地帮助他们过渡到贡献者。

这个系列旨在让读者了解 VictoriaMetrics 的磁盘数据是如何组织的。读者不需要具备任何 Go 语言的知识，但是最好对 VictoriaMetrics 的组件有简单的了解。

{{< /admonition >}}

## vmagnet 简介
**vmagent** 是一个轻量的 Agent，用于像 Prometheus 一样**采集**应用暴露的指标，或者作为 Receiver **接收** Remote-Write Compatible、InfluxDB line 协议的数据推送。

vmagent 可以对数据进行加工，例如 Relabeling、Sorting。最后将数据 remote write 至 VictoriaMetrics 或其他协议兼容的目标。

![](../202405-vm-series/vmagent.png)

## FastQueue
现在让我们关注 vmagent 内部，采集到的数据首先会在内存中不断追加给各个 Remote-Write 的对象（图中未画出）。等到积累了一定的数据量或者等待一定的时间间隔（默认 1 秒）后，它们会被 Flush 到一个 Remote-Write 对象持有的 FastQueue。每个 Remote-Write 对象还会持有多个 worker，负责消费 FastQueue 中的数据，推送至 Remote-Write 的地址。

![](../202405-vm-series/fast_queue.png)

FastQueue 在理想情况下通过 `channel` 数据结构将数据传递给 worker（这是 Go 语言中很典型的协程间通信），这是完全基于内存的。但 `channel` 能够缓冲的数据是有限的，如果网络异常，或者 Remote-Write 的目标没有足够能力处理数据，`channel` 很快就会被填满。

这时候 FastQueue 会选择将数据写入磁盘进行缓冲。

![](../202405-vm-series/fast_queue_2.png)

基于这些介绍，我们可以很容易总结出以下要点：
- 一个 Remote-Write 配置对应一个 Remote-Write 对象；
- 一个 Remote-Write 对象持有 1 个 FastQueue 和多个 workers；
- 一个 FastQueue 对应一个 `channel` 和一个磁盘文件（目录）。

## On-Disk Data Structure
如果你在本地运行过 vmagent，你可以很容易查看到它为每个 Remote-Write 生成的文件目录。例如：
```
./vmagent-remotewrite-data
└── persistent-queue
    ├── 1_E3C1E1E1733E59E4
    │   ├── 0000000000000000
    │   ├── flock.lock
    │   └── metainfo.json
    └── 2_740390E5C841CCAC
        ├── 0000000000000000
        ├── flock.lock
        └── metainfo.json
```

vmagent 根据 Remote-Write 的**配置顺序**以及 URL 生成目录名：
```go
	...
	// Hash value of `URL`. e.g.: E3C1E1E1733E59E4
	h := xxhash.Sum64([]byte(URL.String()))

	// Index + Hash value. e.g.: 1_E3C1E1E1733E59E4
	queuePath := filepath.Join("./vmagent-remotewrite-data", "persistent-queue", fmt.Sprintf("%d_%016X", argIdx+1, h))
	...
```

如果在磁盘上有待消费的数据，且 vmagent 意外退出，那么它在重启时仍然能够读取相同的目录继续进行消费。

{{<admonition type=danger title="思考题 1">}}
1. 如果意外退出后，修改了其中一个 Remote-Write 的 URL 再启动，会发生什么？
2. 问题 1 如果将 “修改” 换成 “移除” 会有什么不同的结果？
3. 问题 2 中 “移除” 第一个 Remote-Write 配置和最后一个 Remote-Write 配置会有差别吗？
{{< /admonition >}}

每个 Remote-Write 的目录下有 3 个文件，分别是数据文件（`0000000000000000`）、元数据文件（`metainfo.json`）和锁（`flock.lock`）。

其中，元数据文件记录了数据文件的读写偏移量：
```
{"Name":"2:secret-url","ReaderOffset":0,"WriterOffset":0}
```

而数据文件中的存放了等待 remote write 的 `[]byte`。要注意的是，他们并不是 Time-Series 数据经过序列化后得到的 `[]byte`。

vmagent 支持 Prometheus 使用的 Snappy 压缩算法，也支持 zstd 压缩算法。vmagent 在**启动时**会根据启动参数或自动协商确认每个 Remote-Write 需要使用的压缩算法。Remote-Write 数据进入 FastQueue 前，需要：
1. 按照 Remote-Write Protocol 序列化成二进制 `[]byte`；
2. 使用 Snappy 或 zstd 算法将其压缩成新的 `[]byte`

![](../202405-vm-series/data_compression.png)

最后才会被发送出去或写入本地数据文件。

{{<admonition type=danger title="思考题 2">}}
假设 Remote-Write 目标不支持 zstd 压缩算法，vmagent 会发送 Snappy 算法压缩后的数据。如果此时：
1. Remote-Write 目标停机 10 分钟进行升级，vmagent 将数据缓冲至本地文件；
2. Remote-Write 目标升级完成，支持新的 zstd 压缩算法；

那么现在：
1. Remote-Write 目标会收到什么算法压缩的数据？
2. 假设 Remote-Write 目标启动后 vmagent 也进行重启，Remote-Write 目标会收到什么算法压缩的数据？
{{< /admonition >}}

## Further Reading
你可以在以下位置找到关键的代码：
- Presistente Queue Folder Path Generation: https://bit.ly/3y2FxEx
- Data Compression Before Writing to FastQueue: https://bit.ly/4dlUeCN
- FastQueue Fast Path & Slow Path: https://bit.ly/4aYkD7X
- Remote-Write worker: https://bit.ly/3JLrfL6
