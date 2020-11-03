---
title: 流畅的Python学习笔记-02序列构成的数组-2-3
date: 2020-10-29 16:43:50
categories:
- [流畅的Python, 第二章 序列构成的数组]
tags: 
- 学习笔记
---

第二章第三节：元组不仅仅是不可变的列表

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201028144038.png)

<!-- more -->

有些 Python 入门教程把元组称为“不可变列表”，然而这并没有完全概括元组的特点。除了用作不可变的列表，它还可以用于没有字段名的记录。鉴于后者常常被忽略，我们先来看看元组作为记录的功能。

### 元组和记录

元组其实是对数据的记录：元组中的每个元素都存放了记录中一个字段的数据，外加这个字段的位置。正是这个位置信息给数据赋予了意义。

如果只把元组理解为不可变的列表，那其他信息——它所含有的元素的总数和它们的位置——似乎就变得可有可无。但是如果把元组当作一些字段的集合，那么数量和位置信息就变得非常重要了。

示例 2-7 中的元组就被当作记录。如果在任何的表达式里我们在元组内对元素排序，这些元素所携带的信息就会丢失，因为这些信息是跟它们的位置有关的。

示例 2-7 把元组用作记录：

```python
>>> lax_coordinates = (33.9425, -118.408056) ➊
>>> city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014) ➋
>>> traveler_ids = [('USA', '31195855'), ('BRA', 'CE342567'), ➌
... ('ESP', 'XDA205856')]
>>> for passport in sorted(traveler_ids): ➍
... print('%s/%s' % passport) ➎
...
BRA/CE342567
ESP/XDA205856
USA/31195855
>>> for country, _ in traveler_ids: ➏
... print(country)
...
USA
BRA
ESP
```

❶ 洛杉矶国际机场的经纬度。

❷ 东京市的一些信息：市名、年份、人口（单位：百万）、人口变化（单位：百分比）和面积（单位：平方千米）。

❸ 一个元组列表，元组的形式为 (country_code, passport_number)。

❹ 在迭代的过程中，passport 变量被绑定到每个元组上。

❺ % 格式运算符能被匹配到对应的元组元素上。

❻ for 循环可以分别提取元组里的元素，也叫作拆包（unpacking）。因为元组中第二个元素对我们没有什么用，所以它赋值给“_”占位符。

拆包让元组可以完美地被当作记录来使用，这也是下一节的话题。

### 元组拆包

示例 2-7 中，我们把元组 ('Tokyo', 2003, 32450, 0.66, 8014) 里的元素分别赋值给变量 city、year、pop、chg 和 area，而这所有的赋值我们只用一行声明就写完了。同样，在后面一行中，一个 % 运算符就把 passport 元组里的元素对应到了 print 函数的格式字符串空档中。这两个都是对元组拆包的应用。

**元组拆包可以应用到任何可迭代对象上**，唯一的硬性要求是，被可迭代对象中的元素数量必须要跟接受这些元素的元组的空档数一致。除非我们用 `*` 来表示忽略多余的元素，在用 `*` 来处理多余的元素一节里，我会讲到它的具体用法。Python 爱好者们很喜欢用元组拆包这个说法，但是可迭代元素拆包这个表达也慢慢流行了起来，比如“PEP 3132—Extended IterableUnpacking”（[https://www.python.org/dev/peps/pep-3132/](https://www.python.org/dev/peps/pep-3132/)）的标题就是这么用的。

上面提到<font color=red>**元组拆包可以应用到任何可迭代对象上**</font>，这句话要怎么理解呢？看下面一段代码：

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
    fic = iter(fib)# 因为类有__getitem__方法，即使它没有__iter__方法，也可以经过iter内置函数获得MyFib的迭代器对象
    a, *b, c = fic# 科迭代对象经过拆包，把其内部属性lists分别拆包赋值给a、b、c变量
    print(a)
    print(b)
    print(c)
    
>>>0
>>>[1, 2, 3, 4, 5, 6, 7, 8]
>>>9

```

在 Python 开发中经常可以看到变量交换的代码，非常优雅，不使用中间变量交换两个变量的值，如下的方式：

```python
>>> b, a = a, b
```

其实 `=` 号右边就是一个元组，然后拆包赋值给 b 和 a 两个变量。怎么看 `=` 号右边是一个元组呢？通过下面代码来论证：

```python
if __name__ == '__main__':
    a = 5
    b = 10
    c = a, b
    print(c)
    
>>>(5, 10)# 变量c打印出来就是一个元组    
```

可以用 `*` 运算符把一个可迭代对象拆开作为函数的参数：

```python
>>> divmod(20, 8)
(2, 4)
>>> t = (20, 8)
>>> divmod(*t)
(2, 4)
>>> quotient, remainder = divmod(*t)
>>> quotient, remainder
(2, 4)
```

下面是另一个例子，这里元组拆包的用法则是让一个函数可以用元组的形式返回多个值，然后调用函数的代码就能轻松地接受这些返回值。比如 os.path.split() 函数就会返回以路径和最后一个文件名组成的元组 (path, last_part):

```python
import os

if __name__ == '__main__':
    path, filename = os.path.split('/home/luciano/.ssh/idrsa.pub')
    print(path)
    print(filename)

>>>/home/luciano/.ssh
>>>idrsa.pub
```

在进行拆包的时候，我们不总是对元组里所有的数据都感兴趣，`_` 占位符能帮助处理这种情况，上面这段代码也展示了它的用法。

除此之外，在元组拆包中使用 `*` 也可以帮助我们把注意力集中在元组的部分元素上。

用 `*` 来处理剩下的元素。

在 Python 中，函数用 `*args` 来获取不确定数量的参数算是一种经典写法了。

于是 Python 3 里，这个概念被扩展到了平行赋值中：

```python
>>> a, b, *rest = range(5)
>>> a, b, rest
(0, 1, [2, 3, 4])
>>> a, b, *rest = range(3)
>>> a, b, rest
(0, 1, [2])
>>> a, b, *rest = range(2)
>>> a, b, rest
(0, 1, [])
```

在平行赋值中，`*` 前缀只能用在一个变量名前面，但是这个变量可以出现在赋值表达式的任意位置：

```python
>>> a, *body, c, d = range(5)
>>> a, body, c, d
(0, [1, 2], 3, 4)
>>> *head, b, c, d = range(5)
>>> head, b, c, d
([0, 1], 2, 3, 4)
```

另外元组拆包还有个强大的功能，那就是可以应用在嵌套结构中。

### 嵌套元组拆包

接受表达式的元组可以是嵌套式的，例如 (a, b, (c, d))。只要这个接受元组的嵌套结构符合表达式本身的嵌套结构，Python 就可以作出正确的对应。示例 2-8 就是对嵌套元组拆包的应用。

示例 2-8 用嵌套元组来获取经度

```python
metro_areas = [
('Tokyo','JP',36.933,(35.689722,139.691667)), # ➊
('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
]
print('{:15} | {:^9} | {:^9}'.format('', 'lat.', 'long.')) # "^9" 代表占9个空格，内容居中对齐
fmt = '{:15} | {:9.4f} | {:9.4f}'# {:15}意思是占位15个空格，{:9.4f} 意思是占位9位空格，4位小数部分
for name, cc, pop, (latitude, longitude) in metro_areas: # ➋
if longitude <= 0: # ➌
print(fmt.format(name, latitude, longitude))
```

❶ 每个元组内有 4 个元素，其中最后一个元素是一对坐标。

❷ 我们把输入元组的最后一个元素拆包到由变量构成的元组里，这样
就获取了坐标。

❸ `if longitude <= 0:` 这个条件判断把输出限制在西半球的城市。

示例 2-8 的输出是这样的：

```python
               |       lat. | long.
Mexico City    |    19.4333 | -99.1333
New York-Newark|    40.8086 | -74.0204
Sao Paul       |   -23.5478 | -46.63
```

在 Python 3 之前，元组可以作为形参放在函数声明中，例如def fn(a, (b, c), d):。然而 Python 3 不再支持这种格式，具体原因见于“PEP 3113—Removal of Tuple ParameterUnpacking”（[http://python.org/dev/peps/pep-3113/](http://python.org/dev/peps/pep-3113/)）。需要弄清楚的是，这个改变对函数调用者并没有影响，它改变的是某些函数的声明方式。

元组已经设计得很好用了，但作为记录来用的话，还是少了一个功能：我们时常会需要给记录中的字段命名。namedtuple 函数的出现帮我们解决了这个问题。

### 具名元组

collections.namedtuple 是一个工厂函数，它可以用来构建一个带字段名的元组和一个有名字的类——这个带名字的类对调试程序有很大帮助。

用 namedtuple 构建的类的实例所消耗的内存跟元组是一样的，因为字段名都被存在对应的类里面。这个实例跟普通的对象实例比起来也要小一些，因为 Python 不会用 `__dict__` 来存放这些实例的属性。

在第 1 章中，展示过这样一段代码：

```python
Card = collections.namedtuple('Card', ['rank', 'suit'])
```

示例 2-9 展示了如何用具名元组来记录一个城市的信息。

```python
>>> from collections import namedtuple
>>> City = namedtuple('City', 'name country population coordinates') ➊
>>> tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667)) ➋
>>> tokyo
City(name='Tokyo', country='JP', population=36.933, coordinates=(35.689722,
139.691667))
>>> tokyo.population ➌
36.933
>>> tokyo.coordinates
(35.689722, 139.691667)
>>> tokyo[1]
'JP
```

❶ 创建一个具名元组需要两个参数，一个是类名，另一个是类的各个字段的名字。后者可以是由数个字符串组成的可迭代对象，或者是由空格分隔开的字段名组成的字符串。

❷ 存放在对应字段里的数据要以一串参数的形式传入到构造函数中（注意，元组的构造函数却只接受单一的可迭代对象）。

❸ 你可以通过字段名或者位置来获取一个字段的信息。

除了从普通元组那里继承来的属性之外，具名元组还有一些自己专有的属性。示例 2-10 中就展示了几个最有用的：_fields 类属性、类方法_make(iterable) 和实例方法 _asdict()。

示例 2-10 具名元组的属性和方法（接续前一个示例）

```python
>>> City._fields ➊
('name', 'country', 'population', 'coordinates')
>>> LatLong = namedtuple('LatLong', 'lat long')
>>> delhi_data = ('Delhi NCR', 'IN', 21.935, LatLong(28.613889, 77.208889))
>>> delhi = City._make(delhi_data) ➋
>>> delhi._asdict() ➌
OrderedDict([('name', 'Delhi NCR'), ('country', 'IN'), ('population',
21.935), ('coordinates', LatLong(lat=28.613889, long=77.208889))])
>>> for key, value in delhi._asdict().items():
print(key + ':', value)
name: Delhi NCR
country: IN
population: 21.935
coordinates: LatLong(lat=28.613889, long=77.208889)
>>>

```

❶ _fields 属性是一个包含这个类所有字段名称的元组。

❷ 用 _make() 通过接受一个可迭代对象来生成这个类的一个实例，它的作用跟 City(*delhi_data) 是一样的。

❸ _asdict() 把具名元组以 collections.OrderedDict 的形式返回，我们可以利用它来把元组里的信息友好地呈现出来。

现在我们知道了，元组是一种很强大的可以当作记录来用的数据类型。它的第二个角色则是充当一个不可变的列表。下面就来看看它的第二重功能。

### 作为不可变列表的元组

如果要把元组当作列表来用的话，最好先了解一下它们的相似度如何。在下表中可以清楚地看到，除了跟增减元素相关的方法之外，元组支持列表的其他所有方法。还有一个例外，元组没有 `__reversed__` 方法，但是这个方法只是个优化而已，reversed(my_tuple) 这个用法在没有 `__reversed__` 的情况下也是合法的。

列表或元组的方法和属性（那些由object类支持的方法没有列出来）：

|                           | 列表 | 元组 |                   |
| ------------------------- | ---- | ---- | ----------------- |
| `s.__add__(s2)`           | *    | *    | s + s2，拼接      |
| `s.__iadd__(s2)`          | *    |      | s += s2，就地拼接 |
| `s.append(e)`             | *    |      |                   |
| `s.clear()`               | *    |      |                   |
| `s.__contains__(e)`       | *    | *    |                   |
| `s.copy()`                | *    |      |                   |
| `s.count(e)`              | *    | *    |                   |
| `s.__delitem__(p)`        | *    |      |                   |
| `s.extend(it)`            | *    |      |                   |
| `s.__getitem__(p)`        | *    | *    |                   |
| `s.__getnewargs__()`      | *    | *    |                   |
| `s.index(e)`              | *    | *    |                   |
| `s.insert(p, e)`          | *    |      |                   |
| `s.__iter__()`            | *    | *    |                   |
| `s.__len__()`             | *    | *    |                   |
| `s.__mul__(n)`            | *    | *    |                   |
| `s.__imul__(n)`           | *    |      |                   |
| `s.__rmul__(n)`           | *    | *    |                   |
| `s.pop([p])`              | *    |      |                   |
| `s.remove(e)`             | *    |      |                   |
| `s.reverse()`             | *    |      |                   |
| `s.__reversed__()`        | *    |      |                   |
| `s.__setitem__(p,e)`      | *    |      |                   |
| `s.sort([key],[reverse])` | *    |      |                   |


每个 Python 程序员都知道序列可以用 s[a:b] 的形式切片，但是关于切片，我还想说说它的一些不太为人所知的方面。