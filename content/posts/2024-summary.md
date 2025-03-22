---
title: "2024 Annual Review: My New Role At VictoriaMetrics"
date: 2025-03-21T21:48:21+08:00
author: blog@jiekun.dev
type: post
category: 
 - Career
comments: true
toc: true
---

> This blog post is written before my trip to [KubeCon London](https://kccnceu2025.sched.com/). Exactly one year ago, I met with the VictoriaMetrics team in Paris for the first time. So, this review is about what I have been doing during this past year and how these things have changed my life.

## Contributing Without Pull Requests: Learning by Supporting
At first, I didn't know much about VictoriaMetrics. We were using Prometheus and Thanos, well-recognized metrics monitoring solutions. Like everyone else, we had performance and other issues, so we started looking for alternatives.

At that time, there were already some companies using VictoriaMetrics at a large scale in China, with some case study blog posts. We saw them and started a PoC(proof of concept). It took months, and I, as the one in charge of the PoC, asked some questions in Slack and GitHub. Everything up to now has nothing special, every developer would do the same.

I noticed that many questions were duplicated and already explained in the document. Therefore, in order to reduce the pressure on the VictoriaMetrics engineers (so they would have time to help with my questions), I started to answer questions based on my limited knowledge or the information available in the document.

![](../202503-2024-summary/comment_trend.webp)

By doing so, I looked through many documents and sometimes read the source code to find the answer. It's very helpful, both for the community and me:
1. **Open-source community always lack first-line supporter**, not matter how good the document is. Most people are asking questions, while only a few are answering. The vast majority of these few are core developers because they understand everything, and a very small number are some interested users who try to help each other.
2. **Questions force me to read and learn**. That's a good first step to get familiar with a open-source project. Sometimes it could be the only way, because you may not have a chance to work with it in your day job (e.g., a Go developer maintaining a payment service trying to learn about Kubernetes).

I am very grateful that these supporting things are receiving attention and sometimes I can see comments like "thank you, this solved my issue". At least it proves that (sometimes) my comments are helpful.

Everything happens for a reason. The documentation and source code of VictoriaMetrics are well-organized. At least they didn't give me too much trouble while reading them.

I'm not sure if this "learn by answering" approach would work in other projects. For example, take a look at the issue list of Kubernetes: If the answer is available in the documentation, that would be ideal. If you need to delve into the code, I believe that would be much more challenging.

In summary, if you want to learn and contribute to open-source projects, choosing a good project and starting by addressing real user issues is a good approach, especially for developers who may not have chance to use it in their day job.

## My New Role At VictoriaMetrics

I really like this scene in the movie "[Catch Me If You Can](https://www.imdb.com/title/tt0264464/)", but I want to make a slight modification to it.

![](../202503-2024-summary/interview.webp)

I actually failed the technical interview. But, fortunately, I am not bothered by that and continue my "unofficial" support within the VictoriaMetrics community. Because it helps me gain knowledge and sometimes a simple word of thanks.

Even more fortunate, the team was willing to accept someone who didn't pass the interview, giving me the opportunity to turn supporting into an day job.

## How My Life Was Changed
It's a remote job, so every day I find myself sitting at home in front of the laptop. Meetings, tasks, everything is conducted virtually online.

Without a commute, I can take time to play sports in the mornings or after work. On the flip side, without a company canteen, I now have to cook for myself or order delivery.

Working remotely is also a test of self-discipline. Sometimes I feel exhausted and want to rest. I have the following options:
1. Go to sleep and continue working when I feel better (e.g., in the evening).
2. Take a leave and rest.
3. Work inefficiently.

The first option is best for me because most of my colleagues work in the European time zone. If I rest during the day and work at night, I will have a chance to communicate with them and seek help rather than working alone. 

While the other options more or less affect your work output. In fact, our team doesn’t have something like "KPI". However, from the users' (whether from the community or enterprises) perspective, they might be waiting for your response. In the worst case, users who have a poor experience with technical support might abandon the product, which would make me feel even more mentally exhausted than physically.

In short, I took some time to adapt to the flexible work hours, thinking about how to help the project without clear objectives (I'm not doing it well, tho). This is very different from the work patterns I've experienced. I believe this requires engineers who are truly passionate about the project, rather than those solely seeking wealth through computer science.

## The Language
I'm not very passionate about exploring AI. What I most enjoy doing with AI is:
1. Translation.
2. Practicing spoken English by having conversations with it.

We have a daily sync where everyone talks about what they've done yesterday and discuss some issues. This is how I started.

![](../202503-2024-summary/daily_sync.webp)

It's interesting because you only flow smoothly when reading this pre-prepared content. If there are any questions or discussions, it becomes choppy. It's somewhat better now, but to make it easier for my colleagues to understand, I will still prepare this cheat sheet for every daily sync.

## New Role At The Conference

I have participated in KubeCon as a speaker twice. So this year, I wanted to try something different. I was selected to be on the program committee for KubeCon Europe 2025 and Observability Day Europe 2025. However, I seem to have used up all my luck on that, as I have received 6 proposal rejections for KubeCon and Co-Located Events so far this year.

![](../202503-2024-summary/pc_1.webp)

Before the official transparency report comes out, I have some numbers about the track/event I've reviewed.

For the observability track of KubeCon Europe 2025, there are **185 submissions** in total. 17 of them have been accepted, resulting in a **9.1% acceptance rate**.

For Observability Day Europe 2025, we have received **284 submissions**. 22 of them (7.7%) have been accepted, along with 5 sponsor keynotes, project updates, and opening and closing remarks sessions.

Reviewing proposals is fun only if you don't have to rate and comment. It took me a total of **11 hours** (and earned me a free all-access pass to the conference).

It is quite evident from the review process that:
1. Many speakers have not read the submission guidelines carefully. For example, they did not include a video recording of their previous speech. 
2. End-user submissions focusing on real-world large-scale practices are more competitive than a simple introduction to tools or projects. 
3. Attendees want to learn about the mistakes you've made on a large scale and how they were rectified. Providing detailed numbers would be even better.

## Never Make the Same Mistake Twice

Now let's discuss some aspects that leave me dissatisfied with myself. As mentioned earlier, I am relatively new to VictoriaMetrics. My colleagues are kind and have been very helpful. There are times when I feel confident that I have grasped a concept fully, but the next day when I come across it again, I struggle to recall it (the root cause).

It happened during one of our root cause analysis meetings with a customer. I was nervously checking all available monitoring dashboards and striving to provide a comprehensive analysis. But, bviously, I failed. 

During the post-mortem, my colleague elaborated on the reasons behind everything in detail. I felt very unconfortable because this was almost identical to what we had analyzed in another case not long ago.

{{< admonition type=tip title="Note" >}}
I think the root cause analysis will be presented on the blog in a different format, because we should not mention any of the customer information. However, the issue itself and how my colleague analyzed it are valuable for learning.
{{< /admonition >}}

It's not irrelevant whether the customer blames us or not. Regardless of their satisfaction with this root cause analysis, making the same mistake twice is highly unacceptable for me.

## Next Steps

These are the only things I remember regarding last year. Currently, I have many interesting things on my plate (beyond work), such as evaluating VictoriaLogs as a distributed tracing storage, shooting video interviews for my employer, and in the next few weeks, I'll have a new piano that I should make good use of...

One notable difference from previous annual reviews is that I decided not to set any goals at this moment. Although it may seem that I have more free time, the list of things I wish to accomplish has also grown.

So, let's just go with the flow for now.
