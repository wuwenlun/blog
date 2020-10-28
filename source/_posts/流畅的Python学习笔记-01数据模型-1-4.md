---
title: 流畅的Python学习笔记-01数据模型-1-4
date: 2020-10-28 12:30:27
categories:
- [流畅的Python, 第一章 Python的数据模型]
tags: 
- 学习笔记
---

第一章第四节：为什么`len()`不是普通方法

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201028144038.png)

<!-- more -->

### 为什么`len()`不是普通方法

我在 2013 年问核心开发者 Raymond Hettinger 这个问题时，他用“`Python`之禅”（[https://www.python.org/doc/humor/#the-zen-of-python](https://www.python.org/doc/humor/#the-zen-of-python)）里的原话回答了我：“实用胜于纯粹。”在 1.2 节里我提到过，如果 x 是一个[内置类型](https://docs.python.org/zh-cn/3/library/stdtypes.html)的实例，那么 `len(x)` 的速度会非常快。背后的原因是 CPython 会直接从一个 C 结构体里读取对象的长度，完全不会调用任何方法。获取一个集合中元素的数量是一个很常见的操作，在`str`、`list`、`memoryview` 等类型上，这个操作必须高效。

<!-- more -->

换句话说，len 之所以不是一个普通方法，是为了让 Python 自带的数据结构可以走后门，abs 也是同理。但是多亏了它是特殊方法，我们也可以把 len 用于自定义数据类型。这种处理方式在保持内置类型的效率和保证语言的一致性之间找到了一个平衡点，也印证了“Python 之禅”中的另外一句话：“不能让特例特殊到开始破坏既定规则。”

如果把 abs 和 len 都看作一元运算符的话，你也许更能接受它们——虽然看起来像面向对象语言中的函数，但实际上又不是函数。有一门叫作 ABC 的语言是 Python 的直系祖先，它内置了一个\# 运算符，当你写出 #s 的时候，它的作用跟 len 一样。如果写成x#s 这样的中缀运算符的话，那么它的作用是计算 s 中 x 出现的次数。在 Python 里对应的写法是 s.count(x)。注意这里的 s 是一个序列类型。