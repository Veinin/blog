---
title: Python 快速上手 - 控制流
date: 2018-03-18 15:22:00
categories: Python
tags: [Python, 编程]
---

程序就是一系列指令。但编程真正的力量不仅在于运行（或“执行”） 一条接一条的指令， 就像周末的任务清单那样。根据表达式求值的结果，程序可以决定跳过指令， 重复指令， 或从几条指令中选择一条运行。实际上， 你几乎永远不希望程序从第一行代码开始， 简单地执行每行代码， 直到最后一行。“控制流语句” 可以决定在什么条件下执行哪些 Python 语句。

## 布尔值
“布尔” 数据类型只有两种值： True 和 False。 Boolean（布尔） 的首字母大写， 因为这个数据类型是根据数学家 George Boole 命名的。
```python
>>> spam = True
>>> spam
True
>>> true
Traceback (most recent call last):
File "<pyshell#2>", line 1, in <module>
true
NameError: name 'true' is not defined
>>> True = 2 + 2
SyntaxError: assignment to keyword
```

<!--more-->

## 比较操作符
“比较操作符” 比较两个值，求值为一个布尔值。
```python
>>> 42 == 42
True
>>> 42 == 99
False
>>> 2 != 3
True
>>> 2 != 2
False
```

## 布尔操作符
3 个布尔操作符（and、 or 和 not） 用于比较布尔值。
```python
>>> True and True
True
False or True
True
>>> not True
False
(4 < 5) and (5 < 6)
True
```

## 控制流语句

### if 语句
if 语句的子句（也就是紧跟 if 语句的语句块），将在语句的条件为 True 时执行。如果条件为 False，子句将跳过。
if 语句包含以下部分：
- if 关键字；
- 条件（即求值为 True 或 False 的表达式）；
- 冒号；
- 在下一行开始，缩进的代码块（称为 if 子句）。
```python
if name == 'Alice':
    print('Hi, Alice.')
```

### else 语句
if 子句后面有时候也可以跟着 else 语句。只有 if 语句的条件为 False 时， else子句才会执行。
lse 语句中包
含下面部分：
- else 关键字；
- 冒号；
- 在下一行开始，缩进的代码块（称为 else 子句）。
```python
if name == 'Alice':
    print('Hi, Alice.')
else:
    print('Hello, stranger.')
```

### elif 语句
有时候可能你希望，“许多” 可能的子句中有一个被执行。 elif 语句是“否则如果”，总是跟在 if 或另一条 elif 语句后面。
在代码中， elif 语句
总是包含以下部分：
- elif 关键字；
- 条件（即求值为 True 或 False 的表达式）；
- 冒号；
- 在下一行开始，缩进的代码块（称为 elif 子句）。

### while 循环语句
while 语句总是包含下面几
部分：
- 关键字；
- 条件（求值为 True 或 False 的表达式）；
- 冒号；
- 从新行开始，缩进的代码块（称为 while 子句）。
```python
spam = 0
while spam < 5:
    print('Hello, world.')
    spam = spam + 1
```

### break 语句
如果执行遇到 break 语句，就会马上退出 while 循环子句。在代码中， break 语句仅包含 break 关键字。
```python
while True:
    print('Please type your name.')
    name = input()
    if name == 'Veinin':
        break
print('Thank you!')
```

### continue 语句
像 break 语句一样， continue 语句用于循环内部。如果程序执行遇到 continue语句，就会马上跳回到循环开始处，重新对循环条件求值
```python
while True:
    print('Who are your?')
    name = input()
    if name != 'Veinin':
        continue
    print('Hello, Veinin. What is the password?')
    password = input()
    if password == '123':
        break
print('Access granted.')
```

### for 循环和 range()函数
通过 for 循环语句和 range()函数来实现一个代码块执行固定次数。
for 语句看起来像 for i in range(5):这样， 总是包含以下部分：
- for 关键字；
- 一个变量名；
- in 关键字；
- 调用 range()方法，最多传入 3 个参数；
- 冒号；
- 从下一行开始，缩退的代码块（称为 for 子句）。
```python
print('My name is')
for i in range(5):
    print('Jimmy Five Time (' + str(i) + ')')
    
total = 0
for num in range(100):
    total = total + num
print(total)
```

### range()的开始、 停止和步长参数
下列代码 `range` 函数中，第一个参数是 for 循环变量开始的值， 第二个参数是上限， 但不包含它， 也就是循环停止的数字。第三个参数是“步长”。 步长是每次迭代后循环变量增加的值。
```python
for i in range(0, 10, 2):
    print(i)
```

### 导入模块
Python 程序可以调用一组基本的函数， 这称为“内建函数”， 包括你见到过的print()、 input()和 len()函数。 Python 也包括一组模块，称为“标准库”。每个模块都是一个 Python 程序， 包含一组相关的函数， 可以嵌入你的程序之中。例如， math模块有数学运算相关的函数， random 模块有随机数相关的函数， 等等。
在代码中，

import 语句包含以下部分：
- import 关键字；
- 模块的名称；
- 可选的更多模块名称，之间用逗号隔开。
```python
import random
for i in range(5):
    print(random.randint(1, 10))
```

#### 使用逗号分隔符来导入多个模块
```python
import random, sys, os, math
```

#### from import 语句 
import 语句的另一种形式包括 from 关键字，之后是模块名称， import 关键字和
一个星号， 例如 `from random import *` 。 
使用这种形式的 import 语句，调用 random模块中的函数时不需要 random.前缀。
但是， 使用完整的名称会让代码更可读， 所以最好是使用普通形式的 import 语句。

### 用 sys.exit()提前结束程序
通过调用 sys.exit()函数， 可以让程序终止或退出。因为这个函数在 sys 模块中，所以必须先导入 sys， 才能使用它。
```python
import sys

while True:
    print('Type exit to exit.')
    response = input()
    if response == 'exit':
        sys.exit()
    print('You typed ' + response + '.')
```
