# **一、Python的数据类型**

1、对于自定义的序列类型类，需要实现两个特殊方法即`__len__(self)`和`__getitem__(self,position)`。例如：
```python
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

6、如果我们需要一个只包含数字的列表，那么 `array.array` 比 list 更高效数组支持所有跟可变序列有关的操作，包括 `.pop`、`.insert` 和` .extend`。另外，数组还提供从文件读取和存 入文件的更快的方法，如 `.frombytes `和 `.tofile`。
```python
from array import array
 from random import random
 floats1 = array('d', (random() for i in range(10**7))) # 创建double数组
​
 with open('floats.bin', 'wb') as fp1: # 把数组存入二进制文件
     floats1.tofile(fp1)
 ​
 floats2 = array('d') 
 with open('floats.bin', 'rb') as fp2:
     floats2.fromfile(fp2, 10**7) # 读取
 print(floats1 == floats2) # 比较读取和存储的是否一致
```
重点：
	1) 用` array.fromfile` 从一个二进制文件里读出1000万个双精度浮点数只需要0.1秒，这比从文本文件里读取的速度要快60倍。
	2) 使用 `array.tofile` 写入到二进制文件，比以每行一个浮点数的方式把所有数字写入到文本文件要快 7倍。
	3) 1000 万个这样的数在二进制文件里只占用 80 000 000 个字节（每个浮点数占8 个字节，不需要任何额外空间），如果是文本文件的话，我们需要 181 515 739 个字节(**20多倍**)。

另外一个快速序列化数字类型的方法是使用` pickle`模块。`pickle.dump` 处理浮点数组的速度几乎跟` array.tofile` 一样快。

7、python的队列模块：

| 队列                | 描述                                                                                                                                                                                                  |
| :---------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| collections.deque | 双向队列是一个线程安全、可以快速从两端添加或者删除元素的数据类型。                                                                                                                                                                   |
| queue             | 提供了同步（线程安全）类 Queue 和 PriorityQueue，不同的线程可以利用 这些数据类型来交换信息。这三个类的构造方法都有一个可选参数 maxsize，它接收正 整数作为输入值，用来限定队列的大小。但是在满员的时候，这些类不会扔掉旧的元素 来腾出位置。相反，如果队列满了，它就会被锁住，直到另外的线程移除了某个元素而 腾出了位置。这一特性让这些类很适合用来控制活跃线程的数量。 |
| multiprocessing   | 这个包实现了自己的Queue，它跟queue.Queue 类似，是设计给进程间通信用的。                                                                                                                                                        |
| asyncio           | 里面有 QueuePriorityQueue 。这些类受到 queue 和 multiprocessing 模块的影响，但是为异步编程里的任务管理提供 了专门的便利。                                                                                                                 |
| heapq             | heapq 没有队列类，而是提供了 `heappush` 和` heappop`方法， 让用户可以把可变序列当作堆队列或者优先队列来使用。                                                                                                                               |

## **三、Python的字典与集合

1、模块的命名空间、实例的属性和函数的关键字参数中都可以看到字典的身影  
跟它有关的内置函数都在`__builtins__.__dict__` 模块中。

2、泛映射类型。collections.abc 模块中有 Mapping 和 MutableMapping 这两个抽象基类，它们的作用是为dict 和其他类似的类型定义形式接口。
![[Pasted image 20240727072605.png]]
然而，非抽象映射类型一般不会直接继承这些抽象基类，它们会直接对dict 或是 collections.User.Dict 进行扩展。这些抽象基类的主要作用是作为形式化的文档，它们定义了构建一个映射类型所需要的最基本的接口。然后它们还可以`isinstance` 一起被用来判定某个数据是不是广义上的映射类型。例如：
```python
 >>> from collections import abc
 >>> my_dict = {}
 >>> isinstance(my_dict,abc.Mapping)
 True
 >>> isinstance(my_dict,abc.MutableMapping)
 True
```

标准库里的所有映射类型都是利用 dict 来实现的，因此它们有个共同的限制，即只有可散列 的数据类型才能用作这些映射里的键。

3、可散列的数据类型。如果一个对象是可散列的，那么在这个对象的生命周期中，它的散列值是不变的，而且这个对象需要实现 `__hash__()`方法。另外可散列对象还要有 `__qe__()`方法，这样才能跟其他键做比较。如果两个可散列对象是相等的，那么它们的散列值一定是一样的。其中，基本的不可变数据类型都是可散列的；此外，**用户自定义的类型对象也是可散列的**。

4、字典。3.9开始支持`|`和`|=`来合并字典。如：
```python
 >>> d1 = {'a': 1, 'b': 3}
 >>> d2 = {'a': 2, 'b': 4, 'c': 6}
 >>> d1 | d2
 {'a': 2, 'b': 4, 'c': 6}
 >>> d1
 {'a': 1, 'b': 3}
 >>> d1 |= d2
 >>> d1
 {'a': 2, 'b': 4, 'c': 6}
```

5、模式匹配。match/case在Python3.10中开始支持，case部分可以用mapping来做pattern。可以支持序列匹配。如：
```python
 def get_creators(record: dict) -> list:
     match record:
         case {'type': 'book', 'api': 2, 'authors': [*names]}:
             return names
         case {'type': 'book', 'api': 1, 'author': name}:
             return [name]
         case {'type': 'book'}:
             raise ValueError(f"Invalid 'book' record: {record!r}")
         case {'type': 'movie', 'director': name}:
             return [name]
         case _:
             raise ValueError(f'Invalid record: {record!r}')
 ​
 ​
 b1 = dict(api=1, author='Douglas Hofstadter', type='book', title='Gödel, Escher, Bach')
 result_b1 = get_creators(b1)
 print(result_b1)  # ['Douglas Hofstadter']
 ​
 from collections import OrderedDict
 ​
 b2 = OrderedDict(api=2, type='book',
                  title='Python in a Nutshell',
                  authors='Martelli Ravenscroft Holden'.split())
 result_b2 = get_creators(b2)
 print(result_b2)  # ['Martelli', 'Ravenscroft', 'Holden']
 ​
 # get_creators({'type': 'book', 'pages': 770}) # ValueError: Invalid 'book' record: {'type': 'book', 'pages': 770}
 ​
 # get_creators('Spam, spam, spam') # ValueError: Invalid record: 'Spam, spam, spam'
 
```
其中，list、tuple、dict 等都能作为模式，并且能配合 `*` 或 `**` 和通配符使用：
- `[*_]` 匹配任意长度的 list;
- `(_, _, *_)` 匹配长度至少为 2 的 tuple。
更多操作参考：
[Python 结构化模式匹配 match case | Python 教程 - 盖若 (gairuo.com)](https://gairuo.com/p/python-match-case)

6、`setdefault`与`defaultdict`。
使用`setdefault`可以在只查找一次字典中的健时，就将没有该键的kv对插入字典中。
```python
 my_dict.setdefault(key, []).append(new_value)
```
比使用如下方法要快（涉及两次键查询）：
```python
 if key not in my_dict:
     my_dict[key] = [] #1-2行等同于my_dick.get(key,[])
 my_dict[key].append(new_value)
```
有时候为了方便起见，就算某个键在映射里不存在，我们也希望在通过这个键读取值的时候能得到一个默认值。有两个途径能帮我们达到这个目的，一个是通过 defaultdict 这个类型而不是普通的 dict，另一个是给自己定义一个 dict 的子类，然后在子类中实现`__missing__`方法。

在实例化一个 defaultdict 的时候，需要给构造方法提供一个可调用对象，这个可调用对象会在 `__getitem__ `碰到找不到的键的时候被调用，让` __getitem__ `返回某种默认值。但要注意，只能用[]，用get可不行。例如：
```python
 from collections import defaultdict
 dd = defaultdict(list)
 print(dd.get('a')) # None
 print(dd['a'])     # []
```
这是因为 defaultdict 里的 default_factory 只会在` __getitem__ `里被调用，在其他的方法里完全不会发挥作用。如果有一个类继承了 dict，然后这个继承类提供了 `__missing__`方法，那么在 `__getitem__`碰到找不到的键的时候，Python 就会自动调用它，而不是抛出一个KeyError 异常。 `__missing__` 方法只会被`__getitem__`调用。
```python
 class StrKeyDict0(dict): 
     def __missing__(self, key):
         print('u r calling missing')
         if isinstance(key, str): 
             raise KeyError(key)
         return self[str(key)] 
     def get(self, key, default=None):
         try:
             print('u r calling get')
             return self[key] 
         except KeyError:
             return default
     def __contains__(self, key):
         print('u r calling contains')
         return key in self.keys() or str(key) in self.keys()
```
上述例子中需注意：
	1）`__missing__` 中的`isinstance`+`raise`是必须的，因为`__missing__` 调用了`self[str(key)]`，从而实现递归判断。
	2）`__contains__`是必须实现的，因为当类对象使用'in'操作时，键不存在时是不会走`__contains__`。
	3）`__contains__`并不存在递归的现象，因为`self.keys()`的类型是dict_keys而不是StrKeyDict0。如果改为`key in self`则会递归调用。
	
7、子类化UserDict
就创造自定义映射类型来说，更倾向于从 UserDict 而不是从 dict 继承。主要原因是，后者有时会在某些方法的实现上走一些捷径，导致我们不得不在它的子类中重写这些方法，但是UserDict 就不会带来这些问题 UserDict 并不是 dict 的子类，但是 UserDict 有一个叫作data 的属性，是 dict 的实例，这个属性实际上是 UserDict 最终存储数据。例如：
```python
 import collections
 class StrKeyDict(collections.UserDict):
     def __missing__(self, key): 
         if isinstance(key, str):
             raise KeyError(key)
         return self[str(key)]
     def __contains__(self, key):
         return str(key) in self.data
     def __setitem__(self, key, item):
         self.data[str(key)] = item 
d = StrKeyDict0([('2', 'two'), ('4', 'four')])
```
上述例子中需注意：
	1）在StrKeyDict初始化的时候会存在赋值的操作即`self.data=data`，从而调用`__setitem__` 方法将键转换为字符串的形式；而继承dict，初始化时并不存在赋值操作。

8、集合。
![[Pasted image 20240727094809.png]]
集合和字典查找元素的效率远高于列表，原因是它们二者都是可散列的，使用了空间来换取时间的策略。

9、散列表算法。
![[Pasted image 20240727095257.png]]

10、不要对字典同时进行迭代和修改。如果想扫描并修改一个字典，最好分成两步来进行：首先对字典迭代，以得出需要添加的内容，把这些内容放在一个新字典里；迭代结束之后再对原有字典进行更新。


# **四、文本与字节序列

1、字符的标识：码位，是0~1114111范围内的数（十进制），在Unicode标准中以4~6个十六进制数表示，前面加“U+”，取值范围是U+0000~U+10FFFF。字符的具体描述取决于所用的编码。编码是在码点和字节序列之间转换时使用的算法。

2、编码问题UnicodeEncodeError。多数非 UTF 编解码器只能处理 Unicode 字符的一小部分子集。把文本转换成字节序列时，如果目标编码中没有定义某个字符，那就会抛出UnicodeEncodeError 异常，除非把 errors 参数传给编码方法或函数，对错误进行特殊处理：`str_.encode('编码方案', errors='replace') #把无法编码的字符换成?`。解码同理。

3、统一字符编码侦测包 Chardet可以识别所支持的 30 种编码。

4、处理文本的最佳实践是“Unicode 三明治”。尽早把输入（例如读取文件时）的字节序列解码成字符串。对输出来说，则要尽量晚地把字符串编码成字节序列。在其他处理过程不进行编解码。

5、使用`open`函数打开文件时始终应该明确传入` encoding= 参数`，因为不同的设备使用的默认编码可能不同。

6、为了正确比较而规范化Unicode字符，建议使用NFC（Normalization Form C）：使用最少的码点构成等价的字符串。
```python
from unicodedata import normalize
s1 = 'café' 
s2 = 'cafe\N{COMBINING ACUTE ACCENT}' 
len(s1), len(s2) # (4,5)
len(normalize('NFC', s1)), len(normalize('NFC', s2)) # (4,4)
```
不区分大小写的比较应该使用 `str.casefold()`。

7、在 Python 中，非 ASCII文本的标准排序方式是使用`locale.strxfrm`函数。如下：
```python
# 巴西产水果的列表排序 
import locale my_locale = locale.setlocale(locale.LC_COLLATE, 'pt_BR.UTF-8') fruits = ['caju', 'atemoia', 'cajá', 'açaí', 'acerola'] 
sorted(fruits, key=locale.strxfrm)
```
或者使用PyUCA 库。
```python
>>> import pyuca
>>> coll = pyuca.Collator()
>>> fruits = ['caju', 'atemoia', 'cajá', 'açaí', 'acerola']
>>> sorted_fruits = sorted(fruits, key=coll.sort_key)
>>> sorted_fruits
['açaí', 'acerola', 'atemoia', 'cajá', 'caju']
```
