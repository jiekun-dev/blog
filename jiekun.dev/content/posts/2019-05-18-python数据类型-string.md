---
title: Python数据类型——String
author: 小黄鸭
type: post
date: 2019-05-18T14:23:16+00:00
excerpt: Python"字符"的"串"
featured_image: /wp-content/uploads/2020/02/str.jpg
rank_math_primary_category:
  - 6
rank_math_seo_score:
  - 10
rank_math_internal_links_processed:
  - 1
categories:
  - Python
archieved: true

---
又隔了好久好久没有写博客，过完了春节元宵清明五一一大堆节日，今年就没了一半了，所以需要抓紧时间充实一下。立好Flag以后看书一定要随时笔记不然厚厚的书翻起来跟天书一样难找。

# 前言

其实这是一篇读书笔记，主要是关于Python的几种基础数据类型，包括顺序结构（List、Tuple等）、哈希结构（Dict、Set等）以及文本和Bytes。这些每天都在打交道的类型其实并不像看上去的简单，简单了解一下背后的理论和一些相关的使用技巧有助于平时编码中提高效率和写出（没）优（人）雅（懂）的代码。  
原书里面大概很多文字都没（看）什（不）么（懂）用，所以文章尽可能附上相关代码方便理解。

# 文本与Bytes

Python3将Python2的万能的`str`分成了`text`类型（Unicode）和`bytes`类型，反人类的拆分背后隐藏着怎样的秘密？

## 什么是字符串

字符串就是“字符”的“串”，问题在于什么是“字符”。  
两个概念：

  * 字符的标识，即`码位`(code point)，是0-1114111的数字构成的，在Unicode中使用4-6个十六进制的数字标示，前缀U+。例如：A-U+0041，€-U+20AC。
  * 代表字符的byte的表示方式取决于具体编码。例如：A-U+0041在UTF-8中编码成单个字节\x41，在UTF-16LE中编码为两个字节\x41\x00。

码位(code points)转字节序(bytes)叫`编码`，反之叫`解码`

```
'你好'
>>> b = s.encode('UTF-8') # 
>>> b  # 这是个bytes对象
b'\xe4\xbd\xa0\xe5\xa5\xbd'
>>> b.decode('UTF-8')
'你好'
>>> b.decode('UTF-16LE')
'뷤\ue5a0붥'
>>> len(s)
2
>>> len(b)
6
```
## 字节

震惊，下面这串东西的[0]和[0:1]的结果竟然不一样？

```
my_char = bytes('很好玩的代码', encoding='utf_8')
>>> my_char
b'\xe5\xbe\x88\xe5\xa5\xbd\xe7\x8e\xa9\xe7\x9a\x84\xe4\xbb\xa3\xe7\xa0\x81'
>>> my_char[0]
229
>>> my_char[0:1]
b'\xe5'
```
好吧本来以为是`str`类型很好玩，其实Python里面的其他顺序类型也是这么做的，指定某个index的时候返回的是对应的元素，[a:b]切片的时候返回的是同类型的序列，只是`str`类型看起来像是比较奇怪，只返回了值。

```
1 = ['好', '玩', '的', '代', '码']
>>> l1[0]
'好'
>>> l1[0:1]
['好']
```
二进制序列的表示方法有几种，如果不知道什么叫二进制序列的表示方法，请看：

```
'cafe咖啡'
>>> s.encode('utf-8')
b'cafe\xe5\x92\x96\xe5\x95\xa1'
>>> s = """
... have
... a 
... test
... """
>>> s.encode('utf-8')
b'\nhave\na \ntest\n'
```
cafe\xe5\x92\x96\xe5\x95\xa1

  * cafe和原文的cafe完全一致，因此二进制序列可以是ASCII字符本身
  * 换行、回车等和\对应的字节，使用\n、\t等表示
  * 其他字节的值，比如`咖啡`，使用十六进制转义序列

除了少数方法（`format`、`format_map`、`casefold`、`isnumeric`等）以外，`str`类型的方法同样支持`bytes`和`bytearray`类型。例如`endswith`、`replace`等。`re`模块中的正则表达式也能处理二进制序列。  
二进制序列有个类方法是`str`没有的，`fromhex`，用于从十六进制数字对构建二进制序列。（大概这东西也没什么用）

## Struct和Memory Views

`struct`提供了一些把字节序列(\xe5\x96)转换成不同类型字段组成的元组的方法和逆向方法。`struct`模块可以处理`bytes` 、`bytearray`和`memoryview`对象。

```
import struct
>>> fmt = '&lt;3s3sHH'
>>> with open('filter.gif', 'rb') as fp:
...     img = memoryview(fp.read())  # 使用文件创建memoryview对象
...
>>> header = img[:10]  # memoryview切片再创建一个memoryview对象，但是memoryview的切片是共享内存地址的
>>> bytes(header)
b'GIF89a+\x02\xe6\x00'
>>> struct.unpack(fmt, header)  # 按照fmt规则unpack这个memoryview对象，得到一个元组
(b'GIF', b'89a', 555, 230)
>>> del header
 del img
```
虽然搞不懂有什么用不过看起来很厉害就是了。

## 编码器/解码器

这个小节最核心的点就是**不要依赖系统的编码**。因为不同系统的编码不一致，依赖系统编码会导致你的Python代码在不同系统上运行结果不一致。手动指定每次的编码器/解码器可以避免这个问题。

很显然不同的编码器对同一段字符串的编码得到的字节序列差异很大。开头已经说过，代表字符的byte的表示方式取决于具体编码：

```
for codec in ['utf_8', 'utf_16', 'latin_1']:
...     print(codec, 'This is 同样的文字'.encode(codec), sep='\t')
... 
utf_8    b'This is \xe5\x90\x8c\xe6\xa0\xb7\xe7\x9a\x84\xe6\x96\x87\xe5\xad\x97'  # UTF-8编码结果
utf_16    b'\xff\xfeT\x00h\x00i\x00s\x00 \x00i\x00s\x00 \x00\x0cT7h\x84v\x87eW['  # UTF-16编码结果
Traceback (most recent call last):
  File "&lt;stdin>", line 2, in &lt;module>
UnicodeEncodeError: 'latin-1' codec can't encode characters in position 8-12: ordinal not in range(256)  # Lartin-1不支持中文
```
不支持中文肯定不行，所以有一些处理`UnicodeEncodeError`的方法，包括将不支持的字符转**跳过**、**替换**。

```
int(codec, 'This is 同样的文字'.encode('latin_1', errors='ignore'), sep='\t')
latin_1    b'This is '
>>> print(codec, 'This is 同样的文字'.encode('latin_1', errors='replace'), sep='\t')
latin_1    b'This is ?????'
>>> print(codec, 'This is 同样的文字'.encode('latin_1', errors='xmlcharrefreplace'), sep='\t')
latin_1    b'This is &#21516;&#26679;&#30340;&#25991;&#23383;'
```
对应的，如果要将`bytes`解码，也会有UnicodeDecodeError。

```
'\xff\xfeT\x00h\x00i\x00s\x00 \x00i\x00s\x00 \x00\x0cT7h\x84v\x87eW['
>>> b.decode('utf-8')
Traceback (most recent call last):
  File "&lt;stdin>", line 1, in &lt;module>
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xff in position 0: invalid start byte  # 解码不了
>>> b.decode('utf-8', errors='replace')
'��T\x00h\x00i\x00s\x00 \x00i\x00s\x00 \x00\x0cT7h�v�eW['  # 代替
>>> b.decode('utf-8', errors='ignore')
'T\x00h\x00i\x00s\x00 \x00i\x00s\x00 \x00\x0cT7hveW['  # 忽略
```
## 使用预期之外的编码抛出SyntaxError

文件顶部加上注释

```
# coding: cp1252
```
Python3默认使用UTF-8，GUN/Linux和OS X默认都是UTF-8，Windows则不是，所以可能会报这种反人类的错误。因此，和前面说的一样，不要依赖系统编码，全部手动指定可以让你的脚本正常运行于不同系统。

## 怎样才能知道一段字节序列的编码

不可能。  
不过可以通过某种编码的特定模式来**猜测**（也就是没有100%准确的方案，a.k.a 不可能）对应编码

## BOM

```
u16_en = 'El Niño'.encode('utf_16')
>>> u16_cn = '有鬼'.encode('utf_16')
>>> u16_en
b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'
>>> u16_cn
b'\xff\xfe\tg&lt;\x9b' 
```
奇怪，好像两段没有一个字相同的字符，但经过编码后的字节序列都是以`\xff\xfe`开头的。没错这就是`BOM`(byte-order mark)，指明编码时使用小字节序（little-endian byte ordering）。  
小字节序中，字母E的位码是U+0045，在字节便宜的第二位和第三位的编码为69和0；而大字节序中是编码顺序是相反的，E的编码为0和69。  
因此需要区分开小字节序系统和大字节序系统。因为按照设计,U+FFFE 字符不存在，在小字节序编码中,字节序列 b&#8217;\xff\  
xfe&#8217; 必定是 ZERO WIDTH NO-BREAK SPACE ,所以编解码器知道该用哪个字节序。

## 处理文本文件

处理文本的原则遵照“Unicode三明治”，就是尽可能早地把输入的字节序列转为字符串，让逻辑层只处理字符串对象。

```
n('cafe.txt', 'w', encoding='utf_8').write('café')
4
 open('cafe.txt').read()
```
如果在Windows上，最后的输出可能就不是`café`了，因为首次打开文件的时候指定了UTF-8编码，而再次打开的时候没有指定编码，则会依照系统的默认编码。Linux上默认均为UTF-8，会给人一种代码没有问题的假象，实际上并不是这样的。  
另外，如果在open的参数中声明是在二进制模式中读取文件，将会得到一个`BufferedReader`对象，而正常情况下会得到一个`TextIOWrapper`对象。

```
f = open('cafe.txt','rb')
>>> f
&lt;_io.BufferedReader name='cafe.txt'>
>>> f = open('cafe.txt','r')
>>> f
&lt;_io.TextIOWrapper name='cafe.txt' mode='r' encoding='UTF-8'>
```
打开文件时没有指定encoding参数，编码会由`locale.getpreferredencoding()`指定，类似的还有一个用于编解码文件名的方法`sys.getfilesystemencoding()`。

## 字符串对比

先看一段代码：

```
1 = 'café'
>>> s2 = 'cafe\u0301'
>>> s1, s2
('café', 'café')
>>> s1 == s2
False
>>> len(s1), len(s2)
(4, 5)
```
看到两种表示的`café`并不相等，因为Python看到的是不同的码位序列。解决办法是将Unicode规范化，使用`unicodedata.normalize`。  
`normalize`有4种参数: `NFC`，`NFD`，`NFKC`，`NFKD`。前两个分别对应“使用最少的bytes构成等价字符串”和“把字符串分解成基本字符和单独的组合字符”，也就是类似于`café`和`cafe\u0301`两种形式；后两个分别是前两个的“兼容分解”模式，`K`表示“compatibility”，这样做格式会有所损失，例如`1⁄2`（这实际上是一个字符，1在上2在下）经过“兼容分解”后会变成`1/2`（这是3个字符）。  
接上面的代码，经过规范化后对比返回`True`：

```
from unicodedata import normalize
>>> s1 == normalize('NFC', s2)
True
>>> s2 == normalize('NFD', s1)
True
```
## 大小写折叠(Case Fold)

字符串的`casefold`方法将文本变为小写，但是对比`lower`方法更加暴力

```
'ß'.casefold()
'ss'  # ß在德语中是“sharp s”
>>> 'ß'.lower()
'ß'
```
## 规范化总结

规范化待比较的字符串使用`NFC`，不区分大小写使用casefold

## 极端的“规范化”

如何将`São Paulo`规范化成`Sao Paulo`，尽管这样做会丢失信息？  
作者的骚操作，不再展示，可以参考原书4.6.3节。

## Unicode的文本排序

这里有个错误的排序：

```
fruits = ['caju', 'atemoia', 'cajá', 'açaí', 'acerola']
>>> sorted(fruits)
['acerola', 'atemoia', 'açaí', 'caju', 'cajá']
```
期望得到的结果是`ç`按照`c`排序，`á`按照`a`排序：

```
['açaí', 'acerola', 'atemoia', 'cajá', 'caju']
```
非ASCII文本的标准排序方式是使用`locale.strxfrm`函数，这个函数的结果跟当前所在区域有关，通过使用`locale.setlocale()`改变所在区域以达到按照特定区域的习惯排序的效果。

```
import locale
>>> locale.setlocale(locale.LC_COLLATE, 'zh_CN.UTF-8')
'zh_CN.UTF-8'
>>> fruits = ['caju', 'atemoia', 'cajá', 'açaí', 'acerola']
>>> sorted(fruits, key=locale.strxfrm)
['açaí', 'acerola', 'atemoia', 'cajá', 'caju']
```
## Unicode 数据库

Unicode标准提供了一个完整的数据库，包括码位和字符名称之间的映射、各字符的元数据、字符之间的关系。