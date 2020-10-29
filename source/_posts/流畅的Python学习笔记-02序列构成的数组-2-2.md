---
title: 流畅的Python学习笔记-02序列构成的数组-2-2
date: 2020-10-29 13:47:30
categories:
- [流畅的Python, 第二章 序列构成的数组]
tags: 
- 学习笔记
---

第二章第一节：列表推导和生成器表达式

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201028144038.png)

<!-- more -->

列表推导是构建列表（list）的快捷方式，而生成器表达式则可以用来创建其他任何类型的序列。如果你的代码里并不经常使用它们，那么很可能你错过了许多写出可读性更好且更高效的代码的机会。

如果你对我说的“更具可读性”持怀疑态度的话，别急着下结论，我马上就能说服你。

很多 Python 程序员都把列表推导（list comprehension）简称为 listcomps，生成式表达器（generator expression）则称为genexps。我有时也会这么用。

### 列表推导和可读性

先来个小测试，你觉得示例 2-1 和示例 2-2 中的代码，哪个更容易读懂？

示例 2-1 把一个字符串变成 Unicode 码位的列表：

```python
symbols = '$¢£¥€¤'
codes = []
for symbol in symbols:
  codes.append(ord(symbol))
print(codes)

>>>[36, 162, 163, 165, 8364, 164]
```

示例 2-2 把字符串变成 Unicode 码位的另外一种写法：

```python
symbols = '$¢£¥€¤'
codes = [ord(symbol) for symbol in symbols]
print(codes)

>>>[36, 162, 163, 165, 8364, 164]
```

虽说任何学过一点 Python 的人应该都能看懂示例 2-1，但是我觉得如果学会了列表推导的话，示例 2-2 读起来更方便，因为这段代码的功能从字面上就能轻松地看出来。

for 循环可以胜任很多任务：遍历一个序列以求得总数或挑出某个特定的元素、用来计算总和或是平均数，还有其他任何你想做的事情。在示例 2-1 的代码里，它被用来新建一个列表。

另一方面，列表推导也可能被滥用。<font color=red>通常的原则是，只用列表推导来创建新的列表，并且尽量保持简短</font>。如果列表推导的代码超过了两行，你可能就要考虑是不是得用 for 循环重写了。就跟写文章一样，并没有什么硬性的规则，这个度得你自己把握。

Python 会忽略代码里 []、{} 和 () 中的换行，因此如果你的代码里有多行的列表、列表推导、生成器表达式、字典这一类的，可以省略不太好看的续行符 `\`。

列表推导不会再有变量泄漏的问题。

Python 2.x 中，在列表推导中 for 关键词之后的赋值操作可能会影响列表推导上下文中的同名变量。像下面这个 Python 2.7 控制台对话：

```python
Python 2.7.6 (default, Mar 22 2014, 22:59:38)
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> x = 'my precious'
>>> dummy = [x for x in 'ABC']
>>> x
'C
```

如你所见，x 原本的值被取代了，但是这种情况在 Python 3 中是不会出现的。

列表推导、生成器表达式，以及同它们很相似的集合（set）推导和字典（dict）推导，在 Python 3 中都有了自己的局部作用域，就像函数似的。表达式内部的变量和赋值只在局部起作用，表达式的上下文里的同名变量还可以被正常引用，局部变量并不会影响到它们。

这是Python 3 代码：

```python
>>> x = 'ABC'
>>> dummy = [ord(x) for x in x]
>>> x ➊
'ABC'
>>> dummy ➋
[65, 66, 67]
>>>
```

➊ x 的值被保留了。

➋ 列表推导也创建了正确的列表。

列表推导可以帮助我们把一个序列或是其他可迭代类型中的元素过滤或是加工，然后再新建一个列表。Python 内置的 `filter` 和 `map` 函数组合起来也能达到这一效果，但是可读性上打了不小的折扣。

### 列表推导同filter和map的比较

filter 和 map 合起来能做的事情，列表推导也可以做，而且还不需要借助难以理解和阅读的 lambda 表达式。详见示例 2-3。

示例 2-3 用列表推导和 map/filter 组合来创建同样的表单：

```python
>>> symbols = '$¢£¥€¤'
>>> beyond_ascii = [ord(s) for s in symbols if ord(s) > 127]
>>> beyond_ascii
[162, 163, 165, 8364, 164]
>>> beyond_ascii = list(filter(lambda c: c > 127, map(ord, symbols)))
>>> beyond_ascii
[162, 163, 165, 8364, 164]
```

我原以为 map/filter 组合起来用要比列表推导快一些，Alex Martelli却说不一定——至少在上面这个例子中不一定。在本书的代码仓库（[https://github.com/fluentpython/example-code](https://github.com/fluentpython/example-code)）中有名为02-array-seq/listcomp_speed.py（[https://github.com/fluentpython/example-code/blob/master/02-array-seq/listcomp_speed.py](https://github.com/fluentpython/example-code/blob/master/02-array-seq/listcomp_speed.py)）的脚本，代码中有这两个方法的效率的比较。

第 5 章会更详细地讨论 map 和 filter。下面就来看看如何用列表推导来计算笛卡儿积：两个或以上的列表中的元素对构成元组，这些元组构成的列表就是笛卡儿积。

### 笛卡尔积

如前所述，用列表推导可以生成两个或以上的可迭代类型的笛卡儿积。笛卡儿积是一个列表，列表里的元素是由输入的可迭代类型的元素对构成的元组，因此笛卡儿积列表的长度等于输入变量的长度的乘积，如下图所示：

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201029144759.png)

上图含有 4 种花色和 3 种牌面的列表的笛卡儿积，结果是一个包含 12 个元素的列表。

如果你需要一个列表，列表里是 3 种不同尺寸的 T 恤衫，每个尺寸都有2 个颜色，示例 2-4 用列表推导算出了这个列表，列表里有 6 种组合。

示例 2-4 使用列表推导计算笛卡儿积：

```python
>>> colors = ['black', 'white']
>>> sizes = ['S', 'M', 'L']
>>> tshirts = [(color, size) for color in colors for size in sizes] ➊
>>> tshirts
[('black', 'S'), ('black', 'M'), ('black', 'L'), ('white', 'S'),
('white', 'M'), ('white', 'L')]
>>> for color in colors: ➋
... for size in sizes:
... print((color, size))
...
('black', 'S')
('black', 'M')
('black', 'L')
('white', 'S')
('white', 'M')
('white', 'L')
>>> tshirts = [(color, size) for size in sizes ➌
... for color in colors]
>>> tshirts
[('black', 'S'), ('white', 'S'), ('black', 'M'), ('white', 'M'),
('black', 'L'), ('white', 'L')]
```

➊ 这里得到的结果是先以颜色排列，再以尺码排列。

➋ 注意，这里两个循环的嵌套关系和上面列表推导中 for 从句的先后顺序一样。

➌ 如果想依照先尺码后颜色的顺序来排列，只需要调整从句的顺序。我在这里插入了一个换行符，这样顺序安排就更明显了。

<font color=red>**列表推导的作用只有一个：生成列表（list）。如果想生成其他类型的序列，生成器表达式就派上了用场。下一节就是对生成器表达式的一个简单介绍，其中可以看到如何用它生成列表以外的序列类型。**</font>

### 生成器表达式

虽然也可以用列表推导来初始化元组、数组或其他序列类型，但是生成器表达式是更好的选择。**这是因为生成器表达式背后遵守了迭代器协议，可以逐个地产出元素，而不是先建立一个完整的列表，然后再把这个列表传递到某个构造函数里**。前面那种方式显然能够节省内存。

生成器表达式的语法跟列表推导差不多，只不过把方括号换成圆括号而已。

示例 2-5 用生成器表达式初始化元组和数组：

```python
>>> symbols = '$¢£¥€¤'
>>> tuple(ord(symbol) for symbol in symbols) ➊
(36, 162, 163, 165, 8364, 164)
>>> import array
>>> array.array('I', (ord(symbol) for symbol in symbols)) ➋
array('I', [36, 162, 163, 165, 8364, 164])
```

➊ 如果生成器表达式是一个函数调用过程中的唯一参数，那么不需要额外再用括号把它围起来。

➋ array 的构造方法需要两个参数，因此括号是必需的。array 构造方法的第一个参数指定了数组中数字的存储方式。2.9.1 节中有更多关于数组的详细讨论。

示例 2-6 则是利用生成器表达式实现了一个笛卡儿积，用以打印出上文中我们提到过的 T 恤衫的 2 种颜色和 3 种尺码的所有组合。与示例 2-4不同的是，用到生成器表达式之后，内存里不会留下一个有 6 个组合的列表，因为生成器表达式会在每次 for 循环运行时才生成一个组合。如果要计算两个各有 1000 个元素的列表的笛卡儿积，生成器表达式就可以帮忙省掉运行 for 循环的开销，即一个含有 100 万个元素的列表。

示例 2-6 使用生成器表达式计算笛卡儿积：

```python
>>> colors = ['black', 'white']
>>> sizes = ['S', 'M', 'L']
>>> for tshirt in ('%s %s' % (c, s) for c in colors for s in sizes): ➊
... print(tshirt)
...
black S
black M
black L
white S
white M
white L
```

➊ 生成器表达式逐个产出元素，从来不会一次性产出一个含有 6 个 T恤样式的列表。

第 14 章会专门讲到生成器的工作原理。这里只是简单看看如何用生成器来初始化除列表之外的序列，以及如何用它来避免额外的内存占用。

### 如何理解可迭代对象、迭代器和生成器

在讨论可迭代对象、迭代器和生成器之前，先说明一下**迭代器模式（iterator pattern）**，维基百科这么解释：

> 迭代器是一种最简单也最常见的设计模式。它可以让用户透过特定的接口巡访容器中的每一个元素而不用了解底层的实现。

迭代是数据处理的基石。当内存中放不下数据集时，我们要找到一种**惰性**获取数据的方式，即按需一次获取一个数据项，这就是迭代器模式。

我们都知道序列是可迭代的。当解释器需要迭代对象x时，会自动调用iter(x)函数时。内置的iter函数有以下作用：

* 检查对象是否实现了`__iter__`方法，如果实现了就调用它，获得一个迭代器。
* 如果没有实现`__iter__`方法，但是实现了`__getitem__`方法，python会创建一个迭代器，尝试按顺序（从索引0开始）获取元素。
* 如果尝试失败，python会抛出`TypeError`异常，通常会提示"C object is not iterable"，其中C是目标对象所属的类。

截止到Python3.6，基本上所有的Python序列也都实现了`__getitem__`方法，这是保证任何序列都可迭代的原因。当然标准的序列也都实现了`__iter__`方法，之所以对`__getitem__`也可以创建迭代器是为了向后兼容，未来可能不在这么做。

但是，从Python3.4开始，检查x能否迭代，最准确的方法是调用iter(x)函数，如果不可迭代，再处理TypeError异常。

**使用iter内置函数可以获取<font color=red>迭代器对象</font>。**也就是说，如果一个对象实现了能返回迭代器的`__iter__`方法，那么对象就是可迭代的，序列都可以迭代；实现了`__getitem__`方法，而且其参数是从零开始的索引，这种对象也是可迭代的。

因此可以明确**可迭代对象**和**迭代器**之间的关系：**Python从可迭代的对象中获取迭代器。**

标准的迭代器接口有两个方法，即：

* `__next__`：返回下一个可用元素，如果没有元素，抛出StopIteration异常。

* `__iter__`：返回self，以便在应该使用可迭代对象的地方使用迭代器，比如for循环中。

因为迭代器只需`__next__`和`__iter__`两个方法，所以除了调用next()方法，以及捕获StopIteration异常之外，没有办法检查是否还有遗留的元素。此外，也没有办法还原迭代器。如果想再次迭代，那就要调用iter(…)，传入之前构建迭代器的可迭代对象。

构建`可迭代对象`和`迭代器`时经常会出现错误，原因是混淆了两者。要知道，`可迭代的对象`有个`__iter__`方法，**调用该方法每次都实例化一个新的迭代器**；而`迭代器`要实现`__next__`方法，返回单个元素，此外还要实现`__iter__`方法，返回迭代器本身(self)，如图。因此，`迭代器`可以迭代，但是`可迭代的对象`不是`迭代器`。

<u>**需要注意的是：**</u>

可迭代的对象必须实现`__iter__`方法，但**不能**实现`__next__`方法。另一方面，迭代器应该一直可以迭代，迭代器的`__iter__`方法应该返回自身。虽然可迭代对象和迭代器都有`__iter__`方法，但是两者的功能不一样，再次强调一下，可迭代对象的`__iter__`用于实例化一个迭代器对象，而迭代器中的`__iter__`用于返回迭代器本身，与`__next__`共同完成迭代器的迭代作用。

```python
class MyFib:
    def __init__(self):
        self.pre = 0
        self.curr = 1
        self.lists = [ii for ii in range(10)]
        return

    def __getitem__(self, item):
        return self.lists[item]


if __name__ == '__main__':
    fib = MyFib()
    fic = iter(fib)#MyFib类实现了__getitem__方法，调用内置函数iter，python会为fib创建一个迭代器对象，并返回之。因为fic是迭代器对象，才可通过for循环迭代。
    # for i in fib: #此行会报错
    for i in fic:
        print(i)
```

迭代器：

```python
class MyFib:
    def __init__(self):
        self.pre = 0
        self.curr = 1
        return

    def __iter__(self):
        return self

    def __next__(self):
        self.curr, self.pre = self.curr + self.pre, self.curr
        return self.curr


if __name__ == '__main__':
    fib = MyFib()
    for i in range(10):
        print(next(fib))
```

**生成器和yield**

生成器其实是一种特殊的迭代器，但是不需要像迭代器一样实现`__iter__`和`__next__`方法，只需要使用关键字yield就可以。

我们来实现一个同样的斐波那契数列，但这次使用的是生成器：

```python
def fib():
    prev, curr = 0, 1
    while True:
        yield curr
        curr, prev = prev + curr, curr

f = fib()
for i in range(5):
    print(next(f))
```

输出打印：

```python
1
1
2
3
5
```

上面的 fib 函数中没有 return 关键字。当运行 f = fib() 的时候，它返回的是一个生成器对象。在调用 fib() 的时候并不会运行 fib 函数中的代码，只有在调用 next() 的时候才会真正运行其中的代码。使用生成器，函数不用一次性生成所有的元素，只需在每次调用next的时候生成元素，这样更节省内存和CPU。

再看一个生成器函数的例子：

```python
def gen_123():  # 只要Python代码中包含yield，该函数就是生成器函数
    yield 1    #生成器函数的定义体中通常都有循环，不过这不是必要条件；此处重复使用了3次yield
    yield 2
    yield 3

if __name__ == '__main__':
    print(gen_123)    # 可以看出gen_123是函数对象
    # <function gen_123 at 0x10be19>
    print(gen_123())  # 函数调用时返回的是一个生成器对象
    # <generator object gen_123 at 0x10be31>

    for i in gen_123(): # 生成器是迭代器，会生成传给yield关键字的表达式的值
        print(i)    
        # 1
        # 2
        # 3

    g = gen_123() # 为了仔细检查，把生成器对象赋值给g
    print(next(g))  # 1
    print(next(g))  # 2
    print(next(g))  # 3
    print(next(g))   # 生成器函数的定义体执行完毕后，生成器对象会抛出异常。
# Traceback (most recent call last):
#   File "test.py", line 17, in <module>
#     print(next(g))
# StopIteration
```



