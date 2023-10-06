---
title: "KubeCon 2023 上海：参会“小”记"
date: 2023-10-05T16:32:23+08:00
author: jiekun.dev@gmail.com
type: post
category: Observability
comments: true
---

![](../202309-kubecon/banner.png)

## 0. 前言
国庆前到上海参加了 2 天的 KubeCon + CloudNativeCon + Open Source Summit China 2023，并解锁了新的角色（Speaker，喇叭）。至此，**2022 年**（没错，是 21 个月前定的）OKR 完成度达到了 50%。国庆长假之后良心发现，决定记录一下这次 Journey，但不限于短短的 2 天内的见闻。

从 2019 年关注起，到 2023 年第一次参加，期间 KubeCon 一共举办了 11 届，分别在中国（上海 2 届，线上 1 届），北美（圣迭戈、洛杉矶、底特律各 1 届，线上 1 届），欧洲（
巴伦西亚、阿姆斯特丹各 1 届，线上 2 届）。在每次活动结束后，CNCF 都会发布 [Transparency Report](https://www.cncf.io/reports/kubecon-cloudnativecon-europe-2023/) 来回顾活动以及公布一些数据。其中，演讲者相关的数据分享让我非常感兴趣:
- [Europe 2023](https://www.cncf.io/reports/kubecon-cloudnativecon-europe-2023/)：1761 投稿 / 186 接收 / 接收率 11%；
- [NA 2022](https://www.cncf.io/reports/kubecon-cloudnativecon-north-america-2022-transparency-report/)：1551 投稿 / 176 接收 / 接收率 11%；
- [Europe 2022](https://www.cncf.io/reports/kubecon-cloudnativecon-europe-2022/)：1187 投稿 / 146 接收 / 接收率 12%；
- [China 2021](https://www.cncf.io/wp-content/uploads/2022/02/KubeCon_China_21_Virtual_TransparencyReport_ENG.pdf)：355 投稿 / 67 接收 / 接收率 19%；
- [NA 2021](https://www.cncf.io/wp-content/uploads/2021/11/KubeCon_NA_21_Report.pdf)：976 投稿 / 136 接收 / 接收率 14%；
- [Europe 2021](https://www.cncf.io/wp-content/uploads/2021/06/KubeCon_EU_21_Virtual_TransparencyReport_FINAL.pdf)：624 投稿 / 93 接收 / 接收率 15%；
- [NA 2020](https://www.cncf.io/wp-content/uploads/2020/12/KubeCon_NA_20_Virtual_Report.pdf)：856 投稿 / 128 接收 / 接收率 15%；
- [Europe 2020](https://events.linuxfoundation.org/wp-content/uploads/2020/09/KubeCon_EU_20_Report_final.pdf)：1525 投稿 / 244 接收 / 接收率 16%；
- [NA 2019](https://www.cncf.io/wp-content/uploads/2020/08/KubeCon_NA_19_Report.pdf)：1801 投稿 / 225 接收 / 接收率 12%；
- [China 2019](https://dokumen.tips/documents/kubecon-cloudnativecon-open-source-summit-china-2019-2019-12-21-conference.html)：937 投稿 / 未公布接收数或接收率。

> 投稿与接收数据出现在每次 Transparency Report 的形式和丰富程度不同，部分 Report 只公布了投稿数和接收率或只公布了 Session 数，因此相关数据经过转换计算。准确数据应以 Report 内容或官方统计为准。

![](../202309-kubecon/stats_of_kubecon.png)

通过数据可以看到，每届活动的投稿接收率维持在 10-20% 之间，其中，欧美的接收率又相较中国区略低一些。而投稿数量上看，虽然受过去几年疫情的影响，但欧美的投稿数量仍然远高于中国区。

个人认为，投稿数量一定程度受这些因素影响：疫情政策限制、活动宣传覆盖率、个人意愿。而具体到个人意愿，如果能事先了解一次活动的主题、内容、完整流程，**有可参考对象**，那参与意愿也会随之提升。

因此，写这篇“小”记的另一个动力是：**分享一些投稿和准备的过程**，希望帮助更多**年轻的开发者**参与类似的活动，特别是贡献更多的分享议题。这里强调年轻的开发者是因为：
1. 通常高级工程师们都已经有足够多的经验和经历支撑他们完成一次好的分享；
2. 年轻的开发者需要的更多是**自信**和**尝试**，“小”记的目标正是**鼓励**分享，而不是**指手画脚**地**教**别人完成一次难点攻关去分享。

## 1. 我的 Speaker 之旅
### 1.1 CFP 阶段
想参与一次活动首先得知道活动的时间，以 KubeCon 为例，你可以在很多地方找到它接下来的举办时间和地点：[The Linux Foundation](https://events.linuxfoundation.org/?_sf_s=KubeCon)。在有明确的投稿目标后，关注对应的 CFP（Call For Proposals）时间。例如，我参与的 2023 年中国区 KubeCon 在 9 月 26 - 28 日举行，它的 CFP 时间为 4 月 26 日 - 6 月 18 日。

> 投稿提醒：目前正是 [2024 年欧洲 KubeCon](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/) 的投稿时间，截止时间为 11 月 26 日。

我的分享主题与分布式追踪的采样策略有关，在撰写投稿内容之前，我发邮件咨询了 OpenTelemetry 社区一位有丰富经验的开发者，因为我并不确定我想分享的内容是否有价值，是否已经印在了各种八股文手册中。


{{< details "Mail: Looking for suggestion of KubeCon speech" >}}
> Hi.
>
> 我在很多地方看过你和其他人的采样策略分享，例如 link1 / link2 / ...，所以有一些问题期望得到你的看法和建议：
> 1. 值得再花 30 分钟介绍采样策略吗？
> 2. 是否应该花时间介绍现有的采样策略，还是说应该只关注新的采样方案，例如回溯采样？
>
> ...
{{< /details >}}

这位开发者的回复认为介绍这个主题是可以接受的，因为距离上一次分享已经过了比较长时间，也有很多新的内容出现，并且邮件抄送了另一位社区的分享者，她有过类似主题的分享，因此我可以得到更多的 CFP 建议。过了几天，我在一家 KFC 里一边吃早餐一边完成了 CFP 的标题和内容，并再次回复邮件请教，并在大约 1 周后得到了新的答复。

{{< details "Mail: CFP Review" >}}
> 分布式跟踪在当今的微服务架构中扮演着重要的角色。然而，收集完整的 Trace 数据的成本很高。OpenTelemetry 提供了各种采样策略来减少资源使用，不同的策略需要不同的网络 I/O、内存和存储空间来实现不同的采样效果，这些都分析采样策略在特定场景表现的的关键维度。我们将分享定量分析不同采样策略的成本和质量的经验，并介绍一些我们在探索的新型采样策略，并将他们与现有策略进行比较。
> - Reviewer 1：可以在介绍方案前先描述清楚为什么收集完整 Trace 数据的成本很高；
> - Reviewer 1：可以简单说一下不同采样策略是怎样组合的，因为它们各有不同的要求，同时这也可以为你场景和解决方案作铺垫；
> - Reviewer 2：这是 Collector 侧的采样策略还是也覆盖到 SDK 侧的，可以在描述里介绍清楚；
> - Reviewer 2：在结尾处介绍听众可以有怎样的收获；
> - ...
{{< /details >}}

CFP 修改是个漫长的过程，邮件沟通的效率也不高，最终我在 5 月 19 日确定了[投递内容](https://sched.co/1PTFF)。在正式投递的过程中，还有许多材料需要准备，你需要介绍：
- 你的分享会对项目生态和社区带来什么好处；
- 你在分享相关项目上的经验；
- 你的分享经历。

作为首次参与的分享者，我提交了 2 段分享视频，分别是 2022 年参加 TiDB Hackathon 的 Lightning Talk 以及临时录制的与投递内容相关的分享。因为是单盲审稿，这些材料有助于让审稿人了解投递者对内容的掌握程度以及讲述能力。

![](../202309-kubecon/additional_resources.png)

到这里，CFP 投递就告一段落了，接下来只需要等待 CFP 结果。2023 年中国区 KubeCon 的 CFP 结果在 8 月 3 日揭晓，中选的议题将有接近 2 个月时间进行准备（当然，提前进行准备总是更好的）。

这里还有一些其他的信息在我 CFP 投递过程中帮到我，希望也能对在阅读的你有帮助：
- [Crafting Compelling Conference Proposals](https://jpkroehling.notion.site/Crafting-Compelling-Conference-Proposals-3e96ad0660584a329354397912691153)；
- [Submission Reviewer Guidelines](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/program/submission-reviewer-guidelines/)；
- [评分准则和最佳实践](https://www.lfasiallc.com/archive/2021/kubecon-cloudnativecon-open-source-summit-china/program/scoring-guidelines/)；
- [Call For Proposals](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/program/cfp/)。

### 1.2 分享准备
分享内容的准备工作因主题分享而异，但是通用的建议是及早收集材料，以免在收到 “议题被接收” 的邮件时因一页 PPT 都写不出来而良心不安。

在参加 KubeCon 之前，我过往的分享范围都是工作所在的小组、部门，所以我需要更多的工具、步骤来帮我完成这次对外的演讲。这些工具、步骤包括：
1. 为每页 PPT 准备完整的文字版本，详细程度基准是：任意路人都可以照念完成分享；
2. 完成 1-2 次的分享演练，录音并聆听自己的分享；
3. 完成 1 次组内分享，听取大家的建议作修改；
4. 录制视频，邀请 3 位相关领域的专家观看和提出修改建议；
5. 在演讲前完成最后的演练，录音并聆听，看看效果是否满意。

在处理第 4 点的时候，有一些比较有趣的事情。因为向一位国外的开发者寻求建议，我需要准备英文的演讲视频。受限于英语水平，我让 ChatGPT 帮我完成了大部分的翻译工作，再一一进行核对。事实证明 ChatGPT 真的很喜欢添油加醋，还会经常在翻译的过程中回答我的一些疑问句。

![](../202309-kubecon/playing_with_chatgpt.png)

在艰难完成翻译后，录制英文演讲视频时我又遇到了新的麻烦。因为对发音、语调总是不满意，在多次尝试后，我不得不寻找 AI 代讲，最终的 Dry Run 视频可以在这里找到：[dryun](https://www.youtube.com/watch?v=TIMDgOVlWP4)。

{{< youtube TIMDgOVlWP4 >}}

回到准备过程本身，最重要的事情是让演练的听众、专家提出建议并调整演讲内容。通常，领域的专家不一定是非常专业的演讲者，或者说未必有很丰富的演讲经验，如果他们在表达专业意见上有阻碍，可以参考 CNCF 提供的[演讲者指南](https://drive.google.com/file/d/1dcXenOObdl2d7h1RBUqHnauTIuA2B2oq/view?pli=1)，**主动询问**以下方面他们的想法：
1. 我的演讲表达足够清晰吗，你是否清晰了解我想分享的主题和内容？
2. 我的 PPT 设计合理吗？
   - 哪些页面文字太多或太少？
   - 哪些页面可以做更多的图表表达，哪些图表不应该使用？
   - 是否有合适的幽默内容片段？
   - 演讲的速度是否容易 Follow？
3. ...

同样，我在这里引用一些 Reviewer 的修改意见作为示例。

{{< details "Conversation: Presentation Review" >}}
> - Section 2：头部采样和尾部采样的优缺点过于简略；
> - Section 3：性能评估的时候没有介绍 Collector 的部署架构和细节，包括多个影响性能测试的参数没有告诉听众；
> - Section 3：性能评估结果的图线太小，并且相关数据也没有对比参考，只展示数据不能说明这个数据是好是坏；
> - Section 4：有一种采样策略的使用场景讲解得不够清晰，应用场景不太明确；
> - ...
{{< /details >}}

多次修改后形成最终的分享，在参加活动前，可以将 PPT 导出上传到议题的页面，可以吸引一些还没决定参与哪场分享的听众。参会者阅读 PPT 可以更有针对性地选择分享会场，并且提前熟悉分享内容，便于跟上演讲者的节奏。

### 1.3 演讲
我的议题整理在之前的博客中，如果感兴趣的话可以前往 [KubeCon 2023 上海：OpenTelemetry 采样分享](https://jiekun.dev/otel) 阅读。演讲整理确实是个非常辛苦的事情，因为书面用语和口头用语有非常大的差别，建议有钞能力的同学外包给 ChatGPT。

演讲之前，工作人员会特地叮嘱要重复听众的提问，以确保问题能清晰地记录在视频中，活动的视频会在 10 月 13 日之前上传至 Youtube。

在演讲的时候，主办方通常会有摄影师在场，这些视频会在活动后 1 周内上传，但是因为场地很多，所以并不保证每位演讲者都有能回去交差的照片。让随行同事帮忙拍一些保底会更稳妥，或者像我一样与家人一同前往上海，并且找主办方索要短暂入场的允许。

### 1.4 小结
以上是对我作为演讲者参与 KubeCon 的一些记录，希望能鼓动感兴趣的年轻开发者参与到未来的活动中。在投稿时，有一些建议我认为对一线开发者是非常有价值的：
1. 分享投稿不必介意以前是否有人分享过，不必追求 100% 新颖的想法、内容，更重要的是你希望听众从你这里收获什么。而这些事情会有审稿人替你操心，如果他们觉得不合适，自然会反馈回来；
2. 分享投稿不会要求你的背景，比如职位 Title，是否是 Tech Leader 或者 CTO。同样，审稿人也会根据你的材料 Review 你是否适合完成这次分享。

希望在下一次 KubeCon 上认识正在阅读本文的你！

## 2. 可观测性 Session 小记
// todo 2023-10-30

## 3. 其他 Session 小记
// todo 2023-10-30

## 4. 更多图片
查看活动照片：[KubeCon + CloudNativeCon + Open Source Summit China 2023](https://www.flickr.com/photos/143247548@N03/albums/72177720311405294)

![](https://live.staticflickr.com/65535/53217714240_f8ba6c8ba8_o.jpg)

![](https://live.staticflickr.com/65535/53220136444_e4921573f8_o.jpg)

![](https://live.staticflickr.com/65535/53235476845_06a90681c7_o.jpg)

![](https://live.staticflickr.com/65535/53236882593_21d661b30c_o.jpg)

![](https://live.staticflickr.com/65535/53236882838_e90578564c_o.jpg)

![](https://live.staticflickr.com/65535/53236951309_c4e943b31f_o.jpg)

![](https://live.staticflickr.com/65535/53236950864_f8fd89b392_o.jpg)

![](https://live.staticflickr.com/65535/53236950874_4539883c7e_o.jpg)

![](https://live.staticflickr.com/65535/53235001851_2de69ce3c4_o.jpg)

![](https://live.staticflickr.com/65535/53234138657_60fa0a956d_o.jpg)
