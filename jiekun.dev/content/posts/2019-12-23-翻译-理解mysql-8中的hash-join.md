---
title: '[翻译] 理解MySQL 8中的HASH JOIN'
author: 小黄鸭
type: post
date: 2019-12-23T12:18:11+00:00
excerpt: MySQL 8的Hash Join功能，借助内存哈希临时表大幅度提高非索引列Join效率
featured_image: /wp-content/uploads/2019/12/Screenshot-from-2019-12-23-20-14-33.jpg
rank_math_primary_category:
  - 31
rank_math_seo_score:
  - 16
rank_math_internal_links_processed:
  - 1
categories:
  - 数据库
  - 翻译
tags:
  - Hash
  - MySQL

---
<blockquote class="wp-block-quote">
  <p>
    原文标题：Understanding Hash Joins in MySQL 8<br /> 原文链接：https://www.percona.com/blog/2019/10/30/understanding-hash-joins-in-mysql-8/<br /> 作者：Tibor Korocz<br /> 译者：2014BDuck<br /> 翻译时间：2019-12-23
  </p>
</blockquote>

在MySQL 8.0.18中有个新功能叫Hash Joins。我打算研究一下它是如何运作的和在什么场景下它能够帮到我们。你可以在[这里][1]了解它的底层原理。

更上层的解释：如果使用join查询，它会基于其中一个表在内存构建一个哈希表，然后一行一行读另一个表，计算其哈希值到内存哈希表中进行查找。

## 很好，但性能上带给我们什么好处呢？

首先，它只会在没有索引的字段上生效，所以它是个实时的表扫描。通常我们不推荐在没有索引的列上join查询，因为这很慢。这种情况下Hash Joins就有它的优势，因为它用的是内存哈希表而不是嵌套循环（Nested Loop）。

让我们来做些测试。首先创建如下表：

```
CREATE TABLE `t1` (
`id` int(11) NOT NULL AUTO_INCREMENT ,
`c1` int(11) NOT NULL DEFAULT '0',
`c2` int(11) NOT NULL DEFAULT '0',
PRIMARY KEY (`id`),
KEY `idx_c1` (`c1`)
) ENGINE=InnoDB;

CREATE TABLE `t2` (
`id` int(11) NOT NULL AUTO_INCREMENT ,
`c1` int(11) NOT NULL DEFAULT '0',
`c2` int(11) NOT NULL DEFAULT '0',
PRIMARY KEY (`id`),
KEY `idx_c1` (`c1`)
) ENGINE=InnoDB;</code></pre>

```
我向两个表都插入了131072行随机数据。

```
mysql> select count(*) from t1;
+----------+
| count(*) |
+----------+
| 131072   |
+----------+</code></pre>

```
## 测试1 &#8211; Hash Joins

基于没有索引的表c2执行Join查询。

```
mysql> explain format=tree select count(*) from t1 join t2 on t1.c2 = t2.c2\G
*************************** 1. row ***************************
EXPLAIN: -> Aggregate: count(0)
-> Inner hash join (t2.c2 = t1.c2) (cost=1728502115.04 rows=1728488704)
-> Table scan on t2 (cost=0.01 rows=131472)
-> Hash
-> Table scan on t1 (cost=13219.45 rows=131472)

1 row in set (0.00 sec)</code></pre>

```
我们使用`explain format=tree`来查看Hash Join是否生效，默认情况下(explain)会误导说这会是个嵌套循环。我已经提了[bug report][2]，在工单中你可以看到开发者回复：

<blockquote class="wp-block-quote">
  <p>
    解决方法就是不要用传统的<code>EXPLAIN</code>。
  </p>
</blockquote>

因此在旧的explain中这不会被修复，我们要使用新的查询方式。

回到语句上，我们可以看到它使用Hash Join了，但性能有多快？

```
mysql> select count(*) from t1 join t2 on t1.c2 = t2.c2;
+----------+
| count(*) |
+----------+
| 17172231 |
+----------+
1 row in set (0.73 sec)</code></pre>

```
17000000多行数据，0.73秒。看起来很不错。

## 测试2 &#8211; 非Hash Joins

我们现在用优化器的开关或hint关掉Hash Join。

```
mysql> select /*+ NO_HASH_JOIN (t1,t2) */ count(*) from t1 join t2 on t1.c2 = t2.c2;
+----------+
| count(*) |
+----------+
| 17172231 |
+----------+
1 row in set (13 min 36.36 sec)</code></pre>

```
同样的查询使用了超过13分钟。非常大的差距，可以看到Hash Join性能提升明显。

## 测试3 &#8211; 索引Join

让我们来创建索引，看看基于索引的join速度如何。

```
te index idx_c2 on t1(c2);
create index idx_c2 on t2(c2);

mysql> select count(*) from t1 join t2 on t1.c2 = t2.c2;
+----------+
| count(*) |
+----------+
| 17172231 |
+----------+
1 row in set (2.63 sec)</code></pre>

```
2.6秒。在这个测试用例中Hash Join比基于索引的Join还要快。

然而，我可以在索引可用的情况下，通过`ignore index`强制优化器使用Hash Join：

```
mysql> explain format=tree select count(*) from t1 ignore index (idx_c2) join t2 ignore index (idx_c2) on t1.c2 = t2.c2 where t1.c2=t2.c2\G
*************************** 1. row ***************************
EXPLAIN: -> Aggregate: count(0)
-> Inner hash join (t2.c2 = t1.c2) (cost=1728502115.04 rows=17336898)
-> Table scan on t2 (cost=0.00 rows=131472)
-> Hash
-> Table scan on t1 (cost=13219.45 rows=131472)

1 row in set (0.00 sec)</code></pre>

```
我在想即使索引存在的情况下，优化器也能够根据提示使用Hash Join，这样我们就不需要在各种表上`ignore index`了。我已经提了[feature request][3]。

然而，如果你有认真阅读我提的[bug report][2]，你会看到MySQL的开发者有表明这可能是不必要的：

<blockquote class="wp-block-quote">
  <p>
    块嵌套循环（Block Nested-Loop）在某些情况下完全不会起作用，这时提示（hint）会被忽略。
  </p>
</blockquote>

这意味着他们在未来可能打算移除块钱套循环Join，使用Hash Join代替。

## 限制

我们可以看到Hash Join很强大，但也有些限制：

  * 我提到过它只在没有索引的列上其作用（或者需要手动忽略索引）
  * 只有Join条件是“=”的情况才会生效
  * `LEFT JOIN`和`RIGHT JOIN`不生效

我还期望看到Hash Join使用次数的统计，所以我又提了[另一个feature request][4]

## 总结

Hash Join是个很强大的join查询方式，我们应该重点关注它，未来如果有更多Feature我也不会感到惊讶。理论上，它应该也能用来做`LEFT JOIN`和`RIGHT JOIN`，我们在bug report的评论里面也能看到Oracle在未来也打算使用Hash Join。

 [1]: https://dev.mysql.com/worklog/task/?id=2241
 [2]: https://bugs.mysql.com/bug.php?id=97299
 [3]: https://bugs.mysql.com/bug.php?id=97302
 [4]: https://bugs.mysql.com/bug.php?id=97301