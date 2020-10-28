---
title: 流畅的Python学习笔记-01数据模型-1-5
date: 2020-10-28 13:39:27
categories:
- [流畅的Python, 第一章 Python的数据模型]
tags: 
- 学习笔记
---

第一章第五节：本章小结以及延申阅读

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201028144038.png)

<!-- more -->

### 本章小结

通过实现特殊方法，自定义数据类型可以表现得跟内置类型一样，从而让我们写出更具表达力的代码——或者说，更具 Python 风格的代码。

Python 对象的一个基本要求就是它得有合理的字符串表示形式，我们可以通过 `__repr__` 和 `__str__` 来满足这个要求。前者方便我们调试和记录日志，后者则是给终端用户看的。这就是数据模型中存在特殊方法 `__repr__` 和 `__str__` 的原因。

对序列数据类型的模拟是特殊方法用得最多的地方，这一点在FrenchDeck 类的示例中有所展现。在第 2 章中，我们会着重介绍序列数据类型，然后在第 10 章中，我们会把 Vector 类扩展成一个多维的数据类型，通过这个练习你将有机会实现自定义的序列。

Python 通过**运算符重载**这一模式提供了丰富的数值类型，除了内置的那些之外，还有 `decimal.Decimal` 和 `fractions.Fraction`。这些数据类型都支持中缀算术运算符。在第 13 章中，我们还会通过对 Vector类的扩展来学习如何实现这些运算符，当然还会提到如何让运算符满足交换律和增强赋值。

Python 数据模型的特殊方法还有很多，本书会涵盖其中的绝大部分，探讨如何使用和实现它们。

### 延申阅读

对本章内容和本书主题来说，Python 语言参考手册里的“Data Model”一章（[https://docs.python.org/3/reference/datamodel.html](https://docs.python.org/3/reference/datamodel.html)）是最符合规范的知识来源。

Alex Martelli 的《Python 技术手册（第 2 版）》对数据模型的讲解很精彩。我写这本书的时候，《Python 技术手册》的最新版本是 2006 年出版的，书里用的还是 Python 2.5，但是 Python 关于数据模型的概念并没有太大的变化，而书中 Martelli 对属性访问机制的描述，应该是除了CPython 中的 C 源码之外在这方面最权威的解释。Martelli 还是 StackOverflow 上的高产贡献者，在他名下差不多有 5000 条答案，你也可以去他的 StackOverflow 主页（[http://stackoverflow.com/users/95810/alex-martelli](http://stackoverflow.com/users/95810/alex-martelli)）上看看。

<center>杂谈</center>

数据模型还是对象模型：

Python 文档里总是用“Python 数据模型”这种说法，而大多数作者提到这个概念的时候会说“Python 对象模型”。Alex Martelli 的《Python技术手册（第 2 版）》和 David Beazley 的《Python 参考手册（第 4版）》是这个领域中最好的两本书，但是他们也总说“Python 对象模型”。维基百科中对象模型的第一个定义（[http://en.wikipedia.org/wiki/Object_model](http://en.wikipedia.org/wiki/Object_model)）是：计算机编程语言中对象的属性。这正好是“Python 数据模型”所要描述的概念。我在本书中一直都会用“数据模型”这个词，首先是因为在 Python 文档里对这个词有偏爱，另外一个原因是 Python 语言参考手册中与这里讨论的内容最相关的一章的标题就是“数据模型”（[https://docs.python.org/3/reference/datamodel.html](https://docs.python.org/3/reference/datamodel.html)）。

魔术方法：

在 Ruby 中也有类似“特殊方法”的概念，但是 Ruby 社区称之为“魔术方法”，而实际上 Python 社区里也有不少人用的是后者。而我恰恰认为“特殊方法”是“魔术方法”的对立面。Python 和 Ruby 都利用了这个概念来提供丰富的元对象协议，这不是魔术，而是让语言的用户和核心开发者拥有并使用同样的工具。考虑一下 JavaScript，情况就正好反过来了。JavaScript 中的对象有不透明的魔术般的特性，而你无法在自定义的对象中模拟这些行为。比如在 JavaScript 1.8.5 中，用户的自定义对象不能有只读属性，然而不少 JavaScript 的内置对象却可以有。因此在 JavaScript中，只读属性是“魔术”般的存在，对于普通的 JavaScript 用户而言，它就像超能力一样。2009 年推出的 ECMAScript 5.1 才让用户可以定义只读属性。JavaScript 中跟元对象协议有关的部分一直在进化，但由于历史原因，这方面它还是赶不上 Python 和 Ruby。

元对象：

`The Art of the Metaobject Protocal （AMOP）`是我最喜欢的计算机图书的标题。客观来说，元对象协议这个词对我们学习Python 数据模型是有帮助的。元对象所指的是那些对建构语言本身来讲很重要的对象，以此为前提，协议也可以看作接口。也就是说，元对象协议是对象模型的同义词，它们的意思都是构建核心语言的 API。一套丰富的元对象协议能让我们对语言进行扩展，让它支持新的编程范式。AMOP 的第一作者 Gregor Kiczales 后来成为面向方面编程的先驱，他写出了一个 Java 扩展叫 AspectJ，用来实现他对面向方面编程的理念。其实在 Python 这样的动态语言里，更容易实现面向方面编程。现在已经有几个 Python 框架在做这件事情了，其中最重要的是 `zope.interface`（[http://docs.zope.org/zope.interface/](http://docs.zope.org/zope.interface/)）。第11 章的延伸阅读里会谈到它。