---
title: Python 快速上手 - 字符串
date: 2018-03-18 16:46:00
categories: Python
tags: [Python, 编程]
---

文本是程序需要处理的最常见的数据形式。涉及字符串的操作不仅仅是使用+操作符连接两个字符串，你还可以从字符串中提取部分字符串， 添加或删除空白字符， 将字母转换成小写或大写， 检查字符串的格式是否正确。你甚至可以编写 Python 代码访问剪贴板， 复制或粘贴文本。

## 处理字符串

### 双引号
在Python中构建一个字符串相当简单：以单引号开始和结束。
如果字符串里面也使用单引号的话，则需要使用双引号开始和结束来表示一个字符串。

```python
>>> s = "I found hi's very selfish."
```

<!--more-->

### 转义字符
“转义字符” 让你输入一些字符，它们用其他方式是不可能放在字符串里的。转义字符包含一个倒斜杠（\）， 紧跟着是想要添加到字符串中的字符。
常用的转移字符包括:\'(单引号)、 \"(双引号)、 \t(制表符)、 \\(倒斜杠)

```python
>>> s = 'I found hi\'s very selfish.'
>>> s
"I found hi's very selfish."
```

### 原始字符串
在字符串开始的引号之前加上 r， 那么它就成为了一个原始字符串。“原始字符串” 会完全忽略所有的转义字符， 打印出字符串中所有的倒斜杠。

```python
print(r'That is Carol\'s cat.')
```

### 用三重引号的多行字符串
虽然可以用\n转义字符将换行放入一个字符串，但使用多行字符串通常更容易。
在 Python 中，多行字符串的起止是 3 个单引号或 3 个双引号。“三重引号” 之间的所有引号、 制表符或换行， 都被认为是字符串的一部分。 Python 的代码块缩进规则不适用于多行字符串。

```python
print('''Dear Alice,

Eve's cat has been arrested for catnapping.

Sincerely,
Bob''')
```

### 多行注释
虽然井号字符（#） 表示这一行是注释， 但多行字符串常常用作多行注释。

```python
"""This is a test Python program.
Written by Al Sweigart al@inventwithpython.com
This program was designed for Python 3, not Python 2.
"""


def spam():
    """This is a multiline comment to help
    explain what the spam() function does."""
    print('Hello!')
```

### 字符串下标和切片
字符串像列表一样，可以使用下标和切片。
字符串切片并不能修改原来的字符串。但可以从一个变量中获取切片，记录在另一个变量中。

```python
>>> text = 'Hello world!'
>>> text[1]
'e'
>>> text[-2]
'd'
>>> text[0:4]
'Hell'
>>> text[:5]
'Hello'
>>> text[4:]
'o world!'
>>> text2 = text[0:5]
'Hello'
```

### 字符串使用 in 和 not in 操作符
像列表一样， in 和 not in 操作符也可以用于字符串。用 in 或 not in 连接两个字符串得到的表达式， 将求值为布尔值 True 或 False。

```python
>>> 'Veinin' in 'Veinin Guo'
True
>>> 'Veinin' not in 'Veinin Guo'
False
>>> '' in 'Veinin Guo'
True
```

## 字符串操作方法
某些字符串需要转换、分析然后产生新的字符串，字符串内置了一些常用的方法。
**注意**：Python 中所有字符串操作方法并不会改变字符串本身的属性，而是返回一个操作后的新字符串。

### upper()、 lower()、 isupper()和 islower()
upper()和 lower() 方法会返回一个新字符串，所有字母都被相应地转换为大写或小写。

```phton
>>> text = 'Hello World!'
>>> print(text.upper())
HELLO WORLD!
>>> print(text.lower())
hello world!
```

isupper()和islower()方法用来判断字符串是否至少有要给字母，并且所有字母都是大写或小写，相应地如果成立就会返回布尔值 True，否则返回 False。

```python
>>> 'Hello world'.isupper()
False
>>> 'HELLO WORLD'.isupper()
True
>>> 'Hello'.islower()
False
>>> 'hello'.islower()
True
```

### isX 方法
为了能判断字符串的特点，提供了一些常用的以 `is` 开头的方法。
- isalpha() 如果字符串非空，且只包含字母，则返回 True
- isalnum() 如果字符串非空，且只包含字母和数字，则返回 True
- isdecimal() 如果字符串非空，且只包含数字，则返回 True
- isspace() 如果字符串非空，且只包含空格、换行和制表符，则返回 True
- istitle() 如果字符串包含以大写字母开头且后面字母都是小写字母的单词，则返回 True

```python
>>> 'abc'.isalpha()
True
>>> 'abc123'.isalpha()
False
>>>
>>> 'abc123'.isalnum()
True
>>>
>>> 'abc123'.isdecimal()
False
>>> '10088'.isdecimal()
True
>>>
>>> '  '.isspace()
True
>>>
>>> 'I Am From China'.istitle()
True
>>> 'I am from China'.istitle()
False
```

### startswith() 和 endswith()
startswith() 和 endswith() 用来判断某个字符串以某个字符串开始或结束，成立则返回 True。

```python
>>> 'Hello World'.startswith('Hello')
True
>>> 'Hello World'.startswith('nihao')
False
>>> 'Hello World'.endswith('World')
True
```

### join() 和 split()
join() 方法用来将一个字符串列表中的每个字符串连接成一个新的字符串。
而 split() 方法与 join() 方法刚好相反，它会将一个字符串按制定分隔符进行分割，返回一个分割后的列表。

```python
>>> ', '.join(['java', 'python', 'golang'])
'java, python, golang'
>>>
>>> 'I am from China'.split()
['I', 'am', 'from', 'China']
>>>
>>> 'ABCDEFGEFGEFG'.split('E')
['ABCD', 'FG', 'FG', 'FG']
```

### 使用 rjust()、 ljust() 和 center() 方法对齐文本
rjust() 和 ljust() 方法使用向左或向右插入空格的方式返回一个字符串的填充版本。而 center() 方法则是让字符串文本居中。
上面三个方法都接受两个参数，第一个参数指定填充数量。第二个参数，指定填充的字符，默认是填充空格。

```python
>>> 'Veinin'.ljust(20, '-')
'Veinin--------------'
>>> 'Veinin'.rjust(20, '-')
'--------------Veinin'
>>> 'Veinin'.center(20)
'       Veinin       '
>>> 'Veinin'.center(20, '#')
'#######Veinin#######'
```

### 使用 strip()、 rstrip() 和 lstrip() 方法删除空白字符
strip()、 rstrip() 和 lstrip() 三个方法分别对一个字符串的两边、右边和左边的空白字符进行删除操作。

```python
>>> name = '   Veinin Guo   '
>>> name.strip()
'Veinin Guo'
>>> name.rstrip()
'   Veinin Guo'
>>> name.lstrip()
'Veinin Guo   '
```