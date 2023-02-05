---
title: "2023 年初的富途牛牛面试复盘"
date: 2023-02-05T11:45:58+08:00
archieved: true
---

## 面试
### 一面（1 小时 40 分钟）
面试官先介绍了富途的面试流程，技术面试一共分为 3 轮，其中第一轮是考察基础、数据结构算法、编程基础、计算机理论，二三轮会继续拓展广度和深度，通过后续还会有 HR 面。
- 介绍一下近两年工作使用的语言、框架、使用的存储组件？
- Golang 用了多少年？
- 找工作的原因和背景？

Go 基础
- 切片和数组有什么区别？
- 切片底层数据结构、扩容时机和策略？
- 问一段代码输出结果（具体代码记不清楚了，但拟了一段类似代码）？
```go
func main() {
	arr := []int{1}

	myfunc1(arr)
	fmt.Println(arr)

	arr = append(arr, 3)
	arr = append(arr, 4)
	myfunc2(arr)
	fmt.Println(arr)
}

func myfunc1(arr []int) {
	arr = append(arr, 2)
	arr[0] = 0
	fmt.Println(arr)
	return
}

func myfunc2(arr []int) {
	arr = append(arr, 5)
	arr[0] = 9
	fmt.Println(arr)
	return
}
```
- 已知元素长度申请切片的时候应该怎么做？
- defer 方法执行顺序是、（参数确定的）时机怎么样的？
- 问另一段代码输出结果（也同样是拟了一段类似代码）？
```go
func main() {
	fmt.Println(test1())
	fmt.Println(test2())
	fmt.Println(test3())
	fmt.Println(test4())

	return
}

func test1() (v int) {
	defer fmt.Println(v)
	return v
}

func test2() (v int) {
	defer func() {
		fmt.Println(v)
	}()
	return 3
}

func test3() (v int) {
	defer fmt.Println(v)
	v = 3
	return 4
}

func test4() (v int) {
	defer func(n int) {
		fmt.Println(n)
	}(v)
	return 5
}
```
- 什么是哈希函数？哈希函数有什么特性？
- Golang 标准库中 map 的底层数据结构是什么样子的？
- Map 的查询时间复杂度如何分析？
- 极端情况下有很多哈希冲突，Golang 标准库如何去避免最坏的查询时间复杂度？
- Golang map Rehash 的策略是怎样的？什么时机会发生 Rehash？
- Rehash 具体会影响什么？哈希结果会受到什么影响？
- Rehash 过程中存放在旧桶的元素如何迁移？
- 并发环境共享同一个 map 是安全的吗？
    - panic
- 如果并发环境想要用这种哈希容器有什么方案？
    - sync.Mutex / sync.RWMutex
    - sync.Map
- 加锁存在什么问题呢？
- sync.Map 比加锁的方案好在哪里，它的底层数据结构是怎样的？
    - 缓存 + map 组成的结构
    - 底层 map 的操作依然是加锁的，但是读的时候使用上缓存可以增加并发性能
- sync.Map 的 Load() 方法流程？
- sync.Map Store() 如何保持缓存层和底层 Map 数据是相同的? 是不是每次执行修改都需要去加锁？
- 一致性哈希了解吗？
- channel 被 close 操作之后进行读和写会有什么问题？
- 未被初始化的 channel 进行读写会有什么问题？
- channel 底层数据结构是怎样的，尝试用结构体来表述一下？

写代码
- 给定一个正整数数组 arr，求 arr[i] / arr[j] 的最大值，其中 i < j。

网络
- TIME_WAIT 状态出现 TCP 的哪个阶段（或者场景）？
- 查看 TCP 连接状态需要用什么命令？
- 发现存在大量 TIME_WAIT 状态会存在什么问题？
- 出现大量 TIME_WAIT 的话应用层有什么优化方案？
- 连接池的中心思想是什么？主要解决的是什么问题？
- 了解 TCP 中的粘包现象吗？
- 如果有一个请求需要发送数据，但是我想把包拆分开（不等待），在应用层应该怎么做？

存储
- 如果一个表中有主键和普通索引，查询命中主键和普通索引有什么区别？
- 聚簇索引具体是个什么样的数据结构？
- 这种数据结构对于范围查找有什么好处？
- InnoDB 中的隔离级别有哪几种？
- Read UnCommitted 存在什么问题？
- Read Committed 解决了什么问题又存在什么问题？
- Repeatable Read 会面临什么问题？
- 什么是幻读现象？如果另一个事务插入了一个条的数据可以被读取到吗？
- Redis 的哈希数据类型，想插入元素，底层数据结构是怎样变化？
- 哈希数据类型一定会用上哈希表吗？
- ziplist 的存储结构是怎样的？
- Redis 的持久化机制有哪几种？
- RDB 的原理是什么，用在哪些场景？
- AOF 的原理和使用场景呢？
- AOF 对于写入特别频繁的场景会存在什么问题？
- AOF 在做压缩的时候 Redis 对外是否正常提供服务？

写代码
- 给定 n 个不同元素的数组，设计算法等概率取 m 个不同的元素

反问
- 了解面试部门的基本情况？
- 部门内微服务的数量？每个研发大概会负责多少服务的开发工作？
- C++ 和 Golang 在团队内的使用场景和比例？
- 研发对开发质量的保障是如何完成的，测试和覆盖率是否有要求？
- 研发和测试的人员配置比例？
- 前面很多问题都没有答对，面试官的建议？