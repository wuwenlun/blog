---
title: 流畅的Python学习笔记-01数据模型-1-1
date: 2020-10-22 09:30:27
categories:
- [流畅的Python, 第一章 Python的数据模型]
tags: 
- 学习笔记
---

第一章第一节：一摞Python风格的纸牌

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201028144038.png)

<!-- more -->

### 一摞Python风格的纸牌

下面这段程序是用来构建一副扑克牌，带红星、黑桃、方块、梅花的`23456789JQKA`牌，共`52`张

```python
import collections

Card = collections.namedtuple("Card", ['rank', 'suit'])

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()

    def __init__(self):
        # 左边第一个for循环是外层，第二个for循环是内层，可遍历_cards看数据的顺序得到
        self._cards = [Card(rank, suit) for suit in self.suits for rank in self.ranks]

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, position):
        return self._cards[position]
    
    def __contains__(self, item):
        for card in self._cards:
            if card == item:
                return True
        return False
```

自`Python 2.6` 开始，`namedtuple`就加入到`Python`里，用以构建只有少数属性但是没有方法的对象。

当然，我们这个例子主要还是关注`FrenchDeck`这个类，它既短小又精悍。首先，它跟任何标准`Python`集合类型一样，可以用`len()` 函数来查看一叠牌有多少张。实际上`len()`函数调用的是`FrenchDeck`类的`__len__`方法。

```python
deck = FrenchDeck()
print(len(deck))

>>>52
```

`__getitem()`方法也是非常有用处，可以得到特定的一张纸牌，如：`deck[0]`、`deck[1]`，还支持切片(`slicing`)操作。上述操作其内部逻辑就是去执行`__getitem__()`方法。

```python
deck[:3]

>>>[Card(rank='2', suit='spades'), Card(rank='3', suit='spades'), Card(rank='4', suit='spades')]
```

另外，仅仅实现了`__getitem__`方法，这一摞牌就变成可迭代的了：

```python
deck = FrenchDeck()
for card in deck:
    pass
```

反向迭代也没关系：

```python
for card in reversed(deck):
    pass
```

`in`关键字在集合类型中除了支持遍历，还可以判断一个对象是否在该集合中，如：

```python
print(Card('Q', 'hearts') in deck)

>>>True
```

它工作的原理是判断该集合类是否实现了`__contains__`方法，如果实现了就用`__contains__`方法来判断一个对象是否在集合中，如果没有实现`__contains__`方法，就一个个遍历去调用`__getitem__`方法。

提问：

按照目前的设计，`FrenchDeck` 是不能洗牌的，因为这摞牌是不可变的（`immutable`）：卡牌和它们的位置都是固定的，除非我们破坏
这个类的封装性，直接对`_cards`进行操作。第`11`章会讲到，其实只需要一行代码来实现 `__setitem__` 方法，洗牌功能就不是问题了。