### 切片slice
```python
# 清除首尾空格
def trim(s):
     while s[0]==' ':
         s=s[1:]
     while s[-1]==' ':
         s=s[:-1]
     return s
```

### 生成器（yield）generator   列表生成式List Comprehensions
   `yield` 是 Python 中用于生成器（Generator）的关键字，它允许一个函数产生一系列的值，而不是一次性返回所有值。生成器可以迭代，逐个产生值，从而减小内存占用，并允许处理大量数据流。
```python
# 杨辉三角
def triangles():
    r = [1]
    while True:
        yield(r)
	    r = [1] + [r[i] + r[i+1] for i in range(len(r) - 1)] + [1] 
def main():
    # 创建一个生成器对象
    triangle_gene = triangles()
    # 产生并输出前几行杨辉三角
    for _ in range(5):
        row = next(triangle_gene)
        print(row)
if __name__ == "__main__":
    main()
```

### 迭代器iterator
```python
for x in [1, 2, 3, 4, 5]:
    pass
    
    等价于

it = iter([1, 2, 3, 4, 5])  # 首先获得Iterator对象:
# 循环:
while True:
    try:
        # 获得下一个值:
        x = next(it)
    except StopIteration:
        # 遇到StopIteration就退出循环
        break
```

### map / reduce
```python
from functools import reduce
from functools import reduce

DIGITS = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}

def str2int(s):
    def fn(x, y):
        return x * 10 + y
    def char2num(s):
        return DIGITS[s]
    return reduce(fn, map(char2num, s))

 # 输出结果
  13579
# 简化
from functools import reduce

DIGITS = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}

def char2num(s):
    return DIGITS[s]

def str2int(s):
    return reduce(lambda x, y: x * 10 + y, map(char2num, s))
```

### 过滤器fliter
`filter()`把传入的函数依次作用于每个元素，然后根据返回值是`True`还是`False`决定保留还是丢弃该元素
```python
def is_palindrome(n):
	if str(n)[::-1] == str(n):  
		return n

print(list(filter(is_palindrome,range(100))))
```

### 闭包
 返回闭包时牢记一点：返回函数不要引用任何循环变量，或者后续会发生变化的变量。
 ```python
def createCounter():
    n = 0
    def counter():     # counter 缓刑
        nonlocal n
        n += 1
        return n
    return counter

def main():
    # 创建计数器
    my_counter = createCounter()

    # 使用计数器
    print(my_counter())  # 输出：1
    print(my_counter())  # 输出：2
    print(my_counter())  # 输出：3

if __name__ == "__main__":
    main()

```

### 装饰器decorator
函数对象有一个`__name__`属性，可以拿到函数的名字
**把`@log`放到`now()`函数的定义处，相当于执行了语句：`now = log(now)`**
	由于`log()`是一个decorator，返回一个函数，所以，原来的`now()`函数仍然存在，只是现在同名的`now`变量指向了新的函数，于是调用`now()`将执行新函数，即在`log()`函数中返回的`wrapper()`函数。
```python
def log(func):
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw) # 保持被装饰函数的原始行为
    return wrapper  # 每次调用 `func` 时，实际上是调用了 `wrapper` 函数

@log
def now():
	print('2023-1-2')

now()
```

```python
from functools import wraps

def metric(func):
    @wraps(func)
    def wrapper(*args, **kw):
        start_time = time.time()
        result = func(*args, **kwargs)   # 调用被装饰的原始函数，保存其返回值
        end_time = time.time()
        print('%s executed in %s ms' % (func.__name__, end_time - start_time))
        return result                    # result为函数suspend的返回值
    return wrapper

@metric
def suspend():
    time.sleep(2)
    print("Function executed")
    return "Output"
    
result = suspend()  # 调用被装饰的函数，保存原始函数返回值
print(result)  # 输出原始函数的返回值
```

### 偏函数
`functools.partial`的作用就是，把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数，调用这个新函数会更简单。

```python
from functools import partial

# 原始函数
def power(base, exponent):
    return base ** exponent

# 创建偏函数
square = partial(power, exponent = 2)
cube = partial(power, exponent = 3)

# 使用偏函数
result_square = square(5)  # 相当于调用 power(5, 2)
result_cube = cube(5)  # 相当于调用 power(5, 3)

print(result_square)  # 输出 25
print(result_cube)  # 输出 125

```


### 面向对象编程OOP
#### 访问限制
加下划线 " __ "
```python
class Student(object):

    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    def print_score(self):
        print('%s: %s' % (self.__name, self.__score))
```

实例属性优先级比类属性高
```python
	>>> class Student(object):
	...     name = 'Student'
	...
    >>> s = Student() # 创建实例s
    >>> print(s.name) # 打印name属性，因为实例并没有name属性，所以会继续查找class的name属性
	Student
    >>> print(Student.name) # 打印类的name属性
    Student
    >>> s.name = 'Michael' # 给实例绑定name属性
    >>> print(s.name) # 由于实例属性优先级比类属性高，因此，它会屏蔽掉类的name属性
    Michael
    >>> print(Student.name) # 但是类属性并未消失，用Student.name仍然可以访问
    Student
    >>> del s.name # 如果删除实例的name属性
    >>> print(s.name) # 再次调用s.name，由于实例的name属性没有找到，类的name属性就显示出来了
    Student
```
```python
class Student(object):
    count = 0

    def __init__(self, name):
        self.name = name
        Student.count += 1    # 调用类属性
```

### 面向对象高级特性
还可以尝试给实例绑定一个方法：
```python
    >>> def set_age(self, age): # 定义一个函数作为实例方法
    ...     self.age = age
    ...
    >>> from types import MethodType
    >>> s.set_age = MethodType(set_age, s) # 给实例绑定一个方法
    >>> s.set_age(25) # 调用实例方法
    >>> s.age # 测试结果
    25
    ```
为了给所有实例都绑定方法，可以给class绑定方法：
```python
	>>> def set_score(self, score):
	>>> ...     self.score = score
	>>> ...
	>>> Student.set_score = set_score

```
#### __ slots __ 
限制该class实例能添加的属性
	仅对当前类实例起作用，对继承的子类是不起作用的：
	除非在子类中也定义`__slots__`，这样，子类实例允许定义的属性就是自身的`__slots__`加上父类的`__slots__`。
```python
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```

#### @property 

    用于将一个方法转换为属性。通过使用 `@property`，你可以在不改变类接口的情况下访问对象的属性，并且你可以在访问属性时执行一些定制的逻辑。
```python
class MyClass: 
	def __init__(self, value): 
		self.__value = value # 以一个下划线开头的属性通常被认为是私有的 

	@property 
	def value(self): 
		print("Getting value") # 这里可以添加定制的逻辑 
		return self.__value
		
	@value.setter 
	def value(self, new_value): 
		print("Setting value") # 这里可以添加定制的逻辑 
		self.__value = new_value 
		
	@value.deleter 
	def value(self): 
		print("Deleting value") # 这里可以添加定制的逻辑 
		del self.__value 
		
# 创建一个实例 
obj = MyClass(42) 

# 访问属性 
print(obj.value) # 调用 value 方法，并输出 "Getting value" 和 42 

# 设置属性 
obj.value = 100 # 调用 value 方法，并输出 "Setting value" 

# 删除属性 
del obj.value # 调用 value 方法，并输出 "Deleting value"
```


#### 多重继承
```python
class Dog(Mammal, RunnableMixIn, CarnivorousMixIn):
    pass
```
#### __ iter __
   返回一个迭代对象，然后，Python的for循环就会不断调用该迭代对象的`__next__()`方法拿到循环的下一个值，直到遇到`StopIteration`错误时退出循环。
```python
class Fib(object):
    def __init__(self):
        self.a, self.b = 0, 1 # 初始化两个计数器a，b

    def __iter__(self):
        return self # 实例本身就是迭代对象，故返回自己

    def __next__(self):
        self.a, self.b = self.b, self.a + self.b # 计算下一个值
        if self.a > 100000: # 退出循环的条件
            raise StopIteration()
        return self.a # 返回下一个值

for n in Fib():
	print(n)
```
####  __ getitem __
   像list那样按照下标取出元素
   f = Fib()
   f[0]
```python
```class CustomObject:
    def __init__(self, data):
        self.data = data

    def __getitem__(self, key):
        # 处理单个索引
        if isinstance(key, int):
            # 处理负数索引
            if key < 0:
                key = len(self.data) + key
            return self.data[key]
        # 处理切片
        elif isinstance(key, slice):
            start, stop, step = key.start, key.stop, key.step
            start = start if start is not None else 0
            stop = stop if stop is not None else len(self.data)
            step = step if step is not None else 1

            # 处理负数索引
            start = start if start >= 0 else len(self.data) + start
            stop = stop if stop >= 0 else len(self.data) + stop

            return [self.data[i] for i in range(start, stop, step)]
        else:
            raise TypeError("Unsupported key type")

# 示例使用
custom_obj = CustomObject([1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89])
print(custom_obj[2])         # 输出: 2
print(custom_obj[:10:2])     # 输出: [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
print(custom_obj[::-1])      # 输出: [89, 55, 34, 21, 13, 8, 5, 3, 2, 1, 1]
```
#### __ setitem __ 
```python
class CustomDict:
    def __init__(self):
        self.data = {}

    def __getitem__(self, key):
        # 获取元素的操作
        return self.data[key]

    def __setitem__(self, key, value):
        # 设置元素的操作
        self.data[key] = value

    def __delitem__(self, key):
        # 删除元素的操作
        del self.data[key]

# 创建 CustomDict 类的实例
custom_obj = CustomDict()

# 使用中括号进行赋值操作，类似字典
custom_obj['a'] = 1
custom_obj['b'] = 2

# 使用 del 语句删除元素
del custom_obj['a']
```
#### __ getattr __
   动态返回一个本不存在的属性
   自动把不存在的**属性变为其参数**
```python
class Student(object):

    def __init__(self):
        self.name = 'Michael'

    def __getattr__(self, attr):
        if attr == 'score':
            return 99
	        raise AttributeError('\'Student\' object has no attribute \'%s\'' % attr)
```

```python
class Chain(object):
    def __init__(self, path=''):
        self._path = path
        
    def __getattr__(self, path):
        return Chain('%s/%s' % (self._path, path)) # path参数就是下面的users

    def __call__(self, user):
        return Chain('%s/%s' % (self._path.replace(':user', user), ''))

    def __str__(self):
        return self._path

    __repr__ = __str__


Chain().users(':user').repos(user='michael')  # users属性不存在，调用__getattr__
# 返回
"/users/michael/repos"
```
#### type（）
使用type() 创建类
1. class的名称；
2. 继承的父类集合，注意Python支持多重继承，如果只有一个父类，别忘了tuple的单元素写法；
3. class的方法名称与函数绑定，这里我们把函数`fn`绑定到方法名`hello`上。
```python
>>> def fn(self, name='world'): # 先定义函数
...     print('Hello, %s.' % name)
...
>>> Hello = type('Hello', (object,), dict(hello=fn)) # 创建Hello class
>>> h = Hello()
>>> h.hello()
Hello, world.
>>> print(type(Hello))
<class 'type'>
>>> print(type(h))
<class '__main__.Hello'>
```


### 错误处理

#### logging  **调试终极武器**
打印错误栈，并正常运行程序
```python 
# err_logging.py

import logging

def foo(s):
    return 10 / int(s)

def bar(s):
    return foo(s) * 2

def main():
    try:
        bar('0')
    except Exception as e:
        logging.exception(e)

main()
print('END')
```
#### 抛出错误
捕获错误目的只是记录一下，便于后续追踪。但是，由于当前函数不知道应该怎么处理该错误，所以，最恰当的方式是继续往上抛 raise，让顶层调用者去处理。
```python
# err_reraise.py

def foo(s):
    n = int(s)
    if n == 0:
        raise ValueError('invalid value: %s' % s)
    return 10 / n

def bar():
    try:
        foo('0')
    except ValueError as e:
        print('ValueError!')
        raise

bar()
```
`raise`语句如果不带参数，就会把当前错误原样抛出。此外，在`except`中`raise`一个Error，还可以把一种类型的错误转化成另一种类型：
```python
try:
    10 / 0
except ZeroDivisionError:
    raise ValueError('input error!')
```
#### logging
import logging

#### 配置 logging
```python
import logging
# 配置logging
logging.basicConfig(level=logging.DEBUG)

def divide(a, b):
    logging.debug(f'Dividing {a} by {b}')
    result = a / b
    logging.debug(f'Result: {result}')
    return result

def main():
    num1 = 10
    num2 = 2

    # 调用需要调试的函数
    result = divide(num1, num2)

    # 输出结果
    print(f'Result: {result}')

if __name__ == "__main__":
    main()
    ```
#### mydict.py
```python
class Dict(dict):

    def __init__(self, **kw):
        super().__init__(**kw)

    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Dict' object has no attribute '%s'" % key)

    def __setattr__(self, key, value):
        self[key] = value
```
####  编写单元测试  测试类
[[03. Testing#单元测试（Unit Test）]]
测试类，从`unittest.TestCase`继承
以`test`开头的方法就是测试方法，不以`test`开头的方法不被认为是测试方法，测试的时候不会被执行
```python
import unittest

from mydict import Dict

class TestDict(unittest.TestCase):
 
    def test_init(self):  
    # 检查初始化时是否正确设置了属性。同时验证 `d` 是 `dict` 类型的实例
        d = Dict(a=1, b='test')
        self.assertEqual(d.a, 1)
        self.assertEqual(d.b, 'test')
        self.assertTrue(isinstance(d, dict))

    def test_key(self):
        d = Dict()
        d['key'] = 'value'
        self.assertEqual(d.key, 'value')

    def test_attr(self):
        d = Dict()
        d.key = 'value'
        self.assertTrue('key' in d)
        self.assertEqual(d['key'], 'value')

    def test_keyerror(self):
	    # 尝试获取一个不存在的键 `'empty'`，并断言是否引发了 `KeyError`
        d = Dict()
        with self.assertRaises(KeyError):
            value = d['empty']

    def test_attrerror(self):
        d = Dict()
        with self.assertRaises(AttributeError):
            value = d.empty
```


### 正则表达式
```python
import re
pattern = re.compile(r'\d+')  # 编译正则表达式，匹配一个或多个数字
text = "12345"
match = pattern.match(text)  # 使用编译后的正则表达式对象进行匹配
print(bool(match))
```
#### 切分字符串
```python
>>> re.split(r'[\s\,\;]+', 'a,b;; c  d')
['a', 'b', 'c', 'd']
```
#### 提取子串
```python
>>> t = '19:05:30'
>>> m = re.match(r'^(0[0-9]|1[0-9]|2[0-3]|[0-9])\:(0[0-9]|1[0-9]|2[0-9]|3[0-9]|4[0-9]|5[0-9]|[0-9])\:(0[0-9]|1[0-9]|2[0-9]|3[0-9]|4[0-9]|5[0-9]|[0-9])$', t)
>>> m.groups()
('19', '05', '30')
```
#### 贪婪匹配

正则匹配默认是匹配尽可能多的字符。举例如下，匹配出数字后面的`0`：
```python
>>> re.match(r'^(\d+)(0*)$', '102300').groups()
('102300', '')
```
由于`\d+`采用贪婪匹配，直接把后面的`0`全部匹配了，结果`0*`只能匹配空字符串了。
必须让`\d+`采用非贪婪匹配（也就是尽可能少匹配），才能把后面的`0`匹配出来，加个`?`就可以让`\d+`采用非贪婪匹配：
```python
>>> re.match(r'^(\d+?)(0*)$', '102300').groups()
('1023', '00')
```