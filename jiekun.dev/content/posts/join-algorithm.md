---
title: 我的Join查询是如何得出结果的？
author: 小黄鸭
type: post
date: 2020-08-08T08:54:53+00:00
excerpt: Join查询的几种基础算法
featured_image: /wp-content/uploads/2020/08/join_algorithm.png
category: Database
comments: true
archieved: false

---

# Join Algorithms
在需要连接多表数据时，我们通常会使用到`JOIN`操作。

# Nested Loop Join
## Basic Nested Loop Join
假设现在有两个关系对集合，`R`和`S`，我们需要将它连接起来，连接通过一定的条件来指定，这个条件我们称为**Join Predicate**，即连接谓词:

```
JP(r, s) := r.x == s.x
```

这个连接谓词表明`R`与`S`依靠字段`x`相等作为条件进行连接，当满足条件时，`JP(r, s)`返回为`True`，否则为`False`。

下面用伪代码表述**Nested Loop Join**的处理过程：
```python
# R
# S
# JP(r, s) := r.x == s.x

def nested_loop_join(R, S, JP(r, s)):
    for r in R:
        for s in S:
            if JP(r, s):
                outupt((r, s))
```

可以看出Nested Loop Join的处理过程即是两个关系对集合`R`与`S`的求**Cross Product**过程，因此若`R`中有n个关系对，`S`中有m个关系对，他们的处理复杂度即为`O(m * n)`。

**Nested Loop Join**的优势在于它不仅可以在等值连接（EquiJoin）中使用，还可以处理其他非等值连接：
- 在NLJ中无需关注判定式`JP(r, s)`的实现
- 对于非等值连接，只判定式内部实现对应**Join Predicate**即可，外层循环无感知。如：`JP(r, s) := r.x <= s.x`

## Indexed Nested Loop Join
现在输入的情况稍作改变，我们不仅有`R`、`S`和连接谓词`JP(r, s)`，还知道在其中一个关系集合，无论是`R`还是`S`，的连接谓词的列`x`上有对应的索引：
```
IndexOnRX := catelog.get(indexes, R.x)
```
`IndexOnRX`现在可以视为一个`R`上`x`列的索引查询方法，它可以接收查询值，并且返回一个集合，代表集合中的元素存在于`R`中。

下面用伪代码表述**Indexed Nested Loop Join**的处理过程：
```python
# S
# JP(r, s)
# IndexOnRX

def indexed_nested_loop_join(IndexOnRX, S, JP(r, s)):
    for s in S:
        query_result_set = IndexOnRX(s.x)
        if query_result_set != None:
            output({s} * query_result_set)
```

与NLJ不同，INLJ通过遍历其中1个关系对集合S，使用对应的索引查询得到另一个查询集合**R的子集**，如果有符合的case，将`{s} * query_result_set`结果追加到最后输出的集合中。因此在INLJ中，因为只遍历了其中一个关系对集合，查询复杂度可以视为`O(n)`。

# Hash Join
现在我们同样有两个关系对集合`R`和`S`，一个连接谓词`JP(r, s) := r.x == s.x`。
下面用伪代码来表述**Simple Hash Join**的处理过程：
```python
# R
# S
# JP(r, s) := r.x == s.x

def simple_hash_join(R, S, JP(r, s)):
    indexOnRX := build_ht(R.x)
    for s in S:
        query_result_set = IndexOnRS(s.x)
        if query_result_set != None:
            output({s} * query_result_set)
```

可以看出在循环和判定上，SHJ和INLJ非常相似，最大的区别在于SHJ使用了一个临时生成的Hash内存表作为`IndexOnRS`依据。由于使用Hash Table，因此SHJ相比之前的NLJ而言缺少了非等值查询的支持。同时在查询复杂度上，SHJ先遍历其中一个关系对集合生成HT，再遍历另一个关系对进行判定，因此为`O(m + n)`

# Sort-Merge Join
现在我们有两张表`Customer`和`City`，我们需要通过`Join`操作获取每个customer所在的city。

| name | street | cityID |   |   |   | cityID | city |
|------|--------|--------|---|---|---|--------|------|
| n1   | s1     | 0      |   |   |   | 0      | A    |
| n2   | s2     | 0      |   |   |   | 1      | B    |
| n3   | s3     | 0      |   |   |   | 3      | C    |
| n4   | s4     | 1      |   |   |   | 5      | D    |
| n5   | s5     | 1      |   |   |   | 7      | E    |
| n6   | s6     | 5      |   |   |   | 9      | F    |
| n7   | s7     | 5      |   |   |   |        |      |
| n8   | s8     | 7      |   |   |   |        |      |
| n9   | s9     | 9      |   |   |   |        |      |
| n10  | s10    | 9      |   |   |   |        |      |
| n11  | s11    | 9      |   |   |   |        |      |

我们按照以下原则进行**Merge Join**：
- 使用两个指针，分别指向两表的`cityID`字段
- 当指针对应的值相等时，output一行结果
- 指针需要移动至下一行时，优先移动当前指向值较小的指针，若值相等时优先移动指向`Customer`表的指针

输出结果为：

| name | street | cityID | city |
|------|--------|--------|------|
| n1   | s1     | 0      | A    |
| n2   | s2     | 0      | A    |
| n3   | s3     | 0      | A    |
| n4   | s4     | 1      | B    |
| n5   | s5     | 1      | B    |
| n6   | s6     | 5      | D    |
| n7   | s7     | 5      | D    |
| n8   | s8     | 7      | E    |
| n9   | s9     | 9      | F    |
| n10  | s10    | 9      | F    |
| n11  | s11    | 9      | F    |


注意到这种Merge操作的前提是两表的连接谓词所在字段必须是排序的，因此这种连接算法称为**Sort-Merge Join**。

根据以上条件写出伪代码：
```python
# R
# S
# JP(r, s)

def sort_merge_join(R, S, JP(r, s)):
    sort(R on R.x)
    sort(S on S.x)
    pointer_r = R[0]
    pointer_s = S[0]

    while pointer_r != R.end and pointer_s != S.end:
        if pointer_r.x == pointer_s.x:
            output((pointer_r, pointer_s))

        if pointer_r.x <= pointer_s.x:
            pointer_r++
        else:
            pointer_s++

    # handle rest row here
```

要注意如果将`Customer`表与`City`表位置互换，并且保持优先移动左表(`City`)表指针，我们输出的结果将会是不正确的。

另外，如果两表的排序字段都不是主键或唯一，输出的结果也会有遗漏的case。

因此，**Sort-Merge Join**算法有必须满足的前提条件：
- Join字段在其中一个表中必须唯一
- 保持优先移动ref表（即Join字段非唯一的表）

最后估算查询复杂度，需要先进行排序，为`O(nlogn)`的复杂度，然后双指针遍历两表为`O(m+n)`的复杂度。

# Hash Join Implementation in MySQL
## In-memory Hash Join
经典HJ算法包含两个阶段：
- build phase：指定两个需要Join的表其中之一为"build"表，读入内存中的Hash Table中，其中Hash值以EquiJoin的字段计算。
- probe phase：另一个表作为Probe输入，当build阶段结束后，逐行读取Probe表，同样以Join的字段计算该行的Hash值，并在内存中的HT中进行查找，对于每个命中的匹配，输出一行Join结果。

这种方案要求"build"输入可以被完整加载进内存，如果不行，可以通过以下方式来解决：
- 从"build"表读取尽可能多的行数加载进内存
- 遍历整个"probe"表，输出匹配结果
- 清理In-memory的HT
- 继续读取"build"表的下一部分，重复操作

这种方式缺点在于我们需要多次读"probe"表，所以不是理想的解决方案。因此我们还有On-disk的HJ方案。

## On-disk Hash Join
On-disk HJ将两表都进行分块(partitioning)，并且要让每个块都可以完整加载进内存的HT中。分块同样由一个Hash Function计算Join列的Hash值来决定。分块完毕后，加载"build"表的第一个块进内存，并且遍历"probe"表的第一个块进行匹配。由于都是通过相同的Hash Function计算结果进行分块，我们可以知道Hash值相同的行肯定会落入同一块中。

注意这里要使用不同的Hash Function来计算分块用的Hash值与匹配用的Hash值，否则要么我们会得到很差的分块结果，要么会得到很差的HT。

当第一对的分块处理完后，清理内存中的HT缓存，继续加载后续的分块文件，遍历对应的"probe"块进行匹配，直到所有的分块处理完毕。

如果我们的数据集是非常不均匀的，某些"build"块可能就会放不进内存中，这种情况下，我们在In-memory Hash Join中介绍的逐段读取进内存，并多次扫"probe"块的思路就会被使用到，尽管它并不是最理想的方案。

# Conclusion
Join操作使用的基础算法：
- Nested Loop Join，可以视为两表相乘，嵌套循环逐行检查是否满足Join条件
- Indexed Nested Loop Join，遍历部分表，并用遍历结果作为索引查询条件，借助索引查询
- Hash Join，可以视为INLJ的特殊情况，区别在于Hash Join使用现有或临时生成的Hash值作为索引，此时不支持非等值连接
- Sort-Merge Join，限定条件比较多，先根据Join字段排序，再合并两表，同样不支持非等值连接，否则会退化为NLJ