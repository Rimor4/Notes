### python_print
- **objects** -- 复数，表示可以一次输出多个对象。输出多个对象时，需要用 , 分隔。
- **sep** -- 用来间隔多个对象，默认值是一个空格。
- **end** -- 用来设定以什么结尾。默认值是换行符 \n，我们可以换成其他字符串。
- **file** -- 要写入的文件对象。
- **flush** -- 输出是否被缓存通常决定于 file，但如果 flush 关键字参数为 True，流会被强制刷新。
```python
print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)
```

```python
print(f"hello, {answer}")
```
```python
print("Hello", end=" ")  # 将print 末尾默认换行符改为" "
print("World")
```
```python
# 加载动画
import time

print("---RUNOOB EXAMPLE ： Loading 效果---")

print("Loading",end = "")
for i in range(20):
    print(".",end = '',flush = True)
    time.sleep(0.5)
```
### python数据类型
```python
  range  '''   '''
  list   '''列表'''
  tuple  '''元组'''
  dict   '''字典'''
  set    '''   '''
```


```python
# Words in dictionary
words = set()


def check(word):
    '''Return true if word is in dictionary else false'''
    if word.lower() in words:
        return True
    else:
        return False


def load(dictionary):
    """Load dictionary into memory, returning true if successful else false"""
    file = open(dictionary, "r")
    for line in file:
        word = line.rstrip()  ''' 去除后面的空白字符 '''
        words.add(word)   '''向words中加入word'''
    file.close()
    return True


def size():
    """Returns number of words in dictionary if loaded else 0 if not yet loaded"""
    return len(words)


def unload():
    """Unloads dictionary from memory, returning true if successful else false自动释放分配的空间"""
    return True
```
### python字符串
[头下标:尾下标] 获取的子字符串包含头下标的字符，但不包含尾下标的字符(还可以设置一个截取步长的参数)
```python
>>> s = 'abcdef'
>>> s[1:5]
'bcde'
```

```python
str = 'Hello World!'
 
print str           # 输出完整字符串
print str[0]        # 输出字符串中的第一个字符
print str[2:5]      # 输出字符串中第三个至第六个之间的字符串
print str[2:]       # 输出从第三个字符开始的字符串
print str * 2       # 输出字符串两次
print str + "TEST"  # 输出连接的字符串
```
### python列表(list)
.append()用于在列表的末尾添加一个元素
```python
list = [ 'runoob', 786 , 2.23, 'john', 70.2 ]
tinylist = [123, 'john']
 
print list               # 输出完整列表
print list[0]            # 输出列表的第一个元素
print list[1:3]          # 输出第二个至第三个元素 
print list[2:]           # 输出从第三个开始至列表末尾的所有元素
print tinylist * 2       # 输出列表两次
print list + tinylist    # 打印组合的列表
```
```python
['runoob', 786, 2.23, 'john', 70.2]
runoob
[786, 2.23]
[2.23, 'john', 70.2]
[123, 'john', 123, 'john']
['runoob', 786, 2.23, 'john', 70.2, 123, 'john']
```
```python
'''list() 函数'''
aTuple = (123, 'Google', 'Runoob', 'Taobao')
list1 = list(aTuple)
print ("列表元素 : ", list1)

str="Hello World"
list2=list(str)
print ("列表元素 : ", list2)
```
### python元组(tuple) 
相当于只读列表， 但它可以包含可变的对象，比如list列表
```python
tup1 = ()    # 空元组
tup2 = (20,) # 一个元素，需要在元素后添加逗号

```
### python_bool
在 Python 中，所有非零的数字和非空的字符串、列表、元组等数据类型都被视为 ==True==，只有 **0、空字符串、空列表、空元组**等被视为 ==False==。

### python集合(set)
创建一个空集合必须用 set() 而不是 { }
```python

sites = {'Google', 'Taobao', 'Runoob', 'Facebook', 'Zhihu', 'Baidu'}
print(sites)   # 输出集合，重复的元素被自动去掉

# 成员测试
if 'Runoob' in sites :
    print('Runoob 在集合中')
else :
    print('Runoob 不在集合中')
    
# set可以进行集合运算
a = set('abracadabra')
b = set('alacazam')

print(a)
print(a - b)     # a 和 b 的差集
print(a | b)     # a 和 b 的并集
print(a & b)     # a 和 b 的交集
print(a ^ b)     # a 和 b 中不同时存在的元素
```

### python字典(dict)
```python
#!/usr/bin/python3

dict = {}
dict['one'] = "1 - 菜鸟教程"
dict[2]     = "2 - 菜鸟工具"

tinydict = {'name': 'runoob','code':1, 'site': 'www.runoob.com'}


print (dict['one'])       # 输出键为 'one' 的值
print (dict[2])           # 输出键为 2 的值
print (tinydict)          # 输出完整的字典
print (tinydict.keys())   # 输出所有键
print (tinydict.values()) # 输出所有值
# 构造函数 dict()
>>> dict([('Runoob', 1), ('Google', 2), ('Taobao', 3)])
{'Runoob': 1, 'Google': 2, 'Taobao': 3}
>>> {x: x**2 for x in (2, 4, 6)}
{2: 4, 4: 16, 6: 36}
>>> dict(Runoob=1, Google=2, Taobao=3)
{'Runoob': 1, 'Google': 2, 'Taobao': 3}
```


### range
- `start`: 可选参数，表示数列的起始值，默认为 0。
- `stop`: 必需参数，表示数列的结束值（不包含在数列内）。
- `step`: 可选参数，表示数列的步长，默认为 1。
```python
for i in range(5):
    print(i)


for i in range(1, 10, 2):   # 输出：   1 3 5 7 9
    print(i)


my_list = list(range(5))
print(my_list)  # 输出：[0, 1, 2, 3, 4]


```




### python内置函数
1. **类型相关函数：**
    
    - `int()`: 将一个数值或字符串转换为整数。
    - `float()`: 将一个数值或字符串转换为浮点数。
    - `str()`: 将对象转换为字符串。
    - `list()`: 将可迭代对象转换为列表。
    - `tuple()`: 将可迭代对象转换为元组。
    - `dict()`: 创建一个字典或将可迭代对象转换为字典。
2. **数学相关函数：**
    
    - `abs()`: 返回一个数的绝对值。
    - `max()`: 返回可迭代对象中的最大值。
    - `min()`: 返回可迭代对象中的最小值.
    - `sum()`: 返回可迭代对象中所有元素的和。
3. **输入输出相关函数：**
    
    - `input()`: 从用户获取输入。
    - `print()`: 打印输出到控制台。
4. **容器操作函数：**
    
    - `len()`: 返回容器（如字符串、列表、元组等）的长度。
    - `sorted()`: 对可迭代对象进行排序。
5. **逻辑与判断函数：**
    
    - `bool()`: 将一个值转换为布尔类型。
    - `all()`: 判断可迭代对象中所有元素是否为真。
    - `any()`: 判断可迭代对象中是否有任一元素为真。
6. **文件操作函数：**
    
    - `open()`: 打开文件。
    - `read()`: 读取文件内容。
    - `write()`: 写入文件内容。
    - `close()`: 关闭文件。
7. **其他常用函数：**
    
    - `range()`: 创建一个范围对象，常用于循环。
    - `enumerate()`: 枚举可迭代对象的元素及其索引。
    - `zip()`: 将多个可迭代对象组合成元组的序列。
**sorted(iterable, key=None, reverse=False)**:
e.g. 
```python
# Sorts favorites by value using lambda function
import csv

# Open CSV file
with open("favorites.csv", "r") as file:

    # Create DictReader
    reader = csv.DictReader(file)

    # Counts
    counts = {}

    # Iterate over CSV file, counting favorites
    for row in reader:
        favorite = row["language"]
        if favorite in counts:
            counts[favorite] += 1
        else:
            counts[favorite] = 1

  

# Print count/s
for favorite in sorted(counts, key=lambda language: counts[language], reverse=True):  """ sorted 函数将对counts中每个元素调用 lambda 函数，以便根据计数来排序"""
    print(f"{favorite}: {counts[favorite]}")
```

**lambda arguments: expression**:
```python
add = lambda x, y: x + y
result = add(3, 5)
print(result)  # 输出：8
```
