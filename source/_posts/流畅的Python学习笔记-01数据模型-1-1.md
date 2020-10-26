### 1.1 一摞Python风格的纸牌

下面这段程序是用来构建一副扑克牌，带红星、黑桃、方块、梅花的`23456789JQKA`牌，共`52`张

```python
import collections

Card = collections.namedtuple("Card", ['rank', 'suit'])

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()

    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits for rank in self.ranks]

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, position):
        return self._cards[position]
```

自`Python 2.6` 开始，`namedtuple`就加入到`Python`里，用以构建只有少数属性但是没有方法的对象。

当然，我们这个例子主要还是关注`FrenchDeck`这个类，它既短小又精悍。首先，它跟任何标准`Python`集合类型一样，可以用`len()` 函数来查看一叠牌有多少张。实际上`len()`函数调用的是`FrenchDeck`类的`__len__`方法。

```python
deck = FrenchDeck()
print(len(deck))

>>>52
```



