---
title: Lua入门教程：模块与包
date: 2018-12-20 20:34:15
categories: Lua入门教程
tags: [Lua 5.3, Lua编程]
---

Lua 从 5.1 版本开始为模块与包定义了一系列规则，这些规则不需要引入额外的功能特性。对用户来说，一个模块就是一些代码，这些代码可以通过 require 函数加载。

值得注意的是，从 Lua 5.2 开始编写模块的建议方式已经发生改变，而不在是 Lua 5.1 中的 module("mymodule", package.seall) 。现在根据推荐的是创建一个本地表，将所有模块函数放入其中并返回表，其最大的区别是不会再使用全局命名空间来注册模块。

## 模块的基本方法

定义一个简单的模块，该模块在文件 `test_module.lua` 中，其模块有两个函数 `foo` 和 `bar`：

``` lua
-- test_module.lua

local M = {}

function M.foo()
    print("foo")
end

function M.bar()
    print("bar")
end

return M
```

<!--more-->

另外，还有一种编写模块的方法是把所有函数定义为局部变量，然后在最后构造模块并返回一个表：

``` lua
-- test_module.lua

local function foo()
    print("foo")
end

local function bar()
    print("bar")
end

return {
    foo = foo,
    bar = bar,
}
```

这种编写模块的方式与上一种运行结果是一样的，其优点是无需在每个函数前使用表示符 M. 或类似的这种东西，最后再显示的导出相应的表。但这种方式有一个缺点就是导出表处在模块的最后，我们需要把导出的名字都写两边，会有点冗余。

当然，不管怎么，最后我们都能通过 require 函数获取到模块并使用它：

``` lua
-- module_main.lua

local tm = require "test_module.lua"

tm.foo() --> foo
tm.bar() --> bar
```

## require 函数

require 只是一个简单的加载模块的函数，我们如果需要使用某个模块，我们只需向其传入模块名作为参数，就可以获取指定的模块代码。 require 使用方式通常有两种：

```lua
-- 使用括号
local m = require("test_module")

-- 不使用括号
local m = require "tet_module"
```

这两种方式并没有什么差异性，它们的运行方式是一样的。但对于 require 执行方式，我们还是有必要了解以下其运行原理的：

- 首先 rquire 函数调用时，会去一个叫做 package.loaded 的全局表中查找模块是否已经加载，如果已经加载则返回对于的值。
- 如果 package.loaded 没有找到对于的模块名，则会进一步搜索具有自定模块名的 Lua 文件（搜索路径由 package.path 决定），如果找打对于文件则调用 loadfile 函数加载。
- 进一步，如果找不到指定模块名的 Lua 文件，那么会继续搜索对于名称的的 C 标准库文件（搜索路径由 package.cpath 决定），如果找到 C 标准库，则会使用 package.loadlib 函数进行加载。
- 最后，如果找到模块由返回值，那么 require 会返回这个值，并保存在表 package.loaded 中，以便下次加载时返回相同的值。如果找到的模块没有任何返回值，返回值会设置为 true。

因为由 package.loaded 的存在，除了第一次加载会执行一系列搜索模块规则外，后续的调用都会直接返回 package.loaded 中的值，如果我们希望再次强制加载一此模块，以达到比如重新更新模块的代码，我们可以这样做：

``` lua
package.loaded.modname = nil
local m = require "modname"
```

## require 搜索路径

require 使用的路径是一组模板，其中每一项都指定了如何将模块名转换为文件名的方式，其中每一个模板都是一个包含了可选问好的文件名， require 会将传入的模块名替换每一个问号，然后再一次检查每一项是否存在该文件名，如果目标不存在，则检查下一个模板。我们可以看下面这组路径：

``` lua
D:\Lua\bin\lua\?;D:\Lua\bin\lua\?\?.lua;
```

如果使用 require "test" 加载模块，则会将上面路径转换为：

``` lua
D:\Lua\bin\lua\test
D:\Lua\bin\lua\test\test.lua
```

前面提到了 require 用于搜索 Lua 文件路径的变量是 package.path，其默认值初始化后会设置为环境变量 `LUA_PATH_5_3` 的值，如果没有 `LUA_PATH_5_3` 这个环境变量则会尝试找到 `LUA_PATH` 这个变量进行设置。

对于 C 标准库的路径其用变量名叫做 package.cpath，其默认值来自环境变量 `LUA_CPATH_5_3` 或者 `LUA_CPATH`。需要注意的是，package.cpath 的值在 POSIX 系统和 Windows 系统中是由差异的，比如在 POSIX 系统中，其典型的值为：

``` lua
./?.so;/usr/local/lib/lua/5.3/?.so
```

可以看到 POSIX 系统中，库名称都是 .so 结尾，而在 Windows 中会变成 .dll：

``` lua
.\?.dll;D:\Lua\lua53\lib\?dll
```

另外，我们还需要了解函数 `package.searchpath(name, path)` 函数，该函数实现了搜索库的所有规则，我们可以传入一个模块名和路径来搜索文件，该函数会返回抵押给存在的文件，如果不存在则会返回 nil 和无法加载成功的错误信息。

``` lua
> path = ".\\?.dll;D:\\lua\\lib\\?.dll"
> print(package.searchpath("test", path))
nil
        no file '.\test.dll'
        no file 'D:\lua\lib\test.dll'
```

## 加载器

加载器的作用是用来搜索 Lua 文件或者 C 标准库文件。在变量数组 `package.searchers` 中存放了 require 所需要使用的所有类型的加载器，当查找一个模块是，require 会按次序使用这些加载器，并传入模块名进行搜索。如果某个加载器找到模块，则停止搜索并返回，如果使用完所有加载器都没有找到模块，则 require 函数会抛出异常。

Lua 用四个查找器函数初始化 `package.searchers` 表：

- 第一个查找器就是简单的在 package.preload 表中查找加载器。
- 第二个查找器用于查找 Lua 库文件，它使用变量 package.path 中的路径来做查找工作。
- 第三个查找器用于查找 C 库文件，它使用变量 package.cpath 中的路径来做查找工作。
- 第四个查找器我们称之为一体化加载器，它从 C 库路径中查找指定模块的根名字，比如请求 a.b.c　时，它将查找 a 这个 C 库，如果找到，再加载函数，这个例子中则是 `luaopen_a_b_c`。利用这个机制，可以把若干 C 子模块打包进单个库，每个子模块都可以有原本的加载函数名。

除了上面所说的加载器之外，我们也可以为一些特殊模块定义预加载器，预加载查找器会使用要给名为 `package.preload` 的表来映射模块名称和加载函数，当加载指定模块时，如果检查到 `package.preload` 定义了预加载器函数，则会直接使用该函数作为模块加载，并使用其返回值返回。

## 子模块与包

在一个负责的软件系统中，其一般会包含多个软件模块，每个模块又会分配多个子模块，而 Lua 是支持这种具有层次结构的模块分配的。比如，一个名为 `mod.sub` 的模块是模块 `mod` 的一个子模块 `sub`，我们可以看下面这样的树形结构的模块示意：

``` shell
src
    mod
        sub
    mod2
        sub2
        sub3
```

当搜索一个子模块时，函数 require 会将点转换为对于操作系统中的分隔符，如 POSIX 操作系统中的斜杠或 Windows 操作系统中的反斜杠。转换完成后，再对搜索路径的问号进行替换，然后走上面介绍的的模块搜索流程。如下面路径：

``` shell
.\?.lua;D:\Lua\?.lua;
```

当调用 `require "a.b"` 时，会生成以下搜索路径：

``` shell
.\a\b.lua
D:\Lua\a\b.lua;
D:\Lua\a\b\init.lua;
```

这种搜索行为可以让我们很容易的把一个模块中的所有文件放置到同一个目录下，比如模块 m、m.a、m.b 对于的文件可以分别时 m/init.lua、m/a.lua、m/b.lua，而目录 m 可以放置在合适的任意位置，其他模块搜索只需要设置其搜索路径即可。

另外，C 语言中名称是不能包含点，因此在编写 C 标准库时，子模块 a.b 是无法导出函数 lua_a.b的，这时，require 函数会将点替换为下划线，即名为 lua_a_b 的 C 标准库函数。