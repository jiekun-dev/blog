---
title: "OpenTelemetry, Prometheus, and More: 谁是采集监控指标的最佳选择?"
date: 2024-12-21T01:22:23+08:00
author: blog@jiekun.dev
type: post
category: 
 - Observability
comments: true
toc: true
---

{{<admonition type=note title="Medium">}}
This blog post is also available in **English**: 
- [OpenTelemetry, Prometheus, and More: Which Is Better for Metrics Collection and Propagation?]()
{{< /admonition >}}

## Prometheus 和 Remote Write

Prometheus 是云原生指标监控领域的事实标准。它的工作模式很简单：应用提供 `/metrics` HTTP API，以文本格式暴露指标数据；Prometheus 访问这些 API，将数据采集，然后提供查询 API 进行展示。

虽然 Prometheus 的生态发展出了很丰富的组件，它的核心 Prometheus Server（下称 Prometheus）仍然保持 All-in-One 的设计，提供一个可执行文件，无需任何依赖即可运行，这使得它的安装和部署更加简单。

但是，这也使得 Prometheus 不易扩展。想象一下，一开始的时候，你在一台 2 CPU，4GiB 内存的机器上运行 Prometheus，用它来监控 100 个应用程序，这很容易。很快你需要用它监控 10000 个、100000 个应用程序，Prometheus 需要更多的机器资源，但是单台机器的配置总是有限的；另外，这些应用可能部署于不同的集群，不同的可用区，用单个 Prometheus 来四处收集数据并不高效。

所以，Prometheus 也提供了两个重要功能：
1. Remote Write：将指标数据发送给**远端存储**，例如 Thanos，Cortex，Mimir 和 VictoriaMetrics。
2. Agent Mode：省略查询、告警、本地存储功能，降低 Prometheus 仅用作数据采集 Agent 时的成本。

有了这些功能，面对大量应用时，监控架构可能是如下图这样：

![]()

## OpenTelemetry 和 OTLP

2019 年，OpenTelemetry 诞生，它提供了统一、开源的可观测性标准，避免用户因依赖特定供应商或者协议而难以更换、迁移到新的技术服务上。

OpenTelemetry 定义了一系列的概念，例如 Signal，即一类 Telemetry，包括 Tracing Signal、Metric Signal、Log Signal 等。而在不同组件间传输这些 Telemetry 数据需要遵循的协议就是 OpenTelemetry 协议（OpenTelemetry Protocol，OTLP）。

## Prometheus 和 OpenTelemetry

那么当谈及 Metrics 时，似乎很容易将 OpenTelemetry 与 Prometheus 进行类比：

|                              | Prometheus   | OpenTelemetry  |
|------------------------------|--------------|----------------|
| Data Model                   | Metrics      | Metrics Signal |
| Protocol of Data Propagation | Remote Write | OTLP           |

不知道大家是否有过这样的疑问：**Prometheus 既然已经是云原生指标监控领域的事实标准，大多数供应商、项目也支持它的 Data Model 和 Remote Write Protocol，那为什么要考虑 OpenTelemetry 呢？**

假设你已经在使用 Kubernetes，这个生态里的许多组件都是以 HTTP API 的形式暴露 Prometheus 文本格式的指标数据：
1. [Kubernetes](https://kubernetes.io/docs/reference/instrumentation/metrics/).
2. [Istio](https://istio.io/latest/docs/reference/config/metrics/).
3. [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics).
4. ...

当这些基础设施都不可改变的时候，前面的问题就变成了：**抓取指标数据并将他发送给 Remote Storage，Prometheus 还是 OpenTelemetry 更好？**

不妨来做一下性能测试。

## Benchmark

### 环境搭建

