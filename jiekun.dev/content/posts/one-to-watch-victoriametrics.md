---
title: "[翻译] 聚焦 VictoriaMetrics：一家乌克兰初创企业的故事"
date: 2024-06-23T15:32:11+08:00
author: blog@jiekun.dev
type: post
category: 
 - Observability
comments: true
toc: true
image: https://jiekun.dev/posts/202406-one-to-watch-victoriametrics/cover.jpg
---

![](../202406-one-to-watch-victoriametrics/cover.jpg)

{{<admonition type=note title="Medium">}}
This blog post is originally posted by **The Stack** on: 
- [One to Watch: VictoriaMetrics - The story of a startup from Ukraine](https://www.thestack.technology/one-to-watch-victoriametrics-the-story-of-a-startup-from-ukraine/)
{{< /admonition >}}

这是一个初创公司追寻成功的故事，虽然不管从哪个方面看，这个故事都耳熟能详：
- [x] 一群聪明人有一个填补市场空白的创意；
- [x] 他们成立了一家自力更生（Bootstrapped）的公司；
- [x] 他们花时间编码实现这个创意；
- [x] 他们对外宣传并展示 POC（Proof Of Concept）；
- [x] 最终出现了愿意买单的用户；
- [x] 然后他们继续努力将其发展成为一个更大的项目。

但是 [VictoriaMetrics](https://victoriametrics.com/) 有一点不同：它是一家乌克兰公司，尽管现在总部设在旧金山，它曾受到过战争的严重打击。

让我们先来聊下公司本身。它的客户包括阿迪达斯（Adidas）、Grammarly 和欧洲粒子物理实验室（CERN）等，并被 [The Stack](https://thestack.technology) 收录在最新一期“值得关注公司”名单中。VictoriaMetrics 提供时序数据库和监控软件服务，它们的口号是 “We make monitoring simple & reliable for everyone”。

## VictoriaMetrics 具体是做什么的？
这家公司聚焦的场景是：业务扩张与其带来的海量事件（Events），两者之间的脱节问题。例如，VictoriaMetrics 专注于指标（Metrics），指标代表的是系统的表现（译注：表现包括且不限于性能、业务量等），掌握这些指标，就能够构建监控面板并预测走势。但是指标数量随着业务规模增长，对计算和存储提出了实际的需求。

![](../202406-one-to-watch-victoriametrics/roman.jpg)

“当业务增长了10%，需要追踪的指标就可能会增长 100 倍甚至 1000 倍，这时企业就会开始寻找（现有指标监控的）替代方案。” Co-Founder Roman Khavronenko 说道。

“他们从简单的解决方案开始，碰到扩展上的瓶颈，但要想办法满足未来的增长。VictoriaMetrics 就是专门为处理这种海量数据的场景而诞生的。”

该公司采用了一套很标准的商业产品开源模式：“99% 的功能对所有人免费开源，1% 的功能针对企业需求、安全性和租户管理闭源，仅向企业用户提供。企业服务的另一个重要部分是提供支持，帮助和指导客户管理指标。大公司希望保证一切正常运行，而监控是非常复杂的，他们需要专家来回答系统设计等方面的问题。”

VictoriaMetrics 拒绝就营收问题作出评论，但表示已经有大约 1.2 亿次下载，并且有像网站构建工具 Wix、游戏平台 Roblox 和写作插件 [Grammarly](https://www.grammarly.com/) 这些公司的使用案例。Grammarly 表示，与之前的监控解决方案相比，它降低了 10 倍的成本。其他用户还包括 CERN（用于大型强子对撞机的 CMS 通用探测器系统）和阿迪达斯（Adidas）等公司。

## VictoriaMetrics: 与众不同的发展路线
该公司有机地进行招聘，仅依靠 10 至 20 名员工运作，并广泛使用 Contractors。与硅谷一些热门的初创公司不同，它目前没有计划寻求大规模的融资。

“我们对来自企业业务的收入感到满意，” Khavronenko 说道。“这足以支持我们的团队扩张、发展和开发新的解决方案。”

那为什么不像大多数初创公司那样迫切向前发展呢？

“（如果融资）你会向投资者承诺份额，这是 OK 的，但这意味着你正在失去公司的一部分。 设想一下，你已经拥有了足够的资金，那么为什么还需要额外的融资呢？我并不是说融资是件坏事，这只是一个愿不愿意的权衡。”

Khavronenko 强调并不是说绝对不会接受第三方资金，尽管目前还没有从外部筹集到任何资金。他的言辞中透露出，如果存在一个与众不同的提议，他可能会考虑接受第三方资金。

“我坚持认为，筹集资金可以带来快速发展，并在市场上获得时间上的优势，就如人们所说的先行者优势。”

他说道：“你可以更快地前进，但速度并不等同于质量。”

“在许多情况下，这会带来压力并导致错误。”

目前，该公司“不愿意为了投资者的 KPI 而牺牲自身”，但“如果未来有需要大量资金的机遇，这也会发生改变。一切都取决于 Oportunity、Cost 和 Cost of Opportunity”。

并且，只要数据的规模足够大，机遇就无处不在。这个领域非常广阔，因为“如果你拥有巨大的数据规模，就需要收集所有这些数据的指标”。数据可以来自一个应用、你的汽车，甚至是你的冰箱，关键是观察正在发生的事情，然后窥探未来。

因此，有很多机会扩展指标，然后对应调整市场上的操作。

Khavronenko 表示：“日志与指标有所不同，但仍然是可观测性的三大支柱之一。我们正在努力研究这个领域，并对今年有很大的期望。”第三个支柱是追踪，VictoriaMetrics 似乎希望在日志之后构建一个完整的可观察性堆栈，这是一个很自然的发展方向。但他们还有很多其他的事情要考虑，比如在支持 AWS 和本地部署之后，增加对 Google 和 Microsoft 云的支持。

## Code of war

因此，VictoriaMetrics 知道如何发展并扩展其影响力，追随着无数人遵循古老的创新智慧（“build a better mousetrap and the world will beat a path to your door”）和诗歌（“a man’s reach should exceed his grasp/Or what’s a heaven for?”）。

但该公司拥有与大多数其他公司不同的视角，它曾经经历过一段战争。

Khavronenko 以他特有的轻描淡写的方式说：“它确实对我们的工作地点和生活产生了影响。”

他在疫情爆发前半年离开了伦敦 Cloudflare，他曾经在那里担任工程师，然后前往基辅加入了 VictoriaMetrics：“我买了一套公寓，我的儿子就在那里出生，然后战争开始了；那是一段可怕的经历，”他回忆道。“有些人留在了 VictoriaMetrics，有些人离开了。但这并没有影响公司的增长。我和其他同事在工作中找到了拯救。当你无法左右局势时，你只能去左右你能控制的事情...你可以控制自己写什么样的代码。”

确实，在第一次危机期间，代码量出现了激增，比过往任何时候都多。

如今，Khavronenko 表示他很平静，VictoriaMetrics 的员工们也安全。他提到，另一种不同类型的危机，即新冠疫情和随后的 Lockdown，燃起了在物理隔离的环境下合作工作的苗头。目前，VictoriaMetrics 是一家现代化的、成员分散在各地的初创公司，没有固定的工作时间，并通过线上渠道运作，如使用 GitHub 跟进工单等。“我们不遵循传统的 9 点到 5 点的工作时间，如果有人想多睡一会，也完全没问题，这是为了让你找到最适合的工作方式。”

毕竟，还有更重要的事情需要考虑，Khavronenko 表示他对欧盟、对那些向他的同胞伸出援手并帮助他们重新开始生活的国家和公司“非常感激”。

他目前居住在奥地利，另外两位联合创始人分别在美国和波兰。

“每个人想象过的成功的样子，而这种想象又人人不同。”，他思考着说，“我们在 VictoriaMetrics 中打造的实际上是人和团队：喜欢相互合作的人在一起构建一个被成千上万人使用的产品。即使这个产品消失了，那些参与构建它的人仍然会在，他们可以构建更好的产品并解决更重要的问题。对我来说，这就是成功：人们解决真实问题。”