# **一、Python的数据类型**

1、对于自定义的序列类型类，需要实现两个特殊方法即`__len__(self)`和`__getitem__(self,position)`。例如：
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
2、实现了`__getitem__()`方法可以针对实例对象进行切片操作，即：
```
print(deck[1:3]) -> __getitem__(silce(1,3,1))
```

3、使用“in”操作会调用`__contains__(self,item)`。调用此方法以实现成员检测运算符。如果 item 是 self 的成员则应返回真，否则返回假。对于映射类型，此检测应基于映射的键而不是值或者键值对。对于未定义`__contains___()`的对象，成员检测将首先尝试通过`__iter__()`进行迭代，然后再使用`__getitem__()`的旧式序列迭代协议。由于序列类型是可迭代的，因此可以使用“in”操作，即：
```
for card1 in deck
```

4、`__repr__`和`__str__`的区别在于，后者是在 str() 函数被使用，或是在用 print 函数打印一个对象的时候才被调用的，并且它返回的字符串对终端用户更友好。  
如果你只想实现这两个特殊方法中的一个，`__repr__` 是更好的选择，因为如果一个对象没有 `__str__`函数，而 Python 又需要调用它的时候，解释器会用 `__repr__` 作为替代。


# **二、Python的数据结构**

1、python自带的序列类型（可变[Mutable]与不可变），如下图所示：
![[Pasted image 20240726223758.png]]

2、对序列类型进行切片进行多维切片操作可以使用`...`来进行，即：
```
x[i,...] == x[i,:, :, :] #x为4维数组
```

3、序列的增量赋值运算符`+=`背后的