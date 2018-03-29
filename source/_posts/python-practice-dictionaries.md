---
title: Python 快速上手 - 字典
date: 2018-03-18 16:10:00
categories: Python
tags: [Python, 编程]
---

字典数据类型提供了一种灵活的访问和组织数据的方式。
像列表一样，“字典”是许多值的集合。但不像列表的下标， 字典的索引可以使用许多不同数据类型， 不只是整数。 字典的索引被称为“键”，键及其关联的值称为“键-值”对。
字典仍然可以用整数值作为键， 就像列表使用整数值作为下标一样， 但它们不必从 0 开始， 可以是任何数字。

```python
>>> myCat = {'size': 'fat', 'color': 'gray', 'disposition': 'loud'}
>>> myCat['size']
'fat'
>>> 'My cat has ' + myCat['color'] + ' fur.'
'My cat has gray fur.'
>>>
>>> ticket = {12306: 'websites', 123456: 'phone number'}
>>> ticket[12306]
'websites'
>>> ticket[123456]
'phone number'
```

<!--more-->

## 字典与列表
确定两个列表是否相同时， 表项的顺序很重要。
字典不像列表，字典中的表项是不排序的，键-值对输入的顺序并不重要。
因为字典是不排序的， 所以不能像列表那样切片。
尽管字典是不排序的，但可以用任意值作为键，这一点让你能够用强大的方式来组织数据。
尝试访问字典中不存在的键， 将导致 KeyError 出错信息。

```python
>>> a = ['cats', 'dogs']
>>> b = ['dogs', 'cats']
>>> a == b
False
>>>
>>> c = {'firstName': 'Veinin', 'age': 25}
>>> d = {'age': 25, 'firstName': 'Veinin'}
>>> c == d
True
>>>
>>> d['lastName']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'lastName'
```

## keys()、 values()和 items()方法
有 3 个字典方法，它们将返回类似列表的值，分别对应于字典的键、值和键-值对：keys()、 values()和 items()。
这些方法返回的值不是真正的列表，它们不能被修改，没有append()方法。但这些数据类型（分别是 dict_keys、 dict_values 和 dict_items）可以用于for 循环。

```python
>>> person = {'firstName': 'Veinin', 'lastName': 'Guo', 'age': 18}
>>> for k in person.keys():
        print(k)

firstName
lastName
age
>>>
>>> for v in person.values():
        print(v)

Veinin
Guo
18
>>>
>>> for k, v in person.items():
        print(k, v)

firstName Veinin
lastName Guo
age 18
```

## 检查字典中是否存在键或值
in 和 not in 操作符可以检查值是否存在于列表中。也可以利用这些操作符，检查某个键或值是否存在于字典中。

```python
>>> person = {'firstName': 'Veinin', 'lastName': 'Guo', 'age': 18}
>>> 'name' in person
False
>>> 'sex' not in person
True
```

## get()方法
在访问一个键的值之前，检查该键是否存在于字典中，这很麻烦。好在，字典有一个 get()方法，它有两个参数：要取得其值的键，以及如果该键不存在时，返回的备用值。

```python
>>> person = {'firstName': 'Veinin', 'lastName': 'Guo', 'age': 18}
>>> "I'm " + str(person.get('age', 0)) + '.'
"I'm 18."
>>>
>>> 'His height is ' + str(person.get('height', 188)) + 'cm.'
'His height is 188cm.'
```

## setdefault()方法
你常常需要为字典中某个键设置一个默认值，当该键没有任何值时使用它。
setdefault()方法的第一个参数，是要检查的键。第二个参数，是如果该键不存在时要设置的值。

```python
>>> person = {'firstName': 'Veinin', 'lastName': 'Guo', 'age': 18}
>>> person.setdefault('height', 180)
180
>>> print('His height is ' + str(person.get('height', 188)) + 'cm.')
His height is 180cm.
```

