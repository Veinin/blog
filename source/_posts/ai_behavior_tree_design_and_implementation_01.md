---
title: AI 行为树设计与实现-理论篇
date: 2018-08-07 18:36:00
categories: AI
tags: [AI, 行为树, 状态机]
---

一个典型的 AI 系统通常包含：感知、导航和决策三个子系统。对于游戏来说，感知系统是可以“作弊”的，不需要NPC真的去“感知”世界，系统可以告诉NPC世界是怎么的。所以，对于导航系统，不再属于本文的讨论范畴。而决策系统才是让NPC看起来可以有自己的意图和信念的，所以本文讨论的是决策系统。

一个 AI 决策系统模型看起来是这样的：

![decision](/images/btree/decision.png)

最开始，游戏 AI 的决策系统往往会这样写：

```c
switch(自己)
    case '血量充足':
        攻击();
        break;
    case '快死了':
        加血();
        break;
    case ‘死了’:
        游戏结束();
        break;
}
```

<!--more-->

随机计算机硬件的不断提升，可以分配给 AI 运算的 CPU 时间越来越长，我们对游戏 AI 的要求也自然提高了，比如我们可以相处这样的策略：有多个敌人时，使用群体技能；只有单个敌人时，使用强力的单体攻击；魔法低于50时，吃药补魔法；血量低于100时，吃个大的血瓶回血。

于是，AI 程序员在上面的需求到来的时候，不得不继续扩充上的 swtich 条件，然后在 case 里面增加自己的逻辑。可以想象，如果一个 Moba 类游戏中，一个地狱难度的电脑的 AI 需要根据场上的情况使用各种策略。当策划的需求越来越多，很快，一个带有上万行的代码函数就横空出世啦。如果这时候遇到了一个 Bug ，不要说修复，仅仅是阅读这个函数的代码都恐怕让人觉得畏惧了。

毫无疑问，当一个函数遇到大量的状态转换判断的时候，很容易让整个程序崩溃，不过经过后辈们前赴后继的努力，目前市面上关于游戏的 AI 有了更加精简的代码手段。比如常见 的 AI 模型：FSM（有限状态机）和 Behavior Tree（行为树）。

## 有限状态机（FMS）

相对于 `switch - case` 来说， FSM 编程与人类的思维相似，更易于梳理，更加灵活。当每种状态封装后，就不在会有一个“中央”函数来控制所有的逻辑，每个状态只要管好自己的业务就行。这样，一个复杂的决策系统就被切割成了两子系统，不同的状态以及状态之间的转换。切割后的子系统与原有的系统被大大简化，从而使得代码变得可以维护。FSM 在相当多的游戏中已经被应用，甚至 Unreal Engine 的脚本语言是直接支持状态编程的。

当游戏中的NPC决策并不太复杂时候，FSM是非常有效的。比如 Half-Life 这款游戏，里面的AI被业界称赞了很久，而其中的AI就是通过FSM来实现的。

### 有限状态机举例

我们接下来通过一个简单的例子来认识一下FSM。比如一个AI文字表述如下：

1.平时的状态是巡逻
2. 如果遇到敌人之后打量一下敌人
3. 如果敌人比自己弱小，那就打攻击
4. 如果敌人比自己强大，那就跑逃跑

那么这个可以很自然的转换成 FSM，然后进行编程实现，我们可以看看整个 AI 流程图：

![fms](/images/btree/fms.png)

### 有限状态机缺点

虽然FSM简洁，和人的直觉思维相近，但是FSM也是有缺点的：

- 由于我们所能做的仅是编辑从一状态到另一状态的转换，而无法做出更高层次的模式功能，所以会导致我们发现自己总是在构建相似的行为，这会花费我们大部分时间。
- 使用 FSM 实现目标导向的行为需要做很多工作。这是一个大问题，因为大部分有针对性的AI 需要处理长远目标。
- FSM 难以并发。当并行运行多个状态机，要么死锁，要么我们通过手工编辑来确保它们在某个程度上能够兼容。
- 大规模支持较差，即使是分层的有限状态机，也难以大规模扩展。它们往往是在其中夹杂一大块逻辑代码，而非行为编辑模块化。
- 用 FSM 实现任何设计都需要做大量工作，需要花费设计师的大量时间(并非编程时间)，甚至最终这还会成行为中的 bugs 的来源。

## 行为树（Behavior Tree）

行为树是在Next-Gen AI中提出的模型，虽说是Next-Gen AI，但距其原型提出已有约10年时间。其中Spore(孢子)，Crysis(孤岛危机)2，Red Dead Redemption(荒野大镖客：救赎)等就是用行为树作为它们的AI模型。而越来越多的引擎也都开始直接支持行为树，比如 Cry Engine, Havok等。

对于用行为树定模型构造的AI系统来说，每次执行AI时 ，系统都会从根节点遍历整个树，父节点执行子节点，子节点执行完后将结果返回父节点，然后父节点根据子节点的结果来决定接下来怎么做。

所谓树，那么其就存在很多节点，而对于行为树来说，它把基本节点类型分为两大类：

### 控制节点

在行为树中我们所看到的所有父节点都被称为控制节点。控制节点是行为树的核心部分，它与具体的游戏是无关的，它只负责整个行为树的逻辑控制。其包含以下几种类型：

- 选择节点(Selector)：属于组合节点，顺序执行子节点，只要碰到一个子节点返回true，则停止继续执行，并返回true，否则返回false，类似于程序中的逻辑或。
- 顺序节点(Sequence)：属于组合节点，顺序执行子节点，只要碰到一个子节点返回false，则停止继续执行，并返回false，否则返回true，类似于程序中的逻辑与。
- 平行节点(Parallel)：提供了平行的概念，无论子节点返回值是什么都会遍历所有子节点。所以不需要像 Selector/Sequence 那样预判哪个 Child Node 应摆前，哪个应摆后。Parallel Node增加方便性的同时，也增加实现和维护复杂度。
- 组合节点(Coposite)：可以组合多个子节点。
- 装饰节点(Decoraor)：可以作为某种节点的一种额外的附加条件，如允许次数限制，时间限制，错误处理等。
- 循环节点(Loop)：循环执行相应的动作，并返回结果。

### 行为节点

行为树的行为定义都在行为节点中，也就是我们说的叶子节点。行为节点是与我们的具体需求相关的，不同的需求定义会有不同的行为节点。

- 条件节点(Condition)：属于叶子节点，判断条件是否成立。
- 动作节点(Action)：属于叶子节点，执行动作，一般返回true。

当然，如果有更多需求，也可以自己继续拓展，通过控制与行为节点的组合，最终，会产生一颗行为树，它看起来会是这样的：

![behavior_tree](/images/btree/behavior_tree.png)

### 行为树举例

我们可以来看一个行为树构造 AI 的例子，这个AI的逻辑文字表述为：

1.一个NPC在晚上需要执行守夜巡逻的任务。
2.如果到了白天，那么NPC需要休息。
3.如果天下雨的话，则需要打伞。
4.如果天气变晴，处在打伞状态下的人需要把伞收起。

通过以上条件，我们可以转换成一颗行为树：

![behavior_tree_npc](/images/btree/behavior_tree_npc.png)

### 行为树优缺点

行为树模型看似简单，但是以下几个优点让行为树目前变成了复杂AI的主流模型：

- 静态性。越复杂的功能越需要简单的基础，否则最后连自己都玩不过来。即使系统需要某些"动态"性，也应该尽量使用静态的行为树来表示。静态性直接带来的好处就是整棵树的规划无需再运行时动态调整，大大方便设计人员和编程人员，并且大大减少诡异的bug，同时这也为很多优化和预编辑都带来方便。
- 直观性。行为树可以方便地把复杂的AI知识条目组织得非常直观。默认的组合节点处理子节点的迭代方式就像是处理一个预设优先策略队列，也非常符合人类的正常思考模式：先最优再次优。此外，行为树编辑器对优秀的程序员来说也是唾手可得。
- 复用性。各种节点，包括叶子节点，可复用性都极高。
- 扩展性。可以容易地为项目量身定做新的组合节点或修饰节点。还可以积累一个项目相关的节点库，长远来说非常有价值。