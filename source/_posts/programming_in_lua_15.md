---
title: Lua入门教程：协程
date: 2019-01-03 23:10:10
categories: Lua入门教程
tags: [Lua 5.3, Lua编程, Lua协程]
---

从多线程的角度来看，协程（coroutine）与线程（thread）类似：协程时一系列的可执行语句，拥有自己的栈、局部变量和指令指针，同时协程又与其他协程共享全局变量和其他几乎一切资源。

协程与线程的主要区别在于，一个多线程程序可以并行运行多个线程，而协程则需要彼此协作运行，即在任意时刻只能有一个协程运行，且只有当正在运行的协程要求挂起时其执行才会暂停。

## 协程基础

与协程相关的函数都放在表 coroutine 中。没个新建立的协程都拥有一个完整的生命周期，其包含四种状态：挂起（suspended）、运行（running）、正常（normal）和死亡（dead)。

### 创建协程

coroutine.create(f)，创建一个以执行代码的函数 f（协程体）的新协程，并返回以类型 `thread` 为标志的新协程。

``` lua
co = coroutine.create(function() print("hello") end)
print(type(co)) --> thread
```

当协程被创建时，默认处于**挂起状态**，即协程不会在被创建时自动运行，我们可以通过 `coroutine.status` 来检查协程的状态：

``` lua
print(coroutine.status(co)) --> suspended
```

<!--more-->

### 运行协程

要想启动或再启动一个协程，我们可以使用函数 `coroutine.resume` 来执行，并将去状态改为运行：

``` lua
coroutine.resume(co) --> hello
```

上面的协程创建后运行的函数会输出 `hello` 然后该协程就终止了，最后协程执行完成后会变为**死亡状态**：

``` lua
print(coroutine.status(co)) --> dead
```

### 挂起协程

协程的强大之处在于函数 `coroutine.yield`，该函数可以让一个运行中的协程挂起自己，然后在后续执行中恢复运行：

``` lua
co = coroutine.create(function()
    for i = 1, 10 do
        print("co", i)
        coroutine.yield()
    end
end)
```

上面协程运行着一个循环，在循环中打印输出一个数字后遇到 `yield` 进入挂起状态：

``` lua
coroutine.resume(co) --> co 1
```

此时我们可以查看协程状态，会发现协程处于挂起状态：

``` lua
print(coroutine.status(co)) --> suspended
```

我们可以继续运行，直到上面 for 循环执行结束：

``` lua
coroutine.resume(co) --> co 2
coroutine.resume(co) --> co 3
    ...
coroutine.resume(co) --> co 10
coroutine.resume(co) --> 不输出任何数据
```

当最后一次调用 resume 时，协程体执行完毕并返回，不会输出任何数据，如果我们试图再次唤醒它，函数 resume 会返回 false 已经一条错误信息：

``` lua
print(coroutine.resume(co)) --> false  cannot resume dead coroutine
```

请注意，resume 函数运行在保护模式中，如果协程出错，Lua 不会显示错误信息，而是将错误信息返回给 resume 函数。

当协程 A 唤醒协程 B 时，协程 A 不是挂起状态（因为不能唤醒协程A），而不是运行状态（因为正在运行协程B），此时的状态称之为**正常状态**。

### 交换数据

协程中一个非常有用的机制是通过一对 `resume-yield` 来交换数据。第一个 `resume` 函数会把所有的额外参数传递给协程的主函数：

``` lua
co = coroutine.create(function(a, b, c)
        print("co", a, b, c)
    end)

coroutine.resume(co, 1, 2, 3) --> co 1 2 3
```

函数 `resume` 第一个返回值为 `true` 时表示没有错误，之后的返回值对应函数 `yield` 的参数：

``` lua
co = coroutine.create(function(a, b)
        coroutine.yield(a + b, a - b)
    end)

print(coroutine.resume(co, 10, 20)) --> true  30  -10
```

函数 `yield` 返回值则会对应 `resume` 的参数：

```lua
co = coroutine.create(function(x)
        print("co1", x)
        print("co2", coroutine.yield())
    end)

coroutine.resume(co, "hi") --> co1    hi
coroutine.resume(co, 4, 5) --> co2    4    5
```

当协程运行结束后，主函数所返回的值将变成对应函数 `resume` 的返回值：

``` lua
co = coroutine.create(function()
        return 1, 2
    end)

print(coroutine.resume(co)) --> true    1    2
```

## 使用协程解决生产者于消费者问题

生产者-消费者设计两个函数，一个函数不断的生成值，而另外一个函数则不断的消费这些值，我们可以看以下代码实现：

``` lua
function producer()
    while true do
        local x = io.read() -- 生产新值
        send(x)             -- 发送给消费者
    end
end

function send(x)
    coroutine.yield(x)
end

function consume()
    while true do
        local x = receive() -- 接受来自生产者的值
        io.write(x, "\n")   -- 消费
    end
end

function receive()
    local status, value = coroutine.resume(producer)
    return value
end

producer = coroutine.create(producer) -- 生成者在协程里面运行
consume() -- 通过消费者启动
```

上面这种设计中，程序通过消费者启动。当消费者需要新值时就唤醒生成者，生成者向消费者返回新值后挂起，知道消费者再次将其唤醒。因此这种设计称之为消费者驱动式设计。