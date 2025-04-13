---
title: "KubeCon 欧洲 2025 参会指南"
date: 2025-04-06T23:05:00+00:00
author: blog@jiekun.dev
type: post
category: 
 - Observability
comments: true
toc: true
draft: true
---

从巴黎到伦敦，这是我的第二次 KubeCon Europe 之旅。

技术会议本该是枯燥的（试想连续 5 天一共参加 20 场分享，大概会想回家种田吧），但是 KubeCon 与众不同之处就是它会让你找到很多乐趣。如果你也想知道这次 KubeCon 为什么是特别的，请接着往下看。

## 1. KubeCon 有什么好玩？

### 1.1 Events

KubeCon 的性质首先是一次线下的技术分享，所以咱就从不同类型的议题说起，这与国内大部分技术会议没有太大区别。如果你对技术分享不感兴趣，或者纯粹只是想来凑个热闹以及旅游，可以直接跳到 1.2 小节。

#### 1.1.1 维护者峰会（Maintainer Summit）

维护者峰会聚集了 CNCF 托管项目的众多维护者，大家来自不同企业，但共同维护着像 Kubernetes、OpenTelemetry、Prometheus 这样的大型项目，所以你可以期待在这里了解到许多开源项目治理的经验。

![](演讲者峰会图)

这次会议我感兴趣的议题有：
- [Automate Maintainer Tasks With GitHub Actions（用 GitHub Actions 来自动化维护者的工作）](https://sched.co/1uSND)
- [Maintaining the Maintainers (如何管理项目维护者们）](https://sched.co/1uSNJ)
- [Avoiding Breaking Changes in Your Project: Lessons From the OpenTelemetry Collector（避免破坏性改动：OpenTelemetry Collector 的前车之鉴）](https://sched.co/1uSNY)

与往年的 Kubernetes Contributor Summit 类似，每位参会者可以获得**一件 Maintainer T恤**，**一枚 Maintainer 徽章**，并且可以在会议当日下午参与大合照。

![](演讲者峰会合影)

{{<admonition type=note title="如何参加 Maintainer Summit">}}

需要同时满足：
- 持有 KubeCon + CloudNativeCon 门票。
- CNCF 任意项目在 GitHub 上的 Organization Member。

{{< /admonition >}}

#### 1.1.2 同场活动（Co-located Events）

Co-located Events 是与云原生相关的 **17** 个分会场（例如 **Observability Day**、**ArgoCon**、**Istio Day** 等），每个分会场按规模为半天或一天，各将数十个分享议题安排在一个或两个 1000 人规模的会议厅举行。

![](Observability Day 图)

Co-located Events 的特点是更加聚焦某个领域或项目。例如我可能只对可观测性感兴趣，Observability Day 一共有 22 个讨论 OpenTelemetry、Logs、Profiling 的议题，可以完美作为主会场可观测性方向主题（通常只有个位数的议题能入选）的补充。

{{<admonition type=note title="如何参加 Co-Located Events">}}

需要持有 KubeCon + CloudNativeCon All-Access 门票，可通过钞能力或演讲议题投稿被 KubeCon 或任意 Co-Located Events 接收获得。

Co-Located Events 的议题接收率视 Event 的不同而不同，热门 Event 如 Observability Day 本次接收率为 7.7%。

{{< /admonition >}}


#### 1.1.3 主会场（KubeCon + CloudNativeCon）

万众期待的 KubeCon 主会场一共有 3 天，当然很少人能全程听下来便是了。但是作为一个**万人参与**的大型活动，每次 Keynote 场面都是很震撼的，特别在巴黎听到 Priyanka Sharma 激动的声音：Twelve thousand plus people right here（超过 12000 人来到现场）！相比之下今年 Chris Aniszczyk 的开场白好像稍微缺点激情，也算是反映了真实的 Engineering 风格。

![](主会场图1)
![](主会场图2)

KubeCon 特色的地方不在于这些演讲，因为除了入选困难、质量相对把控更好以外，没有更多的特色了。技术分享再怎么说花样还是有限的，通常有感兴趣的 3 到 5 场议题能听完有所收获已经是非常理想的了。

{{<admonition type=note title="如何参加 KubeCon + CloudNativeCon">}}

需要持有 KubeCon + CloudNativeCon 门票，可通过钞能力或演讲议题投稿被 KubeCon 或任意 Co-Located Events 接收获得。

KubeCon + CloudNativeCon 的整体接收率低于 10%。其中，以 Observability 方向为例，本次接收率为 9.1%。

{{< /admonition >}}

### 1.2 Sponsor Showcase

真正好玩的内容才刚刚开始。

不知道你在技术会议上见过最多有多少家赞助商？作为云原生领域最大的盛会，本次 KubeCon 一共吸引了 **275** 家企业和组织赞助。赞助商可以获得（不小于）3 * 2.5m 的场地进行展览。

![](sponsor 图)

通常赞助商们都有各式各样的礼品可以赠送，只要你简单和他们交谈几句（或者甚至不用交谈）就可以获得。这次活动我收获的礼品包括：
- T恤（10余件）
- 连帽衫（2件）
- 袜子（3双）
- 签名赠书（1本）
- 各种挂件
- 其他周边（充电线、公仔玩偶、徽章、钥匙扣、贴纸...）

如果你的运气好，现场有许多赞助商有更大的奖品（耳机、PlayStation、Xbox）定时抽奖。当然，NVIDIA 并没有送显卡，Intel 和 AMD 也没有送 CPU，可惜。

### Project Pavilion

CNCF 的（大约 20 个）项目会有自己的展亭（Prometheus、Jaeger、Thanos、Cortex 等等），并且有 Maintainer 轮换值守。如果你不知道在哪个技术分享上能逮到 Maintainer，那直接去 Project Pavilion 就可以了。

![](Project Pavilion图)

这次 KubeCon 我和 Jaeger 及 Prometheus 的 Maintainer 有过交流，主要都是关注一些项目未来功能的规划做得怎么样了，什么时候能到下一个阶段。

同时这也是个很好的 Q&A 机会，如果平时对应项目中有积攒已久的问题，不妨直接面对面聊一下，或许会有意外收获。

{{<admonition type=note title="OpenTelemetry 没有 Project Pavilion？">}}

眼尖的你发现了 OpenTelemetry 似乎没被提及到。没错，因为它拥有独享的超大展厅 OpenTelemetry Observatory，由金主 Splunk 赞助。

![](Observatory 图)

{{< /admonition >}}

### Headshot

上一次拍简历照片是什么时候，应该不会是大学毕业前吧！如果你正好想更新一下简历上的照片，那么扫码预约，KubeCon 提供专业的 Headshot 服务。

专业的摄影师，专业的场地灯光布置，你只需按时前来，摄影师会给你拍摄几套不一样的照片供选择（当然，最终只能选定 1 张）。现场就可以预览选片，成片经过处理会在 3 周内发送至邮箱。

![](Headshot 图)

从实际预约来看，Slot 都很抢手。如果剩余的 Slot 不合适，没关系，先预约上，因为有很多人并没有按时到场，所以只要保证自己名字出现在列表上就可以了，摄影师大部分时间都是空闲的。

## 参加一次 KubeCon 的花销

参加国外 KubeCon 其实类似于一次旅游（实际上它确实是，因为一般我都只有少数时间在会场薅羊毛，其他时间都去景点了），那开销不外乎签证机票酒店和各种门票，以及饮食购物。

TL;DR：人均 20000 元。

### 签证

英国的签证单次需要大约 1000 元，如果通过，有效时间一般是 2 年。除了英国以外，KubeCon Europe 通常会在申根国家举行（如法国、荷兰），它们的签证费用也通常在 1000 元附近。

而如果你打算参加 KubeCon North America，申请美国签证的费用为 1369 元。

### 机票酒店

直飞往返伦敦机票 5660 元，酒店在市区住了 8 天一共 883 英镑，折合 8332 元。

当然，这类活动通常都应该是公司赞助出行，所以机酒开销就可以省去了。

### 日常开销

在伦敦的生活成本还是比国内高很多的，据之前在伦敦工作的同事说，如果你有钱，伦敦是个很好的城市。

在出发前我以为饮食会是最贵的，但是没想到最出乎意料的是交通的成本：
- 地铁刷 Oyster 卡 2.9 英镑起，一天花 10 多英镑坐公交都是家常便饭。
- 公交同上。
- 打车想都不要想，基本上一公里大约 10 英镑的价格，随便跑十公里轻松花掉上千元。

另外饮食方面，英国的早餐挺丰盛，7 英镑的早餐一人两人都可以满足，也有更便宜的选择丰俭由人。正餐似乎花样不太多，所以我们还尝试了各式各样的意大利菜、土耳其烤肉、巴西烤肉、日韩料理，差点连中餐都吃上了，这些价格都在 20-30 英镑/人。

![](早餐图)

如果想要节俭填饱肚子，超市的 Meal Deal 是个不错的选择，3.6 英镑即可拥有主食（三明治）、小吃（虾仁）、饮料（矿泉水）的组合，填饱肚子没有问题。以及 KubeCon 会场本身提供一日三餐，全在会场解决也可以。

最后既然出国旅游，一些当地的景点自然是要去的，通常价格从免费到数十英镑不等。

### 门票

KubeCon 并不是免费的，并且门票对于个人来说相当昂贵（399 美元，含 Co-Located Events 599 美元）。

## 免费的 KubeCon 门票

如果公司能够赞助 KubeCon 门票（999 美元，含 Co-Located Events 1199 美元）那当然是再好不过了，特别如果公司是 Sponsor 的话会收到至少 8 张门票。

如果没有得到公司的赞助，你可以考虑这些途径免费获得门票：
1. CNCF 的奖学金可以为有需要的特定人群（Maintainer、少数群体）赞助门票，申请等待审批即可。
2. 所有的 Speaker 都将得到 All-Access 门票。
3. 所有的 KubeCon Program Committee 都将得到门票，Co-Located Events Program Committee 将减免参与 Co-Located Events 的费用。
4. 偶尔会有公司将多余的 Sponsor 门票赠送给社区用户。

## 写在最后

结束了 8 天的伦敦之旅，在返程飞机上写下这篇文章。感谢公司提供的机会以及支持，毕竟长期远程办公的情况下，能够一次见到这么多同事确实难得。

另外遗憾的就是没有议题入选，所以纯当旅游来看有点对不起报销的机票，算了吧还是下周好好上班吧老板会理解的。

![](英国图1)
![](英国图2)
![](英国图3)