---
title: Lua入门教程：数值
date: 2018-12-15 20:12:33
categories: Lua入门教程
tags: [Lua 5.3, Lua编程]
---

Lua 5.2 以及之前的版本，所有数值类型都是以双精度浮点数表示，并且所有浮点型数值最大都是 `2^54`，但是 Lua 5.3 版本引入了整形，最大可用 64 位表示。

## 数值表示方式

Lua 数值默认是十进制表示，除了整形之外，其他小数或者指数都会当做浮点型数值。我们可以使用函数 `type` 来获数值类型，他们都是使用 `number` 表示数值：

``` lua
> 1         --> 1
> 1.0       --> 1.0
> 1.2e5     --> 120000.0
> 0.2e4     --> 2000.0
> 0xff      --> 255
> type(1)   --> number
> type(1.0) --> number
```

在 Lua 5.3 中为了区分整形和浮点型数值，引入了 `math.type` 函数，如果是整形该函数返回 `integer`，浮点型则返回 `float`：

``` lua
> math.type(1)      --> integer
> math.type(1.0)    --> float
```

同时为了方便判断是否可以把一个数转换到整数，Lua 5.3 引入了 `math.tointeger` 函数，如果目标可以转换为一个整数，返回该整数，否则返回 `nil` ：

``` lua
> math.tointeger(3.0)   --> 3
> math.tointeger(3.1)   --> nil
```

<!--more-->

## 数值运算

Lua 5.3 引入整形后，如果我们对两个整形进行加(+)、减(-)、乘(*)、除(/)、取负数(-)操作，其结果任然是整形。并且为了方便我们对整数进行除法运算，引入了一个新的 `float` 除法运算符 `//`，该运算符会对得到的商向负无穷取整，从而保证结果是一个整形。而如果我们忽略整形和浮点型之间的区别，那么其运算规则和其他的 5.x 版本没有任何差别：

``` lua
>
> 1 + 1     --> 2
> 1 + 1.0   --> 2.0
> 1.0 + 1.0 --> 2.0
> 6 // 3    --> 2
> -6 // -3  --> 2
> 2.0 // 2  --> 1.0
> 1 // 0.5  --> 2.0
> 1 + 0.0   --> 1.0
> 1.0 | 0   --> 1
```

## 数值关系运算

我们可以使用 Lua 的关系运算符大于（>）、小于（<）、大于且等于（>=）、小于且等于（<=）以及不等于（~=）对数数值进行关系运算：

``` lua
> 1 < 2     --> true
> 1 > 2     --> false
> 2 >= 2    --> true
> 3 <= 4    --> true
> 4 ~= 4    --> false
> 5.0 ~= 5  --> false
> 1.0 == 1  --> true
```

## 数值运算符优先级

Lua 中操作符的优先级从高优先级到低优先级排序表示如下：

``` lua
^
一元运算符 (not   #     -     ~)
*     /     //    %
+     -
..
<<    >>
&
~
|
<     >     <=    >=    ~=    ==
and
or
```

通常，你可以用括号来改变运算次序。连接操作符 ('..') 和乘方操作 ('^') 是从右至左的。 其它所有的操作都是从左至右。

``` lua
> 1 + 2 == 2 * 3 // 2      -->  (1 + 2) == (2 * 3) // 2
> 1 and 3 or 2             -->  (1 and 3) or 2
> -1 ^ 2                   -->  -(1^2)
> 1 + 2 > 3 and 4
```

## 位运算

Lua 5.3 版本开始提供针对数值类型的一组标准位运算符，与算术运算符不同的是，位运算只能用于整形数值。

位运算符包括：

- **&**，按位与
- **|**，按位或
- **~**，按位异或
- **>>**，右移
- **<<**，左移
- **~**，按位取反

下面例子中的位运算符使用了函数 string.format 来输出十六进制形式的结果：

``` lua
> string.format("%s", 0xff & 0xabcd) --> cd
> string.format("%s", 0xff | 0xabcd) --> abcd
> string.format("%x", 0xaaaa ~ -1)   --> ffffffffffff5555
> string.format("%x", ~0)            --> ffffffffffffffff
```

## 数学库

除了前面提到 Lua 5.3 新增的 `math.type`、`math.tointeger` 函数外，Lua 还提供了其他一系列基础的数学操作函数库给我们，基本能满足我们日常编程的大部分需求。

### 基本数值运算函数

- math.abs(x)，返回 x 的绝对值。
- math.acos(x)，返回 x 的反余弦值（用弧度表示）。
- math.asin(x)，返回 x 的反正弦值（用弧度表示）。
- math.atan(x)，返回 y/x 的反正切值（用弧度表示）。
- math.ceil(x)，返回不小于 x 的最小整数值。
- math.cos(x)，返回 x 的余弦（假定参数是弧度）。
- math.deg(x)，将角 x 从弧度转换为角度。
- math.exp(x)，返回 ex 的值 （e 为自然对数的底）。
- math.floor(x)，返回不大于 x 的最大整数值。
- math.fmod(x)，返回 x 除以 y，将商向零圆整后的余数。 (integer/float)
- math.log(x)，返回以指定底的 x 的对数。 默认的 base 是 e （因此此函数返回 x 的自然对数）。
- math.modf(x)，返回 x 的整数部分和小数部分。 第二个结果一定是浮点数。
- math.rad(x)，将角 x 从角度转换为弧度。
- math.sin(x)，返回 x 的正弦值（假定参数是弧度）。
- math.sqrt(x)，返回 x 的平方根。
- math.tan(x)，返回 x 的正切值。
- math.ult(x)，如果整数 m 和 n 以无符号整数形式比较， m 在 n 之下，返回布尔真否则返回假。
- math.min(x, ...)，返回参数中最小的值， 大小由 Lua 操作 < 决定。
- math.max(x, ...)，返回参数中最大的值， 大小由 Lua 操作 < 决定。

``` lua
> math.abs(-10)     --> 10
> math.floor(2.33)  --> 2
> math.floor(-2.33) --> -3
> math.ceil(2.33)   --> 3
> math.ceil(-2.33)  --> -2
> math.modf(2.3)    --> 2   0.3
> math.modf(-2.3)   --> -2  -0.3
```

### 数值返回表示

前面说过，Lua 5.3 使用 64 位来存储整数值，其最大值为 2^63 - 1，数学库中可以使用 `math.maxinteger` 与 `math.mininteger` 来表示整数的最大值和最小值。另外 Lua 使用 `math.huge` 这个值来表示最大的数，这个值比我们能用到的所有数值都要大。

``` lua
> math.pi               --> 3.1415926535898 (π 的值)
> math.huge             --> inf
> math.maxinteger       --> 9223372036854775807
> math.mininteger       --> -9223372036854775808
> math.maxinteger + 1   --> -9223372036854775808
> math.mininteger - 1   --> 9223372036854775807
> math.maxinteger < math.huge   --> true
```

再次强调，Lua 中所有双精度浮点型的值最大只能用 `2^53 - 1` 表示，如果超过这个数值，进行数值运算则会造成精度损失：

``` lua
> 9007199254740992 + 0.0 == 9007199254740992    --> true
> 9007199254740993 + 0.0 == 9007199254740993    --> false
```

### 随机数

Lua 使用 `math.random ([m [, n]])` 来产生伪随机数，其有 3 种随机数产生方式：

- 不带任何参数时，返回一个 [0, 1) 区间内一致分布的浮点伪随机数。
- 两个整数 m 与 n 调用时， math.random 返回一个 [m, n] 区间 内一致分布的整数伪随机数。值 n-m 不能是负数，且必须在 Lua 整数的表示范围内。）
- 当调用 math.random(n) 时，其等价于 math.random(1, n)。

``` lua
> math.random()     --> 0.56356811523438
> math.random(1, 4) --> 1
> math.random(4)    --> 4
```

作为伪随机数函数，我们每次重新启动 Lua 虚拟机时，其产生随机数都会与上次启动时一模一样，比如在游戏中，我们每次重启游戏后，其产生的随机数都会和上一次启动时一模一样，这其实不是我们所想要的，为了使程序每次启动时都会有不同的随机数产生，我们就需要使用 `math.randomseed (x)` 函数来初始化随机数生成器。当我们调用次函数后，后面的所有随机数都是基于此函数设置的种子生成的。

通常来说，对于随机数种子，如果没有特殊需求，那么我们直接使用 `os.time()` 获取当前系统时间来当做随机数种子：

``` lua
> math.randomseed(os.time())
> math.random() --> 0.84405517578125
> math.randomseed(os.time())
> math.random() --> 0.79620361328125
```

但是有时候，我们必须设置特殊的随机数种子来让随机数表现出一致性，比如 Moba 游戏中，通常很多数值都是客户端进行运算，如果我们希望一局游戏中所有客户端产生的随机数都是一致的，那么我们可以让所有客户端在开始战斗前都统一设置一个随机数种子，那么不同的玩家客户端在同一局游戏中产生的所有随机数都会是一致的：

游戏客户端A:

``` lua
> math.randomseed(111231254123)
> math.random() --> 0.63229370117188
```

游戏客户端B：

``` lua
> math.randomseed(111231254123)
> math.random() --> 0.63229370117188
```