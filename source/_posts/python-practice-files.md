---
title: Python 快速上手 - 文件
date: 2018-04-19 21:35:00
categories: Python
tags: [Python, 编程, Python 文件]
---

## os.path 模块
os.path 模块包含了许多与文件名和文件路径相关的有用函数。
例如， 你已经使用了 os.path.join()来构建所有操作系统上都有效的路径。
因为 os.path 是 os 模块中的模块， 所以只要执行 import os 就可以导入它。

```
import os
```

## 文件与文件路径
在 Windows 上， 路径书写使用倒斜杠作为文件夹之间的分隔符。但在 OS X 和Linux 上， 使用正斜杠作为它们的路径分隔符。
如果想要程序运行在所有操作系统上，在编写 Python 脚本时， 就必须处理这两种情况。
Python 使用 `os.path.join()` 函数来做这件事很简单。
如果将单个文件和路径上的文件夹名称的字符串传递给它， `os.path.join()` 就会返回一个文件路径的字符串

<!--more-->

```
>>> import os
>>> os.path.join('usr', 'bin', 'spam')
'usr\\bin\\spam'
>>>
>>> myFiles = ['a.txt', 'b.txt', 'c.txt']
>>> for fileName in myFiles:
        print(os.path.join('C:\\Users\\App', fileName))

C:\Users\App\a.txt
C:\Users\App\b.txt
C:\Users\App\c.txt
```

## 当前工作目录
每个运行在计算机上的程序， 都有一个“ 当前工作目录”， 或 cwd。
利用 `os.getcwd()` 函数，可以取得当前工作路径的字符串， 并可以利用 `os.chdir()` 改变它。

```
>>> os.getcwd()
'C:\\Users\\Ansh'
>>> os.chdir('C:\\App')
>>> os.getcwd()
'C:\\App'
```

## 创建新文件夹
可以用 `os.makedirs()` 函数创建新文件夹（ 目录）。
```
os.makedirs('C:\\Users\\Ansh\\Desktop\\Test')
```

## 处理绝对路径和相对路径
有两种方法指定一个文件路径。
- “绝对路径”， 总是从根文件夹开始。
- “相对路径”，它相对于程序的当前工作目录。
还有点（ .）和点点（ ..）文件夹。它们不是真正的文件夹，而是可以在路径中使用的特殊名称。

os.path 模块提供了一些函数， 返回一个相对路径的绝对路径， 以及检查给定的路径是否为绝对路径。
- 调用 `os.path.abspath(path)` 将返回参数的绝对路径的字符串。
- 调用 `os.path.isabs(path)` ，如果参数是一个绝对路径，就返回 True，如果参数是一个相对路径，就返回 False。
- 调用 `os.path.relpath(path, start)` 将返回从 start 路径到 path 的相对路径的字符串。如果没有提供 start，就使用当前工作目录作为开始路径。
- 调用 `os.path.dirname(path)` 将返回一个字符串，它包含 path 参数中最后一个斜杠之前的所有内容。
- 调用 `os.path.basename(path)` 将返回一个字符串，它包含 path 参数中最后一个斜杠之后的所有内容。
- 调用 `os.path.split()` 获得一个路径的目录名称和基本名称， 会返回两个字符串的元组

```
>>> os.path.abspath('.')
'C:\\App'
>>> os.path.abspath('.\\Scripts')
'C:\\App\\Scripts'
>>> os.path.isabs('.')
False
>>> os.path.isabs(os.path.abspath('.'))
True
>>> os.path.relpath('C:\\Windows', 'C:\\')
'Windows'
>>> os.path.relpath('C:\\Windows', 'C:\\spam\\eggs')
'..\\..\\Windows'
>>>
>>> path = 'C:\\Windows\\System32\\calc.exe'
>>> os.path.basename(path)
'calc.exe'
>>> os.path.dirname(path)
'C:\\Windows\\System32'
>>> os.path.split(path)
('C:\\Windows\\System32', 'calc.exe')
>>> path.split(os.path.sep)
['C:', 'Windows', 'System32', 'calc.exe']
```

## 文件大小和文件夹内容
一旦有办法处理文件路径， 就可以开始搜集特定文件和文件夹的信息。
- 调用 `os.path.getsize(path)` 将返回 path 参数中文件的字节数。
- 调用 `os.listdir(path)` 将返回文件名字符串的列表，包含 path 参数中的每个文件。

```
>>> os.path.getsize('C:\\Windows\\System32\\calc.exe')
26112L
>>> os.listdir('C:\\App\\Python27')
['DLLs', 'Doc', 'include', 'Lib', 'libs', 'LICENSE.txt', 'NEWS.txt', 'python.exe', 'pythonw.exe', 'README.txt', 'Scripts', 'tcl', 'Tools', 'w9xpopen.exe']
```

## 检查路径有效性
如果你提供的路径不存在， 许多 Python 函数就会崩溃并报错。 `os.path` 模块提供了一些函数，用于检测给定的路径是否存在，以及它是文件还是文件夹。
- 调用 `os.path.exists(path)` ，返回所指的文件或文件夹是否存在。
- 调用 `os.path.isfile(path)` ，返回目标是否一个文件。
- 调用 `os.path.isdir(path)` ，返回目标是否一个文件夹。

## 打开并读写文件
在 Python 中， 读写文件有 3 个步骤：
- 调用 `open()` 函数， 返回一个 File 对象。
- 调用 File 对象的 `read()` 或 `write()` 方法。
- 调用 File 对象的 `close()` 方法，关闭该文件。

打开文件 `open()` 方法，“读模式”、“ 写模式” 和 “添加模式”。如果打开文件时用读模式，就不能写入文件。而写模式将覆写原有的文件。添加模式将在已有文件的末尾添加文本。
`open` 方法有第二个可选参数，如果不传则是读模式打开， 参数 'w' 将使用写模式打开，而参数 'a' 将以添加模式打开。

对于读取文件，还可以使用 `readlines()` 方法， 从该文件取得一个字符串的列表。

```
>>> helloFile = open('C:\\Users\\Ansh\\Desktop\\hello.txt')
>>> helloContent = helloFile.read()
>>> helloContent
'Hello, World!'
>>>
>>> textFile = open('C:\\Users\\Ansh\\Desktop\\text.txt')
>>> for line in textFile.readlines():
        print(line.strip())

When, in disgrace with fortune and men's eyes,
I all alone beweep my outcast state,
And trouble deaf heaven with my bootless cries,
And look upon myself and curse my fate,
>>>
>>> baconFile = open('bacon.txt', 'w')
>>> baconFile.write('Hello world!\n')
>>> baconFile.close()
>>>
>>> baconFile = open('bacon.txt', 'a')
>>> baconFile.write('Bacon is not a vegetable.')
>>> baconFile.close()
>>>
>>> baconFile = open('bacon.txt')
>>> print(baconFile.read())
Hello world!
Bacon is not a vegetable.
>>> baconFile.close()
```

## 使用 shelve 模块保存变量
你可以将 Python 程序中的变量保存到二进制的 shelf 文件中。这样， 程序就可以从硬盘中恢复变量的数据。 shelve 模块让你在程序中添加“ 保存”和“ 打开” 功能。

要利用 shelve 模块读写数据，首先要导入它。
调用函数 `shelve.open()` 并传入一个文件名，然后将返回的值保存在一个变量中。可以对这个变量的 shelf 值进行修改，就像它是一个字典一样。
当你完成时，在这个值上调用 close()。最后，在当前的工作目录下会生成一个对应的二进制文件。

```
shelfFile = shelve.open('mydata')
cats = ['Zophie', 'Pooka', 'Simon']
shelfFile['cats'] = cats
shelfFile.close()
```

然后，可以使用 shelve 模块， 重新打开这些文件并取出数据。
shelf 值不必用读模式或写模式打开，因为它们在打开后，既能读又能写。

```
shelfFile = shelve.open('mydata')
print(shelfFile['cats'])    # ['Zophie', 'Pooka', 'Simon']
shelfFile.close()
```

### shutil 模块
shutil（或称为 shell 工具）模块中包含一些函数，让你在 Python 程序中复制、移动、改名和删除文件。
要使用 shutil 的函数，首先需要 `import shutil`。

### 复制文件和文件夹
调用 `shutil.copy(source, destination)`，将路径 source 处的文件复制到路径 destination处的文件夹。
如果 destination 是一个文件名，它将作为被复制文件的新名字。该函数返回一个字符串，表示被复制文件的路径。
另外，可以使用 `shutil.copytree()` 复制整个文件夹里面包含的文件夹和文件。

```
import shutil, os

os.chdir('C:\\')
shutil.copy('C:\\Users\\Ansh\\Desktop\\hello.txt', 'C:\\temp')
shutil.copy('C:\\temp\\hello.txt', 'C:\\temp\\hello2.txt')

shutil.copytree('C:\\temp', 'C:\\Users\\Ansh\\Desktop\\temp_backup')
```

### 移动或改名文件和文件夹
调用 `shutil.move(source, destination)`， 将路径 source 处的文件夹移动到路径 destination。

```
shutil.move('C:\\temp', 'C:\\Users\\Ansh\\Desktop')
shutil.move('C:\\text.txt', 'C:\\Users\\Ansh\\Desktop\\temp')
```

### 永久删除文件和文件夹
利用 os 模块中的函数，可以删除一个文件或一个空文件夹。但利用 shutil 模块，可以删除一个文件夹及其所有的内容。
- 调用 `os.unlink(path)` 将删除 path 处的文件。
- 调用 `os.rmdir(path)` 将删除 path 处的文件夹。
- 调用 `shutil.rmtree(path)` 将删除 path 处的文件夹，它包含的所有文件和文件夹都会被删除。

```
for filename in os.listdir('C:\\Users\\Ansh\\Desktop'):
    if filename.endswith('.txt'):
        os.unlink(filename)

os.makedirs('Test')
os.rmdir('Test')

shutil.rmtree('temp_back')
```

## 遍历目录树
如果你需要对某个文件夹中的所有文件改名， 包括该文件夹中所有子文件夹中的所有文件。也就是说， 你希望遍历目录树， 处理遇到的每个文件。
写程序完成这件事，可能需要一些技巧。 好在， Python 提供了一个 `os.walk()` 函数， 替你处理这个过程。
`os.walk()` 函数被传入一个字符串值，即一个文件夹的路径。你可以在一个 for循环语句中使用 os.walk()函数，遍历目录树。

`os.walk()` 在循环的每次迭代中，返回 3 个值：
- 当前文件夹名称的字符串。
- 当前文件夹中子文件夹的字符串的列表。
- 当前文件夹中文件的字符串的列表。

```
import os

path = 'C:\\Users\\Ansh\\Desktop\\Source\\lua-5.3.4'

for folderName, subFolders, fileNames in os.walk(path):
    print('The current folder is ' + folderName)

    for subFolder in subFolders:
        print('Sub folder of ' + folderName + ' : ' + subFolder)

    for fileName in fileNames:
        print('File inside ' + folderName + " : " + fileName)
```

## 使用 zipfile 模块压缩文件
我们经常需要对一组文件进行压缩，减少它的大小，然后在网络上进行传输。ZIP 文件（ 带有.zip 文件扩展名）， 它可以包含许多其他文件的压缩内容。
在 Python 程序可以利用 `zipfile` 模块中的函数创建和打开（或解压） ZIP 文件。

### 创建和添加到 ZIP 文件
要创建你自己的压缩 ZIP 文件， 必须以“写模式”打开 ZipFile 对象，即传入'w'作为第二个参数。
如果向 ZipFile 对象的 `write()` 方法传入一个路径， Python 就会压缩该路径所指的文件， 将它加到 ZIP 文件中。
`write()` 方法的第一个参数是一个字符串， 代表要添加的文件名。第二个参数是“压缩类型”参数，它告诉计算机使用怎样的算法来压缩文件。

```
import zipfile

newZip = zipfile.ZipFile('new.zip', "w")
newZip.write('myCats.py', compress_type=zipfile.ZIP_DEFLATED)
newZip.close()
```

### 读取 ZIP 文件
可以使用 `zipfile.ZipFile()` 函数读取要给 ZIP 文件， 向它传入一个字符串， 表示.zip 文件的文件名。
ZipFile 对象有一个 `namelist()` 方法，返回 ZIP 文件中包含的所有文件和文件夹的字符串的列表。
可以把文件或文件夹名称传递给 ZipFile 对象的 `getinfo()` 方法来返回一个关于特定文件的 ZipInfo 对象。
通过 ZipInfo 对象可以读取到保存的归档文件的有用信息。

```
readZip = zipfile.ZipFile('new.zip')
print(readZip.namelist())

catsFileInfo = readZip.getinfo('myCats.py')
print(catsFileInfo.file_size)
print(catsFileInfo.compress_size)

readZip.close()
```

### 加压缩 ZIP 文件
ZipFile 对象的 `extractall()` 方法从 ZIP 文件中解压缩所有文件和文件夹， 放到当前工作目录中。
可以使用 `extractall()` 方法从 ZIP 文件中解压缩所有文件。
也可以使用 `extract()` 方法从 ZIP 文件中解压缩单个文件。

```
extractZip = zipfile.ZipFile('new.zip')
extractZip.extractall()
extractZip.extract('myCats.py', '.\\temp')
extractZip.close()
```