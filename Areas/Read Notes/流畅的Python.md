# **一、Python的数据类型**

1、对于自定义的序列类型类，需要实现两个特殊方法：__len__(self)和__getitem__(self,position)。例如：
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
deck[]
```
