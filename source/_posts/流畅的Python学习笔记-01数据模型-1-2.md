---
title: 流畅的Python学习笔记-01数据模型-1-2
date: 2020-10-22 09:41:27
categories:
- [流畅的Python, 第一章 Python的数据模型]
tags: 
- 学习笔记
---

### 1.2 如何使用特殊方法

首先明确一点，特殊方法的存在是为了被`Python`解释器调用的，你自己并不需要调用它们。也就是说没有`my_object.__len__()`这种写法，而应该使用`len(my_object)`。在执行`len(my_object)`的时候，如果`my_object`是一个自定义类的对象，那么`Python`会自己去调用其中由你实现的` __len__ `方法。

然而如果是`Python`内置的类型，比如列表（`list`）、字符串（`str`）、字节序列（`bytearray`）等，那么`CPython`会抄个近路，`__len__` 实际上会直接返回`PyVarObject`里的`ob_size`属性。`PyVarObject`是表示内存中长度可变的内置对象的`C `语言结构体。直接读取这个值比调一个方法要快很多。

<!-- more -->

很多时候，特殊方法的调用是隐式的，比如`for i in x:`这个语句，背后其实用的是`iter(x)`，而这个函数的背后则是`x.__iter__()` 方法。当然前提是这个方法在`x`中被实现了。

通常你的代码无需直接使用特殊方法，除非有大量的元编程存在。唯一的例外可能是`__init__ `方法，你的代码里可能经常会用到它，目的是在你自己的子类的`__init__ `方法中调用超类的构造器。

通过内置的函数（例如`len`、`iter`、`str`，等等）来使用特殊方法是最好的选择。这些内置函数不仅会调用特殊方法，通常还提供额外的好处，而且对于内置的类来说，它们的速度更快。`14.12` 节中有详细的例子。不要自己想当然地随意添加特殊方法，比如`__foo__`之类的，因为现在这个名字没有被`Python`内部使用。

#### 1.2.1 模拟数值类型

利用特殊方法，可以让自定义对象通过加号`“+”`（或是别的运算符）进行运算。第`13`章对此有详细的介绍，现在只是借用这个例子来展示特殊方法的使用。

我们来实现一个二维向量（`vector`）类，这里的向量就是欧几里得几何中常用的概念，常在数学和物理中使用的那个（见图`1-1`）。

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201027160403.png)

上图是一个二维向量加法的例子，`Vector(2,4) + Vextor(2,1) = Vector(4,5)`

`Vector`二维向量类如下：

```python
from math import hypot


class Vector:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return 'Vector(%r, %r)' % (self.x, self.y)

    def __abs__(self):
        return hypot(self.x, self.y)

    def __bool__(self):
        return bool(abs(self))

    def __add__(self, other):
        x = self.x + other.x
        y = self.y + other.y
        return Vector(x, y)

    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)

```

对这个类做向量的加法操作：

```python
v1 = Vector(2, 4)
v2 = Vector(2, 1)
print(v1 + v2)

>>>Vector(4, 5)
```

注意其中的**`+`**运算符所得到的结果也是一个向量，而且结果能被控制台友好地打印出来。这个**`+`**操作符实际上是去调用`Vector`类里的`__add__`方法了。所以这也是`Python`强大的一个地方。

`abs`是一个内置函数，如果输入是整数或者浮点数，它返回的是输入值的绝对值；如果输入是复数（`complex number`），那么返回这个复数的模。为了保持一致性，我们的`API`在碰到`abs`函数的时候，也应该返回该向量的模，`abs`函数会去调用`Vector`类的`__abs__()`方法：

```python
v1 = Vector(3, 4)
print(v1)

>>>5.0
```

我们还可以利用**`*`**运算符来实现向量的标量乘法，该运算符实际上是去调用`Vector`类的`__mul__`方法。

```python
v1 = Vector(3, 4)
print(v1 * 3)

>>>Vector(9, 12)
```

虽然`Vector`类代码里有`6`个特殊方法，但这些方法（除了`__init__`）并不会在这个类自身的代码中使用。即便其他程序要使用这个类的这些方法，也不会直接调用它们，就像我们在上面的控制台对话中看到的。上文也提到过，一般只有`Python`的解释器会频繁地直接调用这些方法。接下来看看每个特殊方法的实现。

#### 1.2.2 字符串表示形式

`Python`有一个内置的函数叫`repr`，它能把一个对象用字符串的形式表达出来以便辨认。`repr`函数命令就是通过`__repr__`这个特殊方法来得到一个对象的字符串表示形式的。如果没有实现`__repr__`，当我们在控制台里打印一个向量的实例时，得到的字符串可能会是 `<Vector object at 0x10e100070>`这种抽象表示。

在`__repr__`的实现中，我们用到了`%r`来获取对象各个属性的标准字符串表示形式——这是个好习惯。`__repr__ `所返回的字符串应该准确、无歧义，并且尽可能表达出如何用代码创建出这个被打印的对象。

`__repr__ `和`__str__`的区别在于，后者是在`str()`函数被使用，或是在用`print`函数打印一个对象的时候才被调用的，并且它返回的字符串对终端用户更友好。如果你只想实现这两个特殊方法中的一个，`__repr__`是更好的选择，<font color=red>因为如果一个对象没有`__str__`函数，而`Python`又需要调用它的时候，解释器会用`__repr__`作为替代。</font>

下面来看一个例子，`Vector`类在原来的基础上新增一个方法`__str__`

```python
class Vector:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return 'i am __repr__ mathod, Vector(%r, %r)' % (self.x, self.y)

    def __str__(self):
        return 'i am __str__ mathod, Vector(%r, %r)' % (self.x, self.y)

    ......

if __name__ == '__main__':
    v1 = Vector(3, 4)
    print('%s' % v1) # 注意前面是%s，代表着的意思是它会去执行Vector类里的__str__方法
    print('%r' % v1) # 注意前面是%r，代表着的意思是它会去执行Vector类里的__repr__方法
    
>>>i am __str__ mathod, Vector(3, 4)
>>>i am __repr__ mathod, Vector(3, 4)
```

#### 1.2.3 算术运算符

通过`__add__`和`__mul__`，示例为向量类带来了**`+`**和**`*`**这两个算术运算符。值得注意的是，这两个方法的返回值都是新创建的向量对象，被操作的两个向量还是原封不动，代码里只是读取了它们的值而已。**中缀运算符**的基本原则就是不改变操作对象，而是产出一个新的值。第 13 章会谈到更多这方面的问题。

#### 1.2.4 自定义的布尔值

尽管`Python`里有`bool`类型，但实际上任何对象都可以用于需要布尔值的上下文中（比如`if`或`while`语句，或者`and`、`or`和`not`运算符）。为了判定一个值`x`为真还是为假，`Python`会调用`bool(x)`，这个函数只能返回`True`或者`False`。

默认情况下，我们自己定义的类的实例总被认为是真的，除非这个类对`__bool__`或者`__len__`函数有自己的实现。`bool(x) `的背后是调用`x.__bool__()`的结果；如果不存在`__bool__`方法，那么`bool(x)`会尝试调用`x.__len__()`。若返回`0`，则`bool`会返回`False`；否则返回`True`。

我们对`Vector`类中的`__bool__`的实现很简单，如果一个向量的模是`0`，那么就返回`False`，其他情况则返回`True`。因为`__bool__`函数的返回类型应该是布尔型，所以我们通过`bool(abs(self))`把模值变成了布尔值。