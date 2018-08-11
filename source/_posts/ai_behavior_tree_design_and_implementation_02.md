---
title: AI 行为树设计与实现-实现篇
date: 2018-08-08 21:15:00
categories: AI
tags: [AI, 行为树, 状态机]
---

上一篇文章中，我们主要介绍了有限状态机和行为树的各自特点以及他们之间的优劣势，本文着重讲解如何实现一个可用的行为树，手动实现一个行为树的库，这个库代码不到200行，代码库采用面向对象方式实现，语言采用的是 Lua，当然你也可以翻译成其他任何你顺手的语言。

## 节点

行为树时一个树形结构，我们AI系统要做的是执行每个节点，所以我们最开始必须先抽象一个节点出来，该节点作为所有其他类型节点的父节点：

``` lua
local Node = class('Node')

function Node:ctor(opts)
    self.blackboard = opts.blackboard
end

function Node:doAction()
end

return Node
```

我们可以看到，节点拥有一个可选的成员变量 `blackboard`，我们称它为黑板，你可以搜索查看设计模式关于 [黑板模式](https://www.google.com.hk/search?hl=zh-CN&q=%E9%BB%91%E6%9D%BF%E6%A8%A1%E5%BC%8F&gws_rd=ssl) 的详细介绍。

引入黑板，是因为很多节点在处理AI逻辑时是需要对数据进行处理，黑板中包含了整个行为树可以操作的数据结构。比如上一节我们提到的关于一个NPC守夜巡逻的实现中，整个行为树要处理的对象其实是NPC，NPC打伞、NPC巡逻动作其实是NPC自身的一个行为，而行为树各个节点要处理的目标就是存放在黑板中的这个NPC。

我们实现的行为树节点对外部来说只有一个入口 `doAction` 函数，下面实现的所有其他类型的行为树节点都继承自 `Node` 节点。

<!--more-->

## 节点执行状态

上一章我们解释了行为树包含两大类子节点：控制节点与行为节点。

- 控制节点，这类节点没有任何游戏内的具体逻辑实现，只是为了控制行为节点的执行；
- 行为节点，这是涉及我们游戏内的具体游戏逻辑的节点，行为节点通常会在执行完成后，返回执行成功与否，拥有行为节点的父节点，也就是控制节点，会根据行为节点的返回值，做出相应的控制。

为了更加明白的阐述，我们举个例子，例如上一节提到的NPC夜晚巡逻的AI，我们把“是否晚上”、“巡逻”这种节点称为行为节点，行为节点需要返回一些执行结果，如“是否晚上”节点返回为真，则控制节点再运行“巡逻”行为。

所以，在进入详细代码设计之前，所有节点的执行，都是需要定义一些返回状态常量值的：

``` lua
local BTConst = {}

BTConst.SUCCESS = 0
BTConst.FAIL = 1
BTConst.WAIT = 2

return BTConst
```

行为树的节点，通常需要以上三种执行状态，执行节点成功、失败以及等待，对于等待状态，是因为一些节点单次执行并没有成功，需要等待下一次的执行时间到来以继续执行。

## 控制节点实现

对于一个行为树系统，其核心部分主要是其提供的控制节点，通过控制节点，我们可根据自己的游戏需求拓展出一套相对应的条件与行为节点。我们必须再度强调控制节点是与具体的游戏逻辑是无关的，它只负责整个行为树的逻辑控制。

本文的代码采用面向对象开发，实现语言采用 Lua。在实现控制节点前，我们先看看控制节点的类结构：

![behavior_node](/images/btree/behavior_node.png)

从图中我们可以看到，控制节点核心包含选择节点（Selector）、序列节点（Sequence）和平行节点（Parallel），它们都继承自一个组合节点（Composite）。

### 组合节点(Coposite)

组合节点可以看作是一个节点的集合，可以组合多个子节点。行为树的控制节点都是组合各类行为节点的父节点，然后按一定的规则执行它们。我们先实现一个组合节点：

``` lua
local Composite = class('Composite', Node)

function Conposite:ctor()
    self.children = {}
end

function Composite:addChild(node)
    table.insert(self.children, node)
end

return Composite
```

组合节点成员只有一个，那就是一个孩子节点的集合，我们可以往组合节点上面添加任意数量的孩子节点。

### 选择节点

选择节点，顾名思义，就是需要从众多的子节点中选择一个可以执行成功的节点，如果子节点执行成功，则跳出执行，并且下次执行时继续从头开始。
选择在行为树中运用的频率比其他两个控制节点来说要更多，因为游戏AI系统中充斥着各种不同情况，我们的AI系统需要从中选择一个合理的路径执行。
比如下图某个NPC的AI是选择巡逻还是休息：

![selector](/images/btree/selector.png)

选择节点代码来实现：

``` lua
local Selector = Class('Selector', Composite)

function Selector:ctor()
    self.index = 1
end

local function reset(self)
    self.index = 1
end

function Selector:doAction()
    local size = #self.children

    if size == 0 then
        return BTConst.SUCCESS
    end

    if self.index > size then
        reset(self)
    end

    for i = self.index, size do
        local ret = self.children[i]:doAction()
        self.index = self.index + 1

        if ret == BTConst.SUCCESS then
            reset(self)
            return ret
        elseif ret == BTConst.WAIT then
            return ret
        end
    end

    reset(self)
    return BTConst.FAIL
end

return Selector
```

上面代码中，选择节点的策略是先从第一个子节点开始执行，如果遇到某个节点执行成功，则重置执行节点的位置，并直接返回，那么下次再进入该节点后，会直接从第一个子节点开始执行。
如果执行某个节点是等待状态，则直接返回，下次重新进入该节点执行时，会直接从等待的那个节点继续执行。
如果所有节点都未成功或没有等待，则返回失败，并重置执行位置，下次将会从头开始执行。

### 序列节点

序列节点执行是有序的，它会从从到尾依次执行其子节点，当碰到一个子节点失败是会被打断，并且立刻返回，剩下的节点将不会继续执行。
序列节点的特性适合于执行一系列连续的操作，如果某个操作执行失败，则后续的操作会被打断。
如NPC在选择巡逻还是休息的AI中，在判断巡逻时，需要判断是白天还是晚上，如果满足条件，则巡逻，否则休息：

![sequence](/images/btree/sequence.png)

序列节点代码来实现：

``` lua
local Sequence = class('Sequence', Composite)

function Sequence:ctor()
    self.index = 1
end

local function reset(self)
    self.index = 1
end

function Sequence:doAction()
    local size = #self.children

    if size == 0 then
        return BTConst.SUCCESS
    end

    if self.index > size then
        reset(self)
    end

    for i = self.index size do
        local ret = self.children[i]:doAction()
        self.index = self.index + 1

        if ret == BTConst.WAIT then
            return ret
        elseif ret == BTConst.FAIL then
            reset(self)
            return ret
        end
    end

    reset(self)
    return BTConst.SUCCESS
end

return Sequence
```

上面代码选择从头开始执行各个子节点，遇到某个节点等待，则跳出，下次继续执行剩余节点。
如果某个节点执行失败，则重置执行位置，并返回，下次执行时，会从头开始。
最后，在所有节点都执行成功后，重置执行位置，并返回执行成功，下次执行时，从头继续开始。

### 平行节点

平行节点与序列节点类似，但平行节点，每次执行时都会把所有子节点执行一边，然后统计其执行结果，根据平行节点执行策略，返回相应的结果。
平行节点影响执行结果返回会有好几种策略：

- FAIL_ON_ONE，只要有一个子节点执行返回失败，则执行结果就会返回失败状态。
- FAIL_ON_ALL，所有子节点执行失败了，执行结果才会返回失败，否则算做成功。

平行节点运用于执行一系列不被其他节点影响还的操作，如NPC巡逻AI中，在平行节点中有两个子节点，一个是选择天气，一个是选择守夜巡逻，天气系统中只会影响NPC是否需要打伞，而守夜巡逻分支影响NPC是否需要巡逻，着两个节点会独立的运行，它们之间不会相互影响的，

![parallel](/images/btree/parallel.png)

平行节点代码来实现：

``` lua
local FAIL_ON_ONE = 1
local FAIL_ON_ALL = 2

local Parallel = class('Parallel', Composite)

function Parallel:ctor(opts)
    self.policy = opts and opts.policy or FAIL_ON_ONE
    self.waits = {}
    self.succ = 0
    self.fail = 0
end

local function reset(self)
    self.waits = {}
    self.succ = 0
    self.fail = 0
end

local function checkPolicy(self)
    if self.policy == FAIL_ON_ONE then
        return self.fail > 0 and BTConst.FAIL or BTConst.SUCCESS
    elseif self.policy == FAIL_ON_ALL then
        return self.succ > 0 and BTConst.SUCCESS or BTConst.FAIL
    end
end

function Parallel:doAction()
    local size = #self.children

    if size == 0 then
        return BTConst.SUCCESS
    end

    local ret
    local rest = {}
    local origin = #self.waits > 0 and self.waits or self.children

    for i = 1, #origin do
        ret = origin[i]:doAction()

        if ret == BTConst.SUCCESS then
            self.succ = self.succ + 1
        elseif ret == BTConst.WAIT then
            table.insert(rest, origin[i])
        elseif ret == BTConst.FAIL then
            self.fail = self.fail + 1
        end
    end

    if #rest > 0 then
        self.waits = rest
        return BTConst.WAIT
    end

    ret = checkPolicy(self)
    reset(self)
    return ret
end
```

上面代码中，首先声明了平行节点执行策略 `FAIL_ON_ONE` 与 `FAIL_ON_ALL`。平行节点定义了3个类成员 `waits` 表示执行等待的节点，`succ` 表示执行成功的节点数量，`fail` 表示执行失败的节点数量。
节点开始执行前先获取执行起点，如果有等待执行的节点，有优先执行等待节点；如果没有等待的节点，则从头开始执行各个子节点。
子节点执行成功、失败，则记录，如果是带状态则记录到等待列表 `rest` 中，并在执行完成后，检查 `rest`，如果存在等待节点，则直接返回，下次继续执行等待节点。
如果所有节点执行都返回成功或失败，则根据记录从节点策略中获取返回状态码，并重置所有执行记录，等待下一次重新开始执行。

### 装饰节点

装饰节点可以作为额外的附加条件，例如，时间间隔控制、频率控制、触发概率、结果取反、错误处理等待。
比如我一个怪物使用某个技能，我们实现了使用技能的行为节点，但我们需要对这个使用技能的行为节点增加节点触发概率，这种情况下，我们就可以使用装饰节点来扩充原有的行为节点。

``` lua
local Decorator = class('Decorator', Node)

function Decorator:ctor(opts)
    self.child = opts.child
end

function Decorator:setChild(node)
    self.child = node
end

return Decorator
```

上面装饰节点中，存储了一个装饰子节点，这个子节点将作为被装饰的对象。

### 循环节点

循环节点根据条件不停循环执行子节点，直到子节点返回成功，然后检查循环条件，判断自己是否继续循环。比如巡逻AI中，根节点就是一个循环节点，默认情况下，循环节点会移植运行下去，直到循环节点运行成功，比如我们需求在NPC进行巡逻后，就结束AI的运行等等。

![loop](/images/btree/loop.png)

我们看下代码：

``` lua
local Loop = Class('Loop', Decorator)

function Loop:ctor(opts)
    self.loopCond = data.loopCond
end

function Loop:doAction()
    local ret = self.child:doAction()
    if ret ~= BTConst.SUCCESS then
        return ret
    end

    if self.loopCond and self.loopCond(self.blackboard) then
        return BTConst.WAIT
    end

    return BTConst.SUCCESS
end

return Loop
```

循环节点也是一个装饰节点，其有一个循环检查条件节点用于在子节点执行成功后检查，

## 行为节点实现

有了控制节点后，我们剩下的就是实现行为节点了，行为节点可能每个游戏都会不一样，这主要是因为每个游戏AI可能都不同，具体需求到来时，才针对性的实现，所以行为节点通常依托于控制节点，其只会作为叶子节点使用。

### 条件节点

条件节点的作用是判定，如NPC巡逻AI中，天气判定、打伞判定、时间判定都是一个条件节点。先看条件节点实现：

``` lua
local Condition = class('Condition', Node)

function Condition:ctor(opts)
    self.umpire = opts.umpire
end

function Condition:doAction()
    if self.umpire and self.umpire(self.blackboard) then
        return BTConst.SUCCESS
    end

    return BTConst.FAIL
end

return Condition
```

条件节点有一个成员变量 `umpire` ，我们把他视为一个“裁判员”，用来判定某个条件是否成立。这个变量在 Lua 中实现为一个函数，我们可以自定义一个函数，然后根据上面的接口创建一个条件，比如创建判定天气为晴天的条件节点：

``` lua
local isSunny = function(blackboard)
    return blackboard.world.weather == "Sunny"
end

local isSunnyCondition = Condition.new(isSunny)
isSunnyCondition.doAction()
```

可以看到上面创建一个判定天晴的条件节点是分成简单的，判定条件参数传入的是一个“黑板”成员，里面保存着AI需要的数据对象，比如上面代码中，我们可以直接通过黑板获取到世界`world`对象，来获取当前天气。

### 动作节点

动作节点人如其名，只会执行相应的动作，具体执行什么动作，也是根据游戏 AI 需求去重新实现的，比如我们的巡逻AI中，NPC 执行打伞、收伞、巡逻等一系列的动作。动作节点只需要继承自 Node 节点，然后实现 `doAction` 内容即可，如一个 NPC 的巡逻动作：

``` lua
local PatrolAction = class('PatrolAction', Node)

function PatrolAction:doAction()
    local npc = self.blackboard.npc
    npc:GoPatrol()
end

return PatrolAction
```

上面巡逻动作节点实现很简单，只需要执行时，从黑板中获取到NPC对象，然后调用NPC巡逻函数 `GoPatrol`，NPC就开始进行巡逻了。

## 总结

本章节，我们的行为树核心代码基本全部实现，对于AI不太复杂的游戏，这些代码基本可用了，当然我们也可以根据上面代码在项目需求满足不了的情况下，继续拓展不同类型、不同功能的节点，比如可以支持权值的选择节点等等。下一节，我们将根据这些代码来实现一个具体的 NPC 巡逻 AI。