---
data: 2024-07-26
---
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
# **三、Python的字典与集合**

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
# **四、文本与字节序列**

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

或者使用PyUCA 库：

```python
>>> import pyuca
>>> coll = pyuca.Collator()
>>> fruits = ['caju', 'atemoia', 'cajá', 'açaí', 'acerola']
>>> sorted_fruits = sorted(fruits, key=coll.sort_key)
>>> sorted_fruits
['açaí', 'acerola', 'atemoia', 'cajá', 'caju']
```
# **五、函数对象**

1、使用列表推导式来代替`map`和`filter`函数。

2、任何 Python 对象都可以表现得像函数。为此，只需实现实例方法` __call__`。

3、定位参数和仅限关键字参数。

```python
# 使用reduce函数和operator.mul函数计算阶乘 
from functools import reduce 
from operator import mul 
def factorial(n): 
	return reduce(mul, range(1, n + 1))
```
```python
# 使用itemgetter排序一个元组列表 
metro_data = [ ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)), ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)), ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)), ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)), ('São Paulo', 'BR', 19.649, (-23.547778, -46.635833)), ] 
from operator import itemgetter 
for city in sorted(metro_data, key=itemgetter(1)): 
	print(city)
```

```python
('São Paulo', 'BR', 19.649, (-23.547778, -46.635833)) 
('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)) 
('Tokyo', 'JP', 36.933, (35.689722, 139.691667)) 
('Mexico City', 'MX', 20.142, (19.433333, -99.133333)) 
('New York-Newark', 'US', 20.104, (40.808611, -74.020386))
```

4、支持函数式编程的包

```python
# 使用reduce函数和operator.mul函数计算阶乘 
from functools import reduce 
from operator import mul 
def factorial(n): 
	return reduce(mul, range(1, n + 1))
```
```python
# 使用itemgetter排序一个元组列表 
metro_data = [ ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)), ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)), ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)), ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)), ('São Paulo', 'BR', 19.649, (-23.547778, -46.635833)), ] 
from operator import itemgetter 
for city in sorted(metro_data, key=itemgetter(1)): 
	print(city)
```

```python
('São Paulo', 'BR', 19.649, (-23.547778, -46.635833)) 
('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)) 
('Tokyo', 'JP', 36.933, (35.689722, 139.691667)) 
('Mexico City', 'MX', 20.142, (19.433333, -99.133333)) 
('New York-Newark', 'US', 20.104, (40.808611, -74.020386))
```
# **六、函数中的类型提示**

1、**渐进式类型**
渐进式类型系统的性质：
1）是可选的：默认情况下，类型检查工具不应对没有类型提示的代码发出警告。如果无法确定对象的类型时，会假定为Any类型。
2）不在运行时捕获类型错误：在运行时不能阻止把不一致的值传给函数或分配给变量。
3）不能改善性能。
2、**类型由受支持的操作定义**
1）鸭子类型：对象有类型，但变量（包括参数）没有类型。为对象声明的类型无关紧要，重要的是对象具体支持什么操作。比名义类型更灵活，但代价是运行时潜在的错误更多。
2）名义类型：对象只存在于运行时，类型检查工具只关心使用类型提示注解变量（包括参数）的源码。比鸭子类型更严格，优点是能在构建流水线中，甚至在IDE中输入代码的过程中更早地捕获一些bug。
3、**注释中可用的类型**
1）typing.Any：动态类型，与任何类型相容
2）基本类型：int、float、str、bytes
3）泛化容器：可以用类型参数声明，如：`List[str]`
4）元组类型：如`tuple[str, float, ...]`
5)泛化映射：使用`MappingType[KeyType, ValueType]`形式注解，如：

```python
	def name_index(start: int = 32, end: int) -> dict[str, set[str]]:
```

6）抽象基类：一般来说，在参数得类型提示中最好使用`abc.Mapping`或`abc.MutableMapping`，不要使用`dict`
# **七、使用一等函数实现设计模式**

经典的策略模式来处理订单折扣如下图所示：
![[Pasted image 20240728092457.png]]
**典型示例：**
根据客户的属性或订单中的商品计算折扣，假如某个网店制定了以下折扣规则：
1. 有1000或以上积分的顾客，每个订单享5%折扣。
2. 同一个订单中，单个商品的数量达到20个或以上，享10%折扣。
3. 订单中不同商品的数量达到10个或以上，享7%折扣。
代码如下：

```python
from abc import ABC, abstractmethod
from collections.abc import Sequence
from decimal import Decimal
from typing import NamedTuple, Optional


class Customer(NamedTuple):
    name: str
    fidelity: int


class LineItem(NamedTuple):
    product: str
    quantity: int
    price: Decimal

    def total(self) -> Decimal:
        return self.price * self.quantity


class Order(NamedTuple):
    customer: Customer
    cart: Sequence[LineItem]
    promotion: Optional['Promotion'] = None

    def total(self) -> Decimal:
        totals = (item.total() for item in self.cart)
        return sum(totals, start=Decimal(0))

    def due(self) -> Decimal:
        if self.promotion is None:
            discount = Decimal(0)
        else:
            discount = self.promotion.discount(self)
        return self.total() - discount

    def __repr__(self):
        return f'<Order total: {self.total():.2f} due: {self.due():.2f}>'


class Promotion(ABC):
    """策略：抽象基类"""
    @abstractmethod
    def discount(self, order: Order) -> Decimal:
        """Return discount as a positive dollar amount"""


class FidelityPromo(Promotion):
    """第一个具体策略：为积分是1000或以上的顾客提供5%折扣。"""

    def discount(self, order: Order) -> Decimal:
        rate = Decimal('0.05')
        if order.customer.fidelity >= 1000:
            return order.total() * rate
        return Decimal(0)


class BulkItemPromo(Promotion):
    """第二个具体策略：单个商品的数量为20个或以上时，提供10%折扣。"""

    def discount(self, order: Order) -> Decimal:
        discount = Decimal(0)
        for item in order.cart:
            if item.quantity >= 20:
                discount += item.total() * Decimal('0.1')
        return discount


class LargeOrderPromo(Promotion):
    """第三个具体策略：订单中不同商品的数量达到10个或以上时，提供7%折扣。"""

    def discount(self, order: Order) -> Decimal:
        distinct_items = {item.product for item in order.cart}
        if len(distinct_items) >= 10:
            return order.total() * Decimal('0.07')
        return Decimal(0)
```

将上述策略类代码使用函数进行改写，把具体策略换成简单的函数，去掉抽象基类`Promotion`，Order类也修改了。如下：

```python
from dataclasses import dataclass
from typing import Callable

@dataclass(frozen=True)
class Order:
    customer: Customer
    cart: Sequence[LineItem]
    # 可以接收一个Order参数，并返回一个Decimal值的可调用对象
    promotion: Optional[Callable[['Order'], Decimal]] = None

    def total(self) -> Decimal:
        totals = (item.total() for item in self.cart)
        return sum(totals, start=Decimal(0))

    def due(self) -> Decimal:
        if self.promotion is None:
            discount = Decimal(0)
        else:
            discount = self.promotion(self)
        return self.total() - discount

    def __repr__(self):
        return f'<Order total: {self.total():.2f} due: {self.due():.2f}>'

def fidelity_promo(order: Order) -> Decimal:
    """第一个具体策略：为积分是1000或以上的顾客提供5%折扣。"""
    if order.customer.fidelity >= 1000:
        return order.total() * Decimal('0.05')
    return Decimal(0)


def bulk_item_promo(order: Order) -> Decimal:
    """第二个具体策略：单个商品的数量为20个或以上时，提供10%折扣。"""
    discount = Decimal(0)
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * Decimal('0.1')
    return discount


def large_order_promo(order: Order) -> Decimal:
    """第三个具体策略：订单中不同商品的数量达到10个或以上时，提供7%折扣。"""
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
        return order.total() * Decimal('0.07')
    return Decimal(0)
```

添加选择最佳策略的代码——最简单的实现：

```python
promos = [fidelity_promo, bulk_item_promo, large_order_promo]


def best_promo(order: Order) -> Decimal:
    """选择可用的最佳折扣"""
    return max(promo(order) for promo in promos)
```

使用`globals`函数，帮助`best_promo`自动找到其他可用的`*_promo`函数：

```python
promos = [promo for name, promo in globals().items()  
                if name.endswith('_promo') and        
                   name != 'best_promo'               
]


def best_promo(order: Order) -> Decimal:              
    """选择可用的最佳折扣"""
    return max(promo(order) for promo in promos)
```

使用`inspect.getmembers`函数获取对象的属性，从而实现：

```python
import inspect
# 该模块包含所有的策略函数
import promotions

# 得到promotins模块下所有的函数对象
promos = [func for _, func in inspect.getmembers(promotions, inspect.isfunction)]


def best_promo(order: Order) -> Decimal:
    """选择可用的最佳折扣"""
    return max(promo(order) for promo in promos)
```

使用装饰器来改进策略：

```python
Promotion = Callable[[Order], Decimal]

promos: list[Promotion] = []

# 注册装饰器
def promotion(promo: Promotion) -> Promotion:
    promos.append(promo)
    return promo


def best_promo(order: Order) -> Decimal:
    """选择可用的最佳折扣"""
    return max(promo(order) for promo in promos)  # <3>


@promotion
def fidelity(order: Order) -> Decimal:
    """第一个具体策略：为积分是1000或以上的顾客提供5%折扣。"""
    if order.customer.fidelity >= 1000:
        return order.total() * Decimal('0.05')
    return Decimal(0)


@promotion
def bulk_item(order: Order) -> Decimal:
    """第二个具体策略：单个商品的数量为20个或以上时，提供10%折扣。"""
    discount = Decimal(0)
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * Decimal('0.1')
    return discount


@promotion
def large_order(order: Order) -> Decimal:
    """第三个具体策略：订单中不同商品的数量达到10个或以上时，提供7%折扣。"""
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
        return order.total() * Decimal('0.07')
    return Decimal(0)
```

优点：
1. 促销策略函数无须使用特殊的名称（不用以`_promo`结尾）。
2. `@promotion`装饰器突出了被装饰的函数作用，还便于临时禁用某个促销策略。
3. 促销折扣策略可以在其他模块中定义。
# **八、装饰器与闭包**
1、装饰器的基本性质：
- 装饰器是一个函数或其他可调用对象。
- 装饰器可以把被装饰的函数替换成别的函数。
- 装饰器在加载（导入）模块时立即执行，而被装饰的函数只在明确调用时运行。
```python
@decorate
def target():
    print('running target()')
```

等价于：

```python
def target():
    print('running target()')
target = decorate(target)
```

函数对象替换：

```python
def deco(func): 
	def inner(): print('running inner') 
	return inner 
@deco 
def target(): 
	print('running target()')
print(target()) -> "running inner"
#函数对象被替换
print(target) -> <function __main__.deco.<locals>.inner()>
```

2、闭包：延伸了作用域的函数，包括函数（f）主体中引用的非全局变量和局部变量。这些变量必须来自包含f的外部函数的局部作用域。![[Pasted image 20240730073630.png]]
审查`make_avrtager`创建的函数：
```python
avg = make_averager()
>>> avg.__code__.co_varnames
('new_value', 'total')
>>> avg.__code__.co_freevars
('series',)
```
当多次调用函数对象`avg`时可以发现，虽然`averager`函数的定义域不可用，但是series这一自由变量的值仍然保留（可以理解为类的概念，series为类的属性，`averager`为类的方法），但注意如果series这一自由变量为不可变类型在`averager`进行了赋值操作，则在`averager`内部的series将会变为局部变量，而不是自由变量。例如：
```python
# 计算累计平均值，不保存所有历史
def make_averager():
    count = 0
    total = 0
    
    def averager(new_value):
        count += 1
        total += new_value
        return total / count
    
    return averager
```
报错：
```python
>>> avg = make_averager()
>>> avg(10)
Traceback (most recent call last):
...
UnboundLocalError: local variable 'count' referenced before assignment
>>>
```
在 `averager` 的定义体中为 count 赋值了，这会把 count 变成局部变量。total 变量也受这个问题影响。
解决方法是在count和total前面加nonlocal
```python
# 计算累计平均值，不保存所有历史
def make_averager():
    count = 0
    total = 0
    
    def averager(new_value):
        nonlocal count, total
        count += 1
        total += new_value
        return total / count
    
    return averager
```

变量查找逻辑：
- 如果是`global x`声明，则`x`来自模块全局作用域，并赋予那个作用域中`x`的值。
- 如果是`nonlocal x`声明，则`x`来自最近一个定义它的外层函数，并赋予那个函数中局部变量`x`的值。
- 如果`x`是参数，或者在函数主体中赋了值，那么`x`就是局部变量。
- 如果引用了`x`，但是没有赋值也不是参数，则需要遵循以下规则：
    - 在外层函数主体的局部作用域（非局部作用域）内查找`x`。
    - 如果在外层作用域类没有找到，则从模块全局作用域内读取。
    - 如果在模块全局作用域内没有找到，则从`__builtins__.__dict__`中读取。

3、单分派泛化函数。`functools.singledispatch`装饰器可以把整体方案拆分成多个模块，甚至可以为第三方包中无法编辑的类型提供专门的函数，将普通函数变成了泛化函数的入口，即为单分派。如果根据多个参数选择专门的函数，则是多分派。例子：开发一个调试Web应用程序的工具，生成HTML，以显示不同类型的Python对象。需要满足如下功能：
-  当参数为`str`时，内部的换行符替换为`<br/>\n`，不使用`<pre>`标签，使用`<p>`。
- 当参数为`int`时，以十进制和十六进制显示数（bool除外）。
- 当参数为`list`时，输出一个HTML列表，根据各项的类型进行格式化。
- 当参数为`float`和`Decimal`时，正常输出值，外加分数形式。
```python
from functools import singledispatch
from collections import abc
import fractions
import decimal
import html
import numbers

@singledispatch #**单分派泛化函数代替函数重载**
def htmlize(obj: object) -> str:
    content = html.escape(repr(obj))
    return f'<pre>{content}</pre>'

@htmlize.register #当被装饰函数有参数类型时，register不需添加类型参数
def _(text: str) -> str: 
    content = html.escape(text).replace('\n', '<br/>\n')
    return f'<p>{content}</p>'

@htmlize.register
def _(seq: abc.Sequence) -> str:
    inner = '</li>\n<li>'.join(htmlize(item) for item in seq)
    return '<ul>\n<li>' + inner + '</li>\n</ul>'

@htmlize.register
def _(n: numbers.Integral) -> str:
    return f'<pre>{n} (0x{n:x})</pre>'

@htmlize.register
def _(n: bool) -> str:
    return f'<pre>{n}</pre>'

@htmlize.register(fractions.Fraction)
def _(x) -> str:
    frac = fractions.Fraction(x)
    return f'<pre>{frac.numerator}/{frac.denominator}</pre>'

@htmlize.register(decimal.Decimal)
@htmlize.register(float) #没有参数类型则需添加
def _(x) -> str:
    frac = fractions.Fraction(x).limit_denominator()
    return f'<pre>{x} ({frac.numerator}/{frac.denominator})</pre>'
```

4、参数化装饰器。装饰器将被装饰函数作为第一个参数，而被装饰函数的参数则作为装饰器内部函数的参数调用。装饰器为了接受除被装饰函数外的额外参数创建一个装饰器工厂函数，把参数传给它，返回一个装饰器。