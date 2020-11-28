---
title: 实现Django Models的数据mock
author: 小黄鸭
type: post
date: 2019-08-03T02:46:19+00:00
excerpt: 开发中定义好业务需要的字段后往往因为数据清洗时间较长，拖慢了测试的间奏。于是尝试实现一套代替获取真实QuerySet的Mock方案，减少开发中对数据源的依赖，提高测试效率，这样真实数据入库后只需要针对少量异常情况调整即可完成测试。
featured_image: /wp-content/uploads/2019/09/Mock.jpeg
rank_math_primary_category:
  - 29
rank_math_seo_score:
  - 10
rank_math_internal_links_processed:
  - 1
categories:
  - 最佳实践
tags:
  - Django
  - Mock
  - Python
  - QuerySet

---
# 问题

在开发过程中，整个数据流向为：

```
爬虫抓取数据->数据中端进行数据清洗->入库Web端定义的业务表

```
由于整个流程比较长，而且由于爬虫开发的不稳定性以及数据统计的复杂度，完整的开发往往不能完全异步进行，因为最后面向业务的Web端需要等待清洗入库的数据进行测试。

一般来说，如果Web端需要的业务数据比较简单，开发自测的时候都可以手动生成INSERT等SQL模拟假数据，但是如果业务复杂的时候，往往需要十余个Table联动的数据，手动INSERT比较麻烦，开发效率低。

# 思考

模拟数据的难处主要有：

  * 涉及地方多，如10多个表逐一写入对应数据
  * 表与表之间的对应关系，如测试的时候需要从表1取10条数据，从表2取这10条数据对应的一周内各日的数据一共10 * 7条
  * 编写测试SQL费事效率低，缺少开箱即用的数据生成器

为此需要有一款工具：

  * 根据Django的models字段随机生成数据
  * 支持指定数据内容，如指定数据的id，方便联表查询的时候能够正确JOIN出结果
  * 支持filter、get等常用方法，支持聚合查询
  * 无需写入数据库，返回QuerySet
  * 方便开关，Mock与测试真正数据之间任意切换

# 实现

将以上问题逐一分析：

## 随机生成对应类型数据

Django的Models常用的数据类型有：  
`CharField`、`IntegerField`、`DateTimeField`、`TextField`、`DecimalField`、`DateField`  
其余类型在开发中不常用，因此先实现这几种类型的随机生成器

#### CharField

CharField对应Varchar和Char类型，目标是在有提供选项的时候随机返回选项中的内容，没提供选项的时候随机出0-max_length范围内的字符串，因此采用英文字母进行随机即可。

```
import string
from random import choice, randint

def charfield_generator(min_length=0, max_length=20, choices=[]):
    if not choices:
        return ''.join(choice(string.ascii_letters) for i in range(randint(min_length, max_length)))
    else:
        return choice(choices)

```
#### IntegerField

IntegerField可以对应各类整数类型，包括SmallInt、TinyInt等均可共用同一个生成器通过限制长度来控制返回值。

```
f integerfield_generator(min_value, max_value, choices=[]):
    if not choices:
        return randint(min_value, max_value)
    else:
        return choice(choices)

```
#### TextField

TextField在要求不严格的情况下也可以和CharField共用生成器。业务上一般超长的内容会使用TextField，如文章正文。

#### DatetimeField

DatetimeField生成对应的时间对象，考虑生成一个大于起始时间(start\_dt)小于结束时间(end\_dt)的datetime对象。

```
f datetime_generator(start_dt=datetime.now(), end_dt=datetime.now()):
    dt_delta = end_dt - start_dt
    return start_dt + timedelta(seconds=randint(0, 24 * 3600 * dt_delta.days + dt_delta.seconds))

```
简单执行一下看看第一版效果：

```
for i in range(10):
    print('-------------- GENERATING SET %s -------------- ' % i)
    print(charfield_generator())
    print(integerfield_generator(0, 1000))
    print(datetime_generator(datetime.strptime('2018-09-01 00:00:00', '%Y-%m-%d %H:%M:%S')))
    print('-------------- GENERATED SET %s -------------- ' % i, '\n')

```
```
GENERATING SET 0 -------------- 
I
196
2018-09-30 13:22:31
-------------- GENERATED SET 0 --------------  

-------------- GENERATING SET 1 -------------- 
XBFGGdpVwmlMMbCT
168
2018-11-16 09:02:16
-------------- GENERATED SET 1 --------------  

-------------- GENERATING SET 2 -------------- 
ZgU
293
2018-12-04 08:44:08
-------------- GENERATED SET 2 --------------  

-------------- GENERATING SET 3 -------------- 
TsUkylUiC
791
2018-10-01 03:48:16
-------------- GENERATED SET 3 --------------  

-------------- GENERATING SET 4 -------------- 
IusHQZsKYFtKi
909
2019-04-22 02:02:27
-------------- GENERATED SET 4 --------------  

-------------- GENERATING SET 5 -------------- 
ScRcj
505
2019-02-21 16:16:51
-------------- GENERATED SET 5 --------------  

-------------- GENERATING SET 6 -------------- 
OLmbMrZImnvaF
500
2018-12-24 22:20:47
-------------- GENERATED SET 6 --------------  

-------------- GENERATING SET 7 -------------- 
rNaRvAYSgxVzwLAe
664
2019-08-01 12:43:00
-------------- GENERATED SET 7 --------------  

-------------- GENERATING SET 8 -------------- 
rtLks
532
2019-03-14 07:38:53
-------------- GENERATED SET 8 --------------  

-------------- GENERATING SET 9 -------------- 
oIDFdOUKs
700
2018-09-21 19:59:06
-------------- GENERATED SET 9 --------------  

[Finished in 0.2s]

```
有了模拟数据之后，需要做几件事：

  * 将假数据映射到对应的Models上，实例化成QuerySet，由多个QuerySet组成iterative的QuerySet List
  * 需要支持指定QuerySet List长度，因此可以实现计算页数、分页等操作

```
f model_generator(models, length):
    """
    : models Models.model :
    : length int :
    : rtype QuerySet List :
    """
    return []

```
# 最佳实践

Work in progress!