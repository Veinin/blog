---
title: Python 快速上手 - 列表
date: 2018-03-18 15:50:00
categories: Python
tags: [Python, 编程]
---

列表和元组可以包含多个值，这样编写程序来处理大量数据就变得更容易。而且，由于列表本身又可以包含其他列表，所以可以用它们将数据安排成层次结构。

## 列表数据类型
“列表” 是一个值， 它包含多个字构成的序列。列表用左方括号开始， 右方括号结束，即[]。
```python
colors = ['red', 'green', 'blue']
nums = [1, 2, 3]
```

<!--more-->

## 用下标取得列表中的单个值
列表后面方括号内的整数被称为“下标”。列表中第一个值的下标是 0，第二个值的下标是 1，第三个值的下标是 2，依此类推。
如果使用的下标超出了列表中值的个数， Python 将给出 IndexError 出错信息。
列表也可以包含其他列表值。这些列表的列表中的值， 可以通过多重下标来访问。
```python
>>> colors = ['red', 'green', 'blue', ['red and blue', 'red and green']]
>>> colors[0]
red
>>> colors[1]
green
>>> colors[2]
blue
>>> colors[1] + colors[2]
greenblue
>>>
>>> pcolors[4][1]
red and green
>>>
>>> colors[5]
IndexError: list index out of range
```

## 负数下标
虽然下标从 0 开始并向上增长，但也可以用负整数作为下标。整数值−1 指的是列表中的最后一个下标， −2 指的是列表中倒数第二个下标，以此类推。
```python
>>> colors = ['red', 'green', 'blue']
>>> colors[-1]
blue
>>> colors[-3]
red
>>> colors[-1] + colors[-3]
bluered
```

## 用 len()取得列表的长度
len()函数将返回传递给它的列表中值的个数， 就像它能计算字符串中字符的个数一样。
```python
>>> colors = ['red', 'green', 'blue']
>>> len(colors)
3
```

## 用下标改变列表中的值
可以使用列表的下标来改变下标处的值。例如， `spam[1] = 'aardvark'` 意味着“将列表 spam 下标 1 处的值赋值为字符串'aardvark'。
```python
>>> colors = ['red', 'green', 'blue']
>>> colors[1] = 'red and blue'
>>> colors[1]
'red and blue'
>>>
>>> colors[-1] = 'blue and red'
>>> print(colors[-1])
blue and red
```

## 列表连接和列表复制
操作符可以连接两个列表， 得到一个新列表， 就像它将两个字符串合并成一个新字符串一样。 *操作符可以用于一个列表和一个整数，实现列表的复制。

```python
>>> a = [1, 2, 3] + ['a', 'b', 'c']
>>> a
[1, 2, 3, 'a', 'b', 'c']
>>>
>>> a = a * 2
>>> a
[1, 2, 3, 'a', 'b', 'c', 1, 2, 3, 'a', 'b', 'c']
```


## 用 del 语句从列表中删除值
del 语句将删除列表中下标处的值， 表中被删除值后面的所有值， 都将向前移动一个下标。
```python
>>> colors = ['red', 'green', 'blue']
>>> del colors[2]
>>> colors
['red', 'green']
>>>
>>> del colors[1]
>>> colors
['red']
```

## 列表用于循环
在 for 循环中可以使用 range(len(someList))， 来迭代列表的每一个下标。
```python
>>> colors = ['red', 'green', 'blue']
>>> for i in range(len(colors)):
        print('Index : ' + str(i) + " in color is: " + colors[i])

Index : 0 in color is: red
Index : 1 in color is: green
Index : 2 in color is: blue
```

## in 和 not in 操作符
利用 in 和 not in 操作符， 可以确定一个值否在列表中。 像其他操作符一样， in和 not in 用在表达式中， 连接两个值： 一个要在列表中查找的值， 以及待查找的列表。这些表达式将求值为布尔值。
```python
>>> colors = ['red', 'green', 'blue']
>>> 'red' in colors
True
>>> 'black' not in colors
True
```

## 多重赋值技巧
多重赋值技巧是一种快捷方式， 让你在一行代码中， 用列表中的值为多个变量赋值。
所以不必像这样：
```python
colors = ['red', 'green', 'blue']
red = colors[0]
green = colors[1]
blue = colors[2]
```

可有使用如下技巧：
```python
colors = ['red', 'green', 'blue']
red, green, blue = colors
```

最后要注意变量的数目和列表的长度必须严格相等， 否则 Python 将给出 ValueError。


## 增强赋值
针对+、 -、 *、 /和%操作符， 都有增强的赋值操作符。
```python
a = 10

a += 1
a -= 2
a *= 2
a /= 2
a %= 5

print(a) # 9.0
```

## 方法
方法和函数是一回事，只是它是调用在一个值上。方法部分跟在这个值后面，以一个句点分隔。
每种数据类型都有它自己的一组方法。例如， 列表数据类型有一些有用的方法，用来查找、 添加、 删除或操作列表中的值。

### 用 index()方法在列表中查找值
列表值有一个 index()方法， 可以传入一个值， 如果该值存在于列表中， 就返回它的下标。如果该值不在列表中， Python 就报 ValueError。
```python
>>> colors = ['red', 'green', 'blue']
>>> colors.index('red')
0
>>> colors.index('black')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: 'black' is not in list
```

### 用 append()和 insert()方法在列表中添加值
使用append()方法调用， 可以将参数添加到列表末尾。
insert()方法可以在列表任意下标处插入一个值。 insert()方法的第一个参数是新值的下标， 第二个参数是要插入的新值。
```python
>>> colors = ['red', 'green', 'blue']
>>> colors.append('black')
>>> colors
['red', 'green', 'blue', 'black']
>>> colors.insert(1, 'orange')
>>> colors
['red', 'orange', 'green', 'blue', 'black']
```

### 用 remove()方法从列表中删除值
给 remove()方法传入一个值，它将从被调用的列表中删除。
```python
>>> colors = ['red', 'orange', 'green', 'blue', 'black']
>>> colors.remove('black')
>>> colors.remove('orange')
>>> colors
['red', 'green', 'blue']
```

### 用 sort()方法将列表中的值排序
数值的列表或字符串的列表， 能用 sort()方法排序。
```python
>>> colors = ['red', 'green', 'blue']
>>> colors.sort()
>>> colors
['blue', 'green', 'red']
>>>
>>> a = [5, 3, 2, 1]
>>> a.sort()
>>> a
[1, 2, 3, 5]
```

也可以指定 reverse 关键字参数为 True， 让 sort()按逆序排序。
```python
>>> a.sort(reverse=True)
>>> a
[5, 3, 2, 1]
```

排序注意事项：
- 首先， sort()方法当场对列表排序。不要写出 colors = colors.sort()这样的代码。
- 其次， 不能对既有数字又有字符串值的列表排序，因为 Python 不知道如何比较它们。
- 第三， sort()方法对字符串排序时， 使用“ASCII 字符顺序”， 而不是实际的字典顺序。这意味着大写字母排在小写字母之前。因此在排序时， 小写的 a 在大写的 Z 之后。

## 类似列表的类型：字符串和元组
列表并不是唯一表示序列值的数据类型。例如， 字符串和列表实际上很相似，只要你认为字符串是单个文本字符的列表。对列表的许多操作， 也可以作用于字符串：按下标取值、 切片、 用于 for 循环、 用于 len()， 以及用于 in 和 not in 操作符。
需要注意的是：字符串是“不可变的”， 它不能被更改。尝试对字符串中的一个字符重新赋值， 将导致 TypeError 错误。
```python
>>> name = 'Veinin'                                    
>>> name[0]                                        
'V'                                             
>>> name[-1]                                       
'n'                                             
>>> 'in' in name                                           
True                                         
>>> 'p' not in name                      
True
>>> name[0] = 'A'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'str' object does not support item assignment                                          
```

字符串是“不可变的”， 它不能被更改。
“改变” 一个字符串的正确方式， 是使用切片和连接。构造一个“新的” 字符串， 从老的字符串那里复制一些部分。
```python
>>> name = 'Veinin'
>>> newName = 'Jali' + name[4:6]
>>> name
'Veinin'
>>> newName
'Jaliin'
>>>
```

## 元组数据类型
除了两个方面，“元组” 数据类型几乎与列表数据类型一样。
首先， 元组输入时用圆括号()， 而不是用方括号[]。
其次，元组像字符串一样， 是不可变的。 元组不能让它们的值被修改、 添加或删除。
```python
>>> colors = ('red', 'green', 'blue')
>>> colors[0]
'red'
>>> colors[-2]
'green'
>>> colors[1:3]
('green', 'blue')
>>> colors[2] = 'black'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
```

如果需要元组值的一个可变版本， 使用函数函数 list() 将元组转换成列表就很方便。 相反也可以使用 tuple() 函数将列表转换成元组。

## 引用
对于字符串和整数值赋值操作，将执行拷贝操作，赋值后二者是不同的变量，保存了不同的值。
但列表不是这样的。 当你将列表赋给一个变量时，实际上是将列表的“引用”赋给了该变量。引用是一个值，指向某些数据。列表引用是指向一个列表的值。
```python
>>> a = 40
>>> b = a
>>> a = 50
>>> a
50
>>> b
40
>>>
>>> foo = [1, 2, 3]
>>> bar = foo
>>> foo[0] = 120
>>> foo
[120, 2, 3]
>>> bar
[120, 2, 3]
>>>
```

## 传递引用
当函数被调用时， 参数的值被复制给变元。对于列表以及字典， 这意味着变元得到的是引用的拷贝。
```python
>>> def something(arr):
...     arr.append(4)
...
>>> arr = [1, 2, 3]
>>> something(arr)
>>> arr
[1, 2, 3, 4]
```

## copy()和 deepcopy() 函数
在处理列表和字典时，尽管传递引用常常是最方便的方法， 但如果函数修改了传入的列表或字典， 你可能不希望这些变动影响原来的列表或字典。要做到这一点，Python 提供了名为 copy 的模块， 其中包含 copy()和 deepcopy()函数。
第一个函数copy.copy()， 可以用来复制列表或字典这样的可变值， 而不只是复制引用。
如果要复制的列表中包含了列表， 那就使用 copy.deepcopy()函数来代替。deepcopy()函数将同时复制它们内部的列表。
```python
import copy

a = [1, 2, 3, [4, 5, 6]]
b = copy.copy(a)

a[0] = 110
a[3][0] = 120
print(a) # [110, 2, 3, [120, 5, 6]]
print(b) # [1, 2, 3, [120, 5, 6]]

c = copy.deepcopy(a)
c[3][0] = 4
print(a) # [110, 2, 3, [120, 5, 6]]
print(c) # [110, 2, 3, [4, 5, 6]]
```