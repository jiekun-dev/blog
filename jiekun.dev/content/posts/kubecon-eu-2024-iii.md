---
title: "KubeCon 欧洲 2024: 云原生在浪漫之都 | 终章: 探索"
# date: 2024-03-22T17:00:00+01:00
date: 2024-03-30T22:26:00+08:00
author: blog@jiekun.dev
type: post
category: Observability
comments: true
toc: true
draft: true
---

![](../202403-kubecon-eu/kceu24_banner.png)

{{<admonition type=note title="Medium">}}
This blog post is also available in **English**: 
- [KubeCon Europe 2024: Cloud Native In La Ville-Lumière | Exploration
]()
{{< /admonition >}}

## Impression

这是迄今为止规模最大的 KubeCon + CloudNativeCon，超过 **12000** 人来到法国，相聚巴黎凡尔赛门国际展览中心。

![](../202403-kubecon-eu/explore_1.jpg)

我认为这是一次很有趣的欧洲之旅，来到完全不一样的城市，遇到的所有人都很 Nice，线下与 OpenTelemetry、VictoriaMetrics、Thanos 和 Prometheus 的贡献者交流：**“哈哈，原来你们（在一样的问题上）也翻车了”**。

Day 0 的 Co-located Events 非常丰富，主会场更是有很多爆满的场次。我听了许多 Observability Day 的演讲，并且见到了很多熟悉的面孔，Reese Lee 和 Adriana Villela 再次搭档贡献精彩的内容，Juraci Paixão Kröhling 继续分享着他的分布式追踪采样经验...

当然最让人激动的是 OpenTelemetry 已经在**毕业**流程中。在过去的几年，OpenTelemetry 完成了对不同 Signals（Logs，Metrics 和 Tracing）的支持，并且也积累了足够多的用户，它正在逐步演进成可观测性三大支柱的标准。另外，OpenTelemetry 对 **Profiling** 的支持也随着 OTEP [#239](https://github.com/open-telemetry/oteps/pull/239) 的合入成为 2024 年的重要工作内容之一。

今年是 Kubernetes 诞生的第 10 个年头，在 Day 1，盛大的 Keynote 会场坐满了人，AI 是这次 KubeCon 的重要主题，多个 Keynote 都围绕 AI 的发展展开了讨论。

![](../202403-kubecon-eu/keynote_1.jpg)

另外一个让我印象深刻的地方是 Level 7.2 的 [Exhibition Hall](https://events.linuxfoundation.org/wp-content/uploads/2024/03/KubeCon_SponsorShowcase_Map_030124_nobleed-2.pdf)，这里有 200+ 的赞助商 + CNCF Project 展位，大概也是 KubeCon 最热闹的地方：在 Red Hat 领 Red Hat，在 O'Reilly 找图书作者签名合影顺带催翻译进度，甚至还有项目现场发版的。

OpenTelemetry Observatory 展厅聚集了大量的 OpenTelemetry 用户和维护者，甚至有 Sig Meeting 直接在这里进行。我看到了许多的用户反馈和采访环节，并且也参与了其中的一小部分。再次感谢赞助商 Splunk。

![](../202403-kubecon-eu/observatory.jpg)

因为手上有一部分 Metrics 体系的工作，我也和 VictoriaMetrics 的团队进行了简短的交流。VictoriaMetrics 是我们准备替换 Prometheus 和 Thanos 的 Metrics 组件，它很快、很节约资源，并且还有很 Nice 的社区。问及对于可观测性的看法，VictoriaMetrics 的 Co-Founder Roman Khavronenko 表示，**可观测性不仅仅是监控，它还是很多信息的来源，用数据为开发者作出决策提供支持**。

说了这么多，事不宜迟，让我们走进 KubeCon 的分享中，看看大家都在讨论什么。

## Talks


## People
[Henrik Rexed](https://twitter.com/hrexed) 是第一个和我聊天的人，我们在 Slack 上有过短暂交流，他在我的演讲前找到了我。Henrik 有自己的 [YouTube Channel](https://www.youtube.com/c/IsitObservable)，他和我一样带了很多摄影器材过来，并且承担了很多在 OpenTelemetry Observatory 的活动、采访录制。在 KubeCon 碰到 Panasonic 相机的用户并不容易，Henrik 还指导了我很多采访拍摄的事情。

![](../202403-kubecon-eu/henrik.jpg)

[Severin Neumann](https://github.com/svrnm) 是新任的 OpenTelemetry Governance Committee，SIG Communications 成员，致力于降低新贡献者的门槛。他在我提交 CFP 时给了许多建议，我在演讲后见到了他，并且在 OpenTelemetry Observatory 上作了简短采访。

![](../202403-kubecon-eu/severin.jpg)

## About Paris