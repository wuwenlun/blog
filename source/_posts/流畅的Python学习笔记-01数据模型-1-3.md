---
title: 流畅的Python学习笔记-01数据模型-1-3
date: 2020-10-28 10:30:27
categories:
- [流畅的Python, 第一章 Python的数据模型]
tags: 
- 学习笔记
---

第一章第三节：特殊方法一览

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201028144038.png)

<!-- more -->

### 特殊方法一览

`Python`语言参考手册中的“`DataModel`”（[https://docs.python.org/3/reference/datamodel.html](https://docs.python.org/3/reference/datamodel.html)）一章列出了83个特殊方法的名字，其中47个用于实现算术运算、位运算和比较操作。

跟运算符无关的特殊方法：

| 类别                     | 方法名                                                       |
| ------------------------ | ------------------------------------------------------------ |
| 字符串、字节序列表示形式 | `__repr__`、`__str__`、`__format__`、`__bytes__`             |
| 数值转换                 | `__abs__`、`__bool__`、`__complex__`、`__int__`、`__float__`、`__hash__`、`__index__` |
| 集合模拟                 | `__len__`、`__getitem__`、`__setitem__`、`__delitem__`、`__contains__` |
| 迭代枚举                 | `__iter__`、`__reversed__`、`__next__`                       |
| 可调用模拟               | `__call__`                                                   |
| 上下文管理               | `__enter__`、`__exit__`                                      |
| 实例创建和销毁           | `__new__`、`__init__`、`__del__`                             |
| 属性管理                 | `__getattr__`、`__getattribute__`、`__setattr__`、`__delattr__`、`__dir__` |
| 属性描述符               | `__get__`、`__set__`、`__delete__`                           |
| 跟类相关的服务           | `__prepare__`、`__instancecheck__`、`__subclasscheck__`      |

<!-- more -->

跟运算符相关的特殊方法

| 类别               | 方法名和对应的运算符                                         |
| ------------------ | ------------------------------------------------------------ |
| 一元运算符         | `__neg__`对应`-`、`__pos__`对应`+`、`__abs__`对应`abs()`     |
| 众多比较运算符     | `__lt__`对应`<`、`__le__`对应`<=`、`__eq__`对应`==`、`__ne__`对应`!=`、`__gt__`对应`>`、`__ge__`对应`>=` |
| 算术运算符         | `__add__`对应`+`、`__sub__`对应`-`、`__mul__`对应`*`、`__truediv__`对应`/`、`__floordiv__`对应`//`、`__mod__`对应`%`、`__divmod__`对应`divmod()`、`__pow__`对应`**`或`pow()`、`__round__`对应`round()` |
| 反向算术运算符     | `__radd__`、`__rsub__`、`__rmul__`、`__rtruediv__`、`__rfloordiv__`、`__rmod__`、`__rdivmod__` |
| 增量赋值算术运算符 | `__iadd__`、`__isub__`、`__imul__`、`__itruediv__`、`__ifloordiv__`、`__imod__`、`__ipow__` |
| 位运算符           | `__invert__`对应`~`、`__lshift__`对应`<<`、`__rshift__`对应`>>`、`__and__`对应`&`、`__or__`对应`^` |
| 反向位运算符       | `__rlshift__`、`__rrshift__`、`__rand__`、`__rxor__`、`__ror__` |
| 增量赋值位运算符   | `__ilshift__`、`__irshift__`、`__iand__`、`__ixor__`、`__ior__` |

当交换两个操作数的位置时，就会调用反向运算符（`b * a`而不是`a * b`）。增量赋值运算符则是一种把中缀运算符变成赋值运算的捷径（`a = a * b`就变成了`a *= b`）。第13章会对这两者作出详细解释。