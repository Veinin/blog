---
title: Lua入门教程：元表与元方法
date: 2018-12-22 21:35:05
categories: Lua入门教程
tags: [Lua 5.3, Lua编程, 元表, 元方法]
---

Lua 语言中每中类型的值都有一套可预见的操作集合，比如可以将数字相加，将字符串连接，还可以在表中插入键值对。但我们却无法直接将两个表相加，无法对表进行直接比较，除非我们使用元表。
元表可以修改一个值在面对未知操作时的行为。例如，我们对两个表 a 和 b 执行 `a + b` 操作，Lua 在试图将两个表相加时，会检查其中某个表是否含有元表(metatable)，且元表中是否含有 `__add` 字段，如果 Lua 找到该字段，则调用该字段对应的值，这就是所有的元方法(metamethod)。

在元表中每个元方法的键的命名都是一个双下划线（__）加事件名的，键关联的那些值被称为元方法。上面说的 `_add` 就是元方法键名称，而对应的元方法值是执行加操作的函数。

## 获取与设置元表

在 Lua 中每个值都可以有元表，而元表只是一个普通的Lua表。
每个表和用户数据类型都具有各自独立的元表，而其他类型的值则共享对于类型所属的同一个元表。
我们可以使用 `getmetatable` 获取一个表的元表，注意，刚创建的新表是没有元表的：

``` lua
t = {}
print(getmetatable(t))  --> nil
```

另外，我们可以使用 `setmetatable` 来设置和修改任意表的元表：

``` lua
t1 = {}
setmetatable(t, t1)
print(getmetatable(t) == t1)    --> true
```

在 Lua 中我们只能为表设置元表，如果要为其他类型值设置元表，则必须通过C代码或调试库完成。另外，字符串库为所有字符串都设置了同一个元表，而其他类型默认是没有元表的：

``` lua
print(getmetatable("hello"))    --> table: 0106F578
print(getmetatable("world"))    --> table: 0106F578
print(getmetatable(true))       --> nil
```

<!--more-->

## 算术运算元方法

对于数学运算、位运算这些算术运算符，每一个操作都有唯一对应的元方法：

- **__add**: + 操作。 如果任何不是数字的值（包括不能转换为数字的字符串）做加法， Lua 就会尝试调用该元方法。
- **__sub**: - 操作。 行为和 "add" 操作类似。
- **__mul**: * 操作。 行为和 "add" 操作类似。
- **__div**: / 操作。 行为和 "add" 操作类似。
- **__mod**: % 操作。 行为和 "add" 操作类似。
- **__pow**: ^ （次方）操作。 行为和 "add" 操作类似。
- **__unm**: - （取负）操作。 行为和 "add" 操作类似。
- **__idiv**: // （向下取整除法）操作。 行为和 "add" 操作类似。
- **__band**: & （按位与）操作。 行为和 "add" 操作类似， 不同的是 Lua 会在任何一个操作数无法转换为整数时尝试取元方法。
- **__bor**: | （按位或）操作。 行为和 "band" 操作类似。
- **__bxor**: ~ （按位异或）操作。 行为和 "band" 操作类似。
- **__bnot**: ~ （按位非）操作。 行为和 "band" 操作类似。
- **__shl**: << （左移）操作。 行为和 "band" 操作类似。
- **__shr**: >> （右移）操作。 行为和 "band" 操作类似。

比如下面实现一个表用作集合操作，并对集合操作实现加法运算的元方法：

``` lua
local set = {}
local set_mt = {}

function set.new(v)
    local set = setmetatable({}, set_mt)
    for _, v in ipairs(v) do
        set[v] = true
    end
    return set
end

set_mt.__add = function(a, b)
    local ret = {}
    for k in pairs(a) do ret[k] = true end
    for k in pairs(b) do ret[k] = true end
    return ret
end
```

然后，我们可以按以下方法对一个集合进行加法运算了：

``` lua
local s1 = set.new{10, 20, 30, 40}
local s2 = set.new{20, 50}

print(getmetatable(s1)) --> table: 00E7E978
print(getmetatable(s2)) --> table: 00E7E978

local s3 = s1 + s2

local t = {}
for k in pairs(s3) do
    t[#t + 1] = k
end
print("{" .. table.concat(t, ", ") .. "}") --> {40, 10, 20, 30, 50}
```

## 关系运算符元方法

我们还可以指定关系运算符元方法，其主要包含以下几种操作：

- **__eq**: == （等于）操作。仅在两个值都是表或都是完全用户数据时，且它们不是同一个对象时才尝试该元方法，调用的结果总会被转换为布尔量。
- **__lt**: < （小于）操作。 仅在两个值不全为整数也不全为字符串时才尝试元方法，调用的结果总会被转换为布尔量。
- **__le**: <= （小于等于）操作。 和其它操作不同， 小于等于操作可能用到两个不同的事件。 首先，像 "lt" 操作的行为那样，Lua 在两个操作数中查找 `__le` 元方法，如果一个元方法都找不到，就会再次查找 `__lt` 元方法，Lua 会将 a <= b 转化为 not (b < a)；a ~= b 转换为 not (a == b)；a > b 转换为 b < a；a >= b 转换为 b < a。

我们可以尝试为上面集合增加一个集合相等操作：

``` lua
set_mt.__eq = function(a, b)
    for k in pairs(a) do 
        if not b[k] then return false end
    end

    for k in pairs(b) do
        if not a[k] then return false end
    end

    return true
end
```

然后我们对集合进行比较：

``` lua
local s1 = set.new{2, 4}
local s2 = set.new{2, 4, 5}
local s3 = set.new{2, 4}

print(s1 == s2) --> false
print(s1 == s3) --> true
```

## __index 与 __newindex 元方法

Lua 提供了一种能改变表在**访问**和**修改**表中不存在字段时的行为方式。

### __index 元方法

当我们访问一个表中不存在的字段时，通常情况下会返回 nil。但实际上，这样的访问方式会引发解释器取查找一个名为 `__index` 的元方法。如果没有找到这个元方法，则会直接返回 nil，否则会由这个元方法来提供最终的结果。

下面代码首先定义了一个原型 `prototype` 用来表示窗口坐标和大小信息，然后定义了一个 `new` 构造函数来产生一个对象，返回的对象直接设置成了元表 `mt`，该元表定义了元方法 __index，访问该元方法默认是直接访问 `prototype` 表的熟悉：

``` lua
prototype = {x = 0, y = 0, width = 100, height = 100}

local mt = {}

mt.__index = function(_, key)
    return prototype[key]
end

local function new(o)
    return setmetatable(o, mt)
end
```

然后我们可以调用函数 new 创建新对象，并指定对象的宽度和高度属性，可以预见的是，新对象并不包含坐标 x 和 y 的值，当我们访问不存在的 x 值时会触发直接访问 `__index` 元方法，并返回 `prototype` 的默认值 0：

``` lua
w = new{width = 50, heigh = 50}
print(w.x) --> 0
```

`__index` 虽然叫做元方法，但不一定非得是一个函数，它还可以是一个表。当元方法是一个函数时，Lua 会把当前表和不存在的参数名作为参数调用该函数；当元方法是一个表时，Lua 会直接访问这个表。
上面例子中，我们把 `__index` 字段直接设置为 `prototype` 时，访问不存在的值时，会直接返回 `prototype` 对应的值：

``` lua
mt.__index = prototype
```

### __index 元方法与 rawget 函数

有时候我们希望访问一个表时，不调用 `__index` 元方法，那么我们可以使用 `rawget (table, index)` 函数，该函数会在不触发任何元方法的情况下直接获取 `table[index]` 的值。

``` lua
w = new{width = 50, heigh = 50}
print(rawget(w, "x")) --> nil
```

可以看到上面代码使用 `rawget` 访问 x 属性时，并不会触发对元方法 `__index` 的访问，而是直接返回了 nil。

### __newindex 元方法

`__newindex` 与 `__index` 类似，不同之处在于 `__newindex` 用于表的更新操作，而 `__index` 用于表的查询操作。
当对一个表中不存在的索引赋值时，解释器就会触发 `__newindex` 元方法，如果这个元方法存在则会直接调用它，而不会继续执行赋值操作。

``` lua
mt.__newindex = function(t, key, value)
    error("attempt to update a nonexistent field", 2)
end

obj = new{width = 50, height = 50}
obj.name = "Window" -- 赋值不存在的键时，产生错误
```

上面例子中，我们可以使用 `__newindex` 元方法来拦截对不存在的字段的赋值操作，当不存在的 `name` 字段赋值时，会触发 `__newindex` 元方法，并抛出一个赋值错误的异常。

### __newindex 元方法与 rawset 函数

与函数 `rawget` 类似，原始函数 `rawset (t, k, v)` 允许我们绕过元方法，直接对某个表进行赋值操作。其中参数 t 必须时一张表，当我们调用 `rawset(t, k, v)` 时，其等价于 `t[k] = v`，但不会触发任何元方法。

``` lua
mt.__newindex = function(t, key, value)
    if key == "name" and type(value) ~= "string" then
        error("the assignment must be of type string")
    end
    rawset(t, key, value)
end

obj = new{width = 50, height = 50}
obj.name = 1 -- 非字符串类型，产生错误
```

上面例子中，当我们对对象的 `name` 字段赋值时，会触发 `__newindex` 元方法，该函数会检查 `name` 字段的值是否为字符串类型，如果不是则会抛出错误，否则执行正常赋值操作。