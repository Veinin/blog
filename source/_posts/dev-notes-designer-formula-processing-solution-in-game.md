---
title: 开发笔记：游戏中属性定义、策划公式处理方案
date: 2018-03-24 21:28:00
categories: 开发笔记
tags: [游戏角色属性设计, 游戏策划公式设计]
---

在游戏设计中，策划喜欢通过表格来制定自己的数值方案，通常一个游戏制作会有几十上百张不同功能的表格，这些表格通常采用Excel制定。大部分时候，策划更想要的是自己来操作所有数值，而不需要程序为其各种需求去正对性的通过代码来实现。

这样做有几个好处：

- 1.策划可以独立拓展自己的业务，而不依赖于程序。
- 2.程序可以更加专心于自己的逻辑功能，而不用去特意维护策划的数值变动需求。
- 3.工作内容分离，减少沟通成本，更加提升工作效率，让各部门更加专注于自己所处的领域。

在游戏日常开发中，有两个典型的应用场景，一个是角色的属性，一个是计算这些属性的公式，最后还需要通过这些属性在战斗中通过公式生成伤害数值。如果涉及复杂的战斗系统，通常一个角色的战斗属性可能会有上百种，而计算这些属性的公式，也可能成百上千。这里就会产生几个问题：属性、公式由程序去定义，还是策划自己去主导？属性、公式通过某种方式定义出来了，程序如何去使用？

<!--more-->

最近和策划一起讨论了下，最终给出了解决方案，属性、公式都有策划去配置，但公式需要简化，于是我们先给出了两张表格。

### 属性表

一个属性字典，对于程序来说，特别设计属性服务器、客户端传输的过程，我们可以通过一个唯一ID、类型进行，只要客户端、服务器都拥有这么一张属性字典，那么将很容易对属性进行传输：

属性ID | 名称 | 类型 | 初值 | 描述 | 成长公式
:----: | :--: | :--: | :--: | :--: | :------:
1 | strength | int | 5 | 力量 |
2 | pdCorrect | float | 1.2 | 物理伤害修正系数 | 
3 | physicalDamage | int | 20 | 物理基础伤害 |  公式1
4 | physicalCritical | int | 15 | 物理暴击率 |
5 | physicalCriticalDec | int | 10| 物理暴击抵抗率 | 

### 公式表

一张公式表，由策划去配置，但程序得把策划的公式翻译成程序能读懂的代码。

公式ID | 公式名称 | 公式内容
:--: | :--: | :------:
1      | physicalDamage  | 40 * (level * pdCorrect + 1) * rand(1, 1.5)
1      | physicalCriticalRate  | max(min(a.physicalCritical / 10 - t.physicalCriticalDec), 20), 0)

### 函数支持

另外，对于策划来说，要通过公式来操作属性数据，特别是战斗中产生的伤害数据，就要求我们有一些简单函数支持，比如在Lua中，一些数学公式：`math.min`、`math.ceil`、`math.random` 等等。而对策划来说，大部分其实是不懂编程的，因此我们需要更加简化函数名称的设计。最终，我们得出需要以下函数的支持：

- min(...) ，返回参数中的最小值，如 min(1, 5, 2) ，会得到数值 1
- max(...)，返回参数中的最大值，如 max(4, 10, 3)，会得到数值 10
- rand(m, n)，当不带参数时，返回 [0,1] 区间内的浮点伪随机数，当以两个整数 m 与 n 调用时，返回一个 [m, n] 区间内的一致分布的伪随机数。如 rand(1, 10)，产生1-10区间内的一个随机数。
- ceil(x)，返回不等于 x 的最小整数。如 ceil(1.55)，会得到数值 2
- float(x)，返回不大于 x 的最大整数值，如 flooat(1.55)，会得到数值 1

### 实现

有了上面这些，程序就可以编写工具，把策划配置的公式，导出成程序能够识别的公式函数了，比如：
怪物的随等级增长物理伤害公式：40 * (等级 * 物理伤害修正系数 + 1) * rand(1, 1.5)
怪物的物理暴击计算：max(min(怪物暴击率 / 10 - 目标暴击抵抗率 / 10, 25), 0)

那么其最终需要转换成代码公式函数：

```lua
local formula = {}

-- 怪物物理伤害公式
function formula.physicalDamage(a)
    return 40 * (a.level * a.pdCorrect + 1) * math.random(1, 1.5)
end

-- 怪物暴击几率公式
function formula.physicalCriticalRate(a, t)
    return math.max(math.min(a.physicalCritical / 10 - t.physicalCriticalDec), 20), 0)
end

local formulas = {
    [1] = formula.physicalDamage,
    [2] = formula.physicalCriticalRate,
}

function formula.exec(id, ...)
    return formula[id](...)
end

return formula
```

有了上面的表格，外加生成的公式，我们很容易计算出一个怪物的基础属性或战斗中产生的伤害数值：

```lua
local formula = require "formula"

local monster = {
     level = 3,
     strength = 5,
     pdCorrect = 1.3,
     physicalDamage = 15,
     physicalCritical = 35,
     physicalCriticalDec = 10,
}

local target = {
     level = 2,
     strength = 5,
     pdCorrect = 1.3,
     physicalDamage = 20,
     physicalCritical = 35,
     physicalCriticalDec = 10,
}

-- 得出怪物物理伤害
local damage = formula.exec(1, monster)

-- 得出怪物物理暴击率
local criticalRate = formula.exec(2, monster, target)
```

最终，通过这套方案的实现，程序不用再去代码里维护各种各样的公式，策划也不在需要程序来帮忙维护公式，如果需求变动，只需要更改下表格，然后重新生成一份新公式就行；如果有新属性增加，只需要在表格中创建以个新的属性值，然后不管是角色，还是战斗中都能应用到新增加的变化。