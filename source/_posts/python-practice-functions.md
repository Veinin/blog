---
title: Python 快速上手 - 函数
date: 2018-03-18 15:29:00
categories: Python
tags: [Python, 编程]
---

Python 提供了这样一些内建函数，但你也可以编写自己的函数。“函数”就像一个程序内的小程序。

## 使用 def 语句定义一个函数
```python
def hello():
    print('Veinin')
    print('Veinin Guo')
    print('Hello trere.')

for i in range(3):
    hello()
    print('')
```

<!--more-->

## 函数参数
定义一个函数时可以自己定义接收参数，传入的参数值，放在函数的括号之间。
```python
def hello(name):
    print('Hello ' + name)

hello('Veinin')
hello('Jalin')
```

## 返回值和 return 语句
函数调用求值的结果， 称为函数的“返回值”。
用 def 语句创建函数时， 可以用 return 语句指定应该返回什么值。 return 语句包含以下部分：
- return 关键字；
- 函数应该返回的值或表达式。
```python
import random

def getAnswer(answerNumber):
    if answerNumber == 1:
        return 'It is certain'
    elif answerNumber == 2:
        return 'It is decidedly so'
    elif answerNumber == 3:
        return 'Yes'
    else:
        return 'No'


r = random.randint(1, 4)
fortune = getAnswer(r)
print(fortune)
```

## None 值
在 Python 中有一个值称为 None，它表示没有值。 None 是 NoneType 数据类型的唯一值。
像布尔值 True和 False 一样， None 必须大写首字母 N。
```python
spam = None
print(None == spam)

spam = 'Hello'
print(None == spam)
```

## 关键字参数和 print()
print()函数有可选的变元 end 和 sep， 分别指定在参数末尾打印什么，以及在参数之间打印什么来隔开它们。
默认情况下，print()函数自动在传入的字符串末尾添加了换行符。
可以设置 end 关键字参数，将它变成另一个字符串。例如，如果程序像这样：
```python
print('Hello', end='')
print('World')
```

print()传入多个字符串值时，该函数就会自动用一个空格分隔它们。可以传入 sep 关键字参数， 替换掉默认的分隔字符串。
```python
>>> print('cats', 'dogs', 'mice')
cats dogs mice

>>> print('Red', 'Green', 'Blue')
Red Green Blue
>>> print('Red', 'Green', 'Blue', sep=',')
Red,Green,Blue
```

## 局部和全局作用域
在被调用函数内赋值的变元和变量，处于该函数的“局部作用域”。在所有函数之外赋值的变量，属于“全局作用域”。
处于局部作用域的变量，被称为“局部变量”。处于全局作用域的变量，被称为“全局变量”。

作用域很重要， 理由如下：
- 全局作用域中的代码不能使用任何局部变量
```python
def spam():
    eggs = 31337

spam()
print(eggs) # NameError: name 'eggs' is not defined
```

- 局部作用域可以访问全局变量
```python
name = 'Veinin'
def hello():
    print('Hello, ' + name)

hello()
```

- 一个函数的局部作用域中的代码，不能使用其他局部作用域中的变量
```python
def spam():
    eggs = 99
    bacon()
    print(eggs)

def bacon():
    ham = 101
    print(eggs) # NameError: name 'eggs' is not defined

spam()
```

- 如果在不同的作用域中，你可以用相同的名字命名不同的变量。也就是说，可
以有一个名为 spam 的局部变量，和一个名为 spam 的全局变量。
```python
def spam():
    eggs = 'spam local'
    print(eggs) # prints 'spam local'

eggs = 'global'
spam()
print(eggs) # prints 'global'
```

## global 语句
如果需要在一个函数内修改全局变量， 就使用 global 语句。它就告诉 Python，在这个函数中， 某个值指的是全局变量， 所以不要用这个名字创建一个局部变量。
```python
def spam():
    global eggs
    eggs = 'spam'

eggs = 'global'
spam()
print(eggs)
```

有 4 条法则， 来区分一个变量是处于局部作用域还是全局作用域：
- 1．如果变量在全局作用域中使用（即在所有函数之外），它就总是全局变量。
- 2．如果在一个函数中，有针对该变量的 global 语句，它就是全局变量。
- 3．否则，如果该变量用于函数中的赋值语句，它就是局部变量。
- 4．但是，如果该变量没有用在赋值语句中，它就是全局变量。
```python
def spam():
    global eggs
    eggs = 'spam' # this is the global
    
def bacon():
    eggs = 'bacon' # this is the local
    
def ham():
    print(eggs) # this is the global

eggs = 42 # this is the global
spam()
print(eggs)
```

## 异常处理
在 Python 程序中遇到错误， 或“异常”， 如果不处理，意味着整个程序崩溃。
而我们希望程序能检测错误， 处理它们，然后继续运行。

以下代码，当试图用一个数除以零时，就会发生 `ZeroDivisionError: division by zero` 错误提示。从而导致后面代码中断运行。
```python
def devide(divideBy):
    return 42 / divideBy

print(devide(2))
print(devide(0))
print(devide(22))
```

我们可以使用 try 和 except 语句来处理错误。那些可能出错的语句被放在 try 子句中。如果错误发生，程序执行就转到接下来的 except 子句开始处。
```python
def devide(divideBy):
    try:
        return 42 / divideBy
    except ZeroDivisionError:
        print('Error: Invalid argument.')

print(devide(2))
print(devide(0))
print(devide(22))
```