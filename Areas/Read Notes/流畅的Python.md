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

3、序列的增量赋值运算符`+=`背后的特色方法就是`__iadd__`，从而实现就地加法；否则，加法操作会生成一个新的对象。

4、`list_sort()`会就地排序list，而`new_list = sorted(old_list)`会新建一个list对象。

5、`bisect`模块包含两个主要函数，`bisect` 和 `insort`，两个函数都利用二分查找算法来在有序序列中查找或插入元素。例子：在有序序列中用`bisect`查找某个元素的插入位置。
```python
 import bisect
 import sys
 HAYSTACK = [1, 4, 5, 6, 8, 12, 15, 20, 21, 23, 23, 26, 29, 30]
 NEEDLES = [0, 1, 2, 5, 8, 10, 22, 23, 29, 30, 31]

 def demo(bisect_fn):
     for needle in reversed(NEEDLES):
         position = bisect_fn(HAYSTACK, needle)
         print(position)
 if __name__ == '__main__':
     if sys.argv[-1] == 'left': 
         bisect_fn = bisect.bisect_left 
     else:
         bisect_fn = bisect.bisect # 新元素会被放置于它相等的元素的后面
     demo(bisect_fn)
```
`insort(seq, item)`把变量 item 插入到序列 seq 中，并能保持 seq 的升序顺序：
```python
 import bisect
 import random
 SIZE=7
 random.seed(1729)
 my_list = []
 for i in range(SIZE):
     new_item = random.randrange(SIZE*2)
     bisect.insort(my_list, new_item)
     print('%2d ->' % new_item, my_list)
```
`insort`的源码很简单，底层就是`bisect_right`或`bisect_left`得到索引后用a.insert来插入：
```python
 def insort_right(a, x, lo=0, hi=None):
     lo = bisect_right(a, x, lo, hi)
     a.insert(lo, x)
 ​
 def insort_left(a, x, lo=0, hi=None):
     lo = bisect_left(a, x, lo, hi)
     a.insert(lo, x)
 insort = insort_right
```
