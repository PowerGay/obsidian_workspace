# **一、Python的数据类型**

1、对于自定义的序列类型类，需要实现两个特殊方法即__len__(self)和__getitem__(self,position)。例如：
```
 class FrenchDeck:
     ranks = [str(n) for n in range(2, 11)] + list('JQKA')
     suits = 'spades diamonds clubs hearts'.split()
     def __init__(self):
         self._cards = [Card(rank, suit) for suit in self.suits
         for rank in self.ranks]
     def __len__(self):
         return len(self._cards)
     def __getitem__(self, position):
         return self._cards[position]
deck=FrenchDeck()
len(deck)
print(deck[0])
```
2、实现了__getitem__()方法可以针对实例对象进行切片操作，即：
```
print(deck[1:3])
```

3、使用“in”操作会调用__contains__(self,item)。调用此方法以实现成员检测运算符。如果 item 是 self 的成员则应返回真，否则返回假。对于映射类型，此检测应基于映射的键而不是值或者键值对。对于未定义__contains___()的对象，成员检测将首先尝试通过__iter__()进行迭代，然后再使用__getitem__()的旧式序列迭代协议。由于序列类型是可迭代的，因此可以使用“in”操作，即：
```
for card1 in deck
```

