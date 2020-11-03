---
title: 流畅的Python学习笔记-02序列构成的数组-2-4
date: 2020-11-03 14:01:00
categories:
- [流畅的Python, 第二章 序列构成的数组]
tags: 
- 学习笔记
---

第二章第四节：切片

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201028144038.png)

<!-- more -->

在 Python 里，像列表（list）、元组（tuple）和字符串（str）这类序列类型都支持切片操作，但是实际上切片操作比人们所想象的要强大很多。

这一节主要讨论的是这些高级切片形式的用法，它们的实现方法则会在第 10 章的一个自定义类里提到。这么做主要是为了符合这本书的哲学：先讲用法，第四部分中再来讲如何创建新类。

### 为什么切片和区间会忽略最后一个元素

Python 在切片和区间操作里不包含区间范围的最后一个元素，这个习惯符合 Python、C 和其他语言里以 0 作为起始下标的传统。这样做带来的好处如下。

* 当只有最后一个位置信息时，我们也可以快速看出切片和区间里有几个元素：range(3) 和 my_list[:3] 都返回 3 个元素。

* 当起止位置信息都可见时，我们可以快速计算出切片和区间的长度，用后一个数减去第一个下标（stop - start）即可。

* 这样做也让我们可以利用任意一个下标来把序列分割成不重叠的两部分，只要写成 my_list[:x] 和 my_list[x:] 就可以了，如下所示：

```python
>>> l = [10, 20, 30, 40, 50, 60]
>>> l[:2] # 在下标2的地方分割
[10, 20]
>>> l[2:]
[30, 40, 50, 60]
>>> l[:3] # 在下标3的地方分割
[10, 20, 30]
>>> l[3:]
[40, 50, 60]
```

计算机科学家 `Edsger W. Dijkstar` 对这一风格的解释应该是最好的，详见“延伸阅读”中给出的最后一个参考资料。接下来进一步看看 Python 解释器是如何理解切片操作的。

### 对对象进行分片

一个众所周知的秘密是，我们还可以用 `s[a:b:c]` 的形式对 s 在 a 和 b之间以 c 为间隔取值。c 的值还可以为负，负值意味着反向取值。下面的 3 个例子更直观些：

```python
>>> s = 'bicycle'
>>> s[::3]
'bye'
>>> s[::-1]
'elcycib'
>>> s[::-2]
'eccb'
```

另一个例子是在第 1 章中用 deck[12::13] 的形式在未洗过的牌里把每种花色的 A 拿出来：

```python
>>> deck[12::13]
[Card(rank='A', suit='spades'), Card(rank='A', suit='diamonds'),
Card(rank='A', suit='clubs'), Card(rank='A', suit='hearts')]
```

想要看`deck`具体是什么数据，请查看该[链接](https://wuwenlun.top/2020/10/22/%E6%B5%81%E7%95%85%E7%9A%84Python%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-01%E6%95%B0%E6%8D%AE%E6%A8%A1%E5%9E%8B-1-1/)

`a:b:c` 这种用法只能作为索引或者下标用在 `[]` 中来返回一个切片对象：`slice(a, b, c)`，通过内置函数`slice`返回切片对象。在 10.4.1 节中会讲到，对seq[start:stop:step] 进行求值的时候，Python 会调用`seq.__getitem__(slice(start, stop, step))`。就算你还不会自定义序列类型，了解一下切片对象也是有好处的。例如你可以给切片命名，就像电子表格软件里给单元格区域取名字一样。

比如，要解析示例 2-11 中所示的纯文本文件，这时使用有名字的切片比用硬编码的数字区间要方便得多，注意示例里的 for 循环的可读性有多强。

示例 2-11 纯文本文件形式的收据以一行字符串的形式被解析：

```python
invoice = """
0.....6................................40........52...55........
1909  Pimoroni PiBrella                $17.50    3    $52.50
1489  6mm Tactile Switch x20           $4.95     2    $9.90
1510  Panavise Jr. - PV-201            $28.00    1    $28.00
1601  PiTFT Mini Kit 320x240           $34.95    1    $34.95
"""
SKU = slice(0, 6)
DESCRIPTION = slice(6, 40)
UNIT_PRICE = slice(40, 52)
QUANTITY = slice(52, 55)
ITEM_TOTAL = slice(55, None)
line_items = invoice.split('\n')[2:]
for item in line_items:
print(item[UNIT_PRICE], item[DESCRIPTION])

>>> $17.50 Pimoroni PiBrella
>>> $4.95 6mm Tactile Switch x20
>>> $28.00 Panavise Jr. - PV-201
>>> $34.95 PiTFT Mini Kit 320x240
```

在 10.4 节还有更多机会来了解切片（slice）对象。如果从 Python 用户的角度出发，切片还有个两个额外的功能：多维切片和省略表示法（`...`）

### 多维切片和省略

[] 运算符里还可以使用以逗号分开的多个索引或者是切片，外部库 NumPy 里就用到了这个特性，二维的 `numpy.ndarray` 就可以用 `a[i,j]` 这种形式来获取，抑或是用 a[m:n, k:l] 的方式来得到二维切片。稍后的示例 2-22 会展示这个用法。要正确处理这种 [] 运算符的话，对象的特殊方法 `__getitem__` 和 `__setitem__` 需要以元组的形式来接收 `a[i, j]` 中的索引。也就是说，如果要得到 `a[i, j]` 的值，Python 会调用 `a.__getitem__((i, j))`。

<font color=red>**Python 内置的序列类型都是一维的，因此它们只支持单一的索引，成对出现的索引是没有用的。**</font>

省略（ellipsis）的正确书写方法是三个英语句号（...），省略在 Python 解析器眼里是一个符号，而实际上它是 Ellipsis 对象的别名，而 Ellipsis 对象又是 ellipsis 类的单一实例。 它可以当作切片规范的一部分，也可以用在函数的参数清单中，比如 `f(a,..,z)`，或 `a[i:...]`。在 NumPy 中，`...` 用作多维数组切片的快捷方式。如果 x 是四维数组，那么 `x[i, ...]` 就是 `x[i,:,:,:]` 的缩写。如果想了解更多，请参见“Tentative NumPy Tutorial”（[http://wiki.scipy.org/Tentative_NumPy_Tutorial](http://wiki.scipy.org/Tentative_NumPy_Tutorial)）。

> 是的，你没看错，ellipsis 是类名，全小写，而它的内置实例写作 Ellipsis。这其实跟 bool 是小写，但是它的两个实例写作 True 和 False 异曲同工。

在写这本书的时候，<font color=red>**我还没有发现在 Python 的标准库里有任何Ellipsis 或者是多维索引的用法**</font>。如果你知道，请告诉我。这些句法上的特性主要是<font color=red>**为了支持用户自定义类或者扩展，比如 NumPy 就是个例子**</font>。

除了用来提取序列里的内容，切片还可以用来就地修改可变序列，也就是说修改的时候不需要重新组建序列。

### 给切片赋值

如果把切片放在赋值语句的左边，或把它作为 del 操作的对象，我们就可以对序列进行嫁接、切除或就地修改操作。通过下面这几个例子，你应该就能体会到这些操作的强大功能：

```python
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> l[2:5] = [20, 30]
>>> l
[0, 1, 20, 30, 5, 6, 7, 8, 9]
>>> del l[5:7]
>>> l
[0, 1, 20, 30, 5, 8, 9]
>>> l[3::2] = [11, 22]
>>> l
[0, 1, 20, 11, 5, 22, 9]
>>> l[2:5] = 100 ➊
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: can only assign an iterable
2
2
>>> l[2:5] = [100]
>>> l
[0, 1, 100, 22, 9]
```

➊ 如果赋值的对象是一个切片，那么赋值语句的右侧必须是个可迭代对象。即便只有单独一个值，也要把它转换成可迭代的序列。

序列的拼接操作可谓是众所周知，任何一本 Python 入门教材都会介绍 `+` 和 `*` 的用法，但是在这些用法的背后还有一些可能被忽视的细节。下面就来看看这两种操作。

