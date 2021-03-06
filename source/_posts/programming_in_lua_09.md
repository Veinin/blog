---
title: Lua入门教程：编译、执行和错误
date: 2018-12-17 21:13:20
categories: Lua入门教程
tags: [Lua 5.3, Lua编程]
---

## dofile 与 loadfile 函数

dofile 是一个辅助函数，函数 loadfile 才完成了真正的核心工作。 两个函数都是从文件中加载 Lua 代码，但它不会运行，只是编译代码，然后把编译后的代码作为函数返回。 
与 dofile 不同，loadfile 不会抛出异常，只会返回错误码。

``` lua
function dofile(filename)
    local f = assert(loadfile(filename))
    return f()
end
```

## load 函数

与 loadfile 类似，但该函数是从一个字符串中读取代码。

编写用后即弃的代码：

``` lua
i = 0
s = "i = i + 1"
load(s)() --> 1
```

<!--more-->

该函数加载的代码如果有语法错误，load 会返回 nil 和 错误信息，所以最好使用 assert：

```lua
assert(load(s))()
```

laod 加载编译时不会设计词法定界，该函数总是在全局环境中编译代码：

```lua
i = 32
local i = 0
f = load("i = i + 1; print(i)")
g = function() i = i + 1; print(i) end
f() --> 33
g() --> 1
```

## 错误

Lua 会在遇到非预期的情绪时引发错误，如将非数值类型相加，对不是函数的值进行调用等。

也可以调用函数 error 并传入一个错误信息来作为参数引发一个错误：

``` lua
n = io.read("n")
if not n then error("invalid input") end
```

当然上面这种情况更简单的代码结构是使用 assert 函数来完成，assert 函数检查第一个参数是否为真，如果为真则返回该参数，否则引发一个错误，并用第二个参数作为错误提示信息：

``` lua
n = assert(io.read("n"), "invalid input")
```

Lua 中要给函数发现某个错误是，在进行异常处理时有两种选择：

- 返回错误代码（nil 或是 false）
- 通过函数 error 引发一个错误。

通常容易避免的错误应该引发错误，否则应该返回错误码。

## 错误处理和异常

要执行一段代码并捕获一段错误(try-catch)可以使用pcall函数，该函数以一种保护模式调用它的第一个参数，如果没有发生错误会返回true和调用函数的返回参数，否则返回false和错误信息。
我们可以通过 error(throw an exception) 来抛出异常，并通过 pcall 来捕获 (catch) 异常。

``` lua
local ok, msg = pcall(function()
    some code
    if unexpected_condition then error() end
    some code
end)

if ok then
    regular code
else
    error-handing code
end
```

另外，error 函数还有第2个可选参数 level，用于只想函数调用层次中的哪层函数报告错误，以说明谁应该为错误负责。

``` lua
function foo(str)
    if type(str) ~= "string" then
        error("string expected", 2)
    end
    regualr code
end
```

最后，我们如果希望错误发生时获得更多的调试信息，比如发生错误时一个完整的函数调用栈信息。
pcall 如果发生错误时，部分的调用栈已经被破坏了（从pcall到出错部分），如果希望得到一个完整的有意义的栈回溯，则必须在函数 pcall 返回前先将调用栈构好。
为了实现上面的需求，我们可以使用 xpcall 函数，该函数第二个参数是一个消息处理函数，当错误发生时，Lua 会在调用栈展开前调用这个消息处理函数，我们可以使用 debug.traceback 来获取更多错误信息。

``` lua
local function error_handler(msg)
    print(debug.traceback(tostring(msg), 2))
end

xpcall(function()
    some code
    if unexpected_c  ondition then error() end
    some code
end, error_handler)
```