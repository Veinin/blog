---
title: 寻路算法揭秘（3）：A* 解密
date: 2019-02-23 20:21:00
categories: 寻路算法
tags: [寻路算法, A*]
---

## 介绍

本系列文章中的第一篇介绍了通用的寻路算法，每种不同类型的寻路算法背后都是它的一个微小变化。

第二篇文章揭示了不同搜索算法背后的秘密：这一切都要归结于 `choose_node` 函数。它还提供了一个相当简单的 `choose_node` 函数来生成一个称为**统一搜索成本**的算法。

这个算法非常好：它将找到从开始节点到目标节点的最短路径。然而，它也有一点点浪费：它会前往人们可以很容易察觉到的错误路径节点，这往往会让目标偏移，我们能避免这种情况的发生吗？

## 魔术算法

想象一下，我们在一台特殊的计算机上运行搜索算法，它拥有一个可以变魔术的芯片。有了这个很棒的芯片，我们就能用一种非常简单的方式来表达 `choose_node` 函数，它保证产生最短的路径，不会浪费时间去探索那些不会通向目标地方的道路:

``` test
function choose_node (reachable):
    return magic(reachable, "最短路径的下一个节点是什么")
```

<!--more-->

虽然这很诱人，但是神奇的芯片仍然需要一些更低级别的代码，下面是一个类似的实现：

``` test
function choose_node (reachable):
    min_cost = infinity
    best_node = None

    for node in reachable:
        cost_start_to_node = node.cost
        cost_node_to_goal = magic(node, "shortest path to the goal")
        total_cost = cost_start_to_node + cost_node_to_goal

        if min_cost > total_cost:
            min_cost = total_cost
            best_node = node

    return best_node
```

这是选择下一个节点的好方式：我们选择从开始节点到目标节点的最短路径，而这正是我们想要的。

我们还最大幅度的减少了魔法的使用：我们确切的知道从起始节点到目标节点（即node.cost）所需成本，并且我们只是使用魔法来预测从当前节点前往目标节点的成本。

## 不神奇但是非常棒的 A* 算法

不幸的是，魔术芯片很新，但我们希望其支持传统的硬件。上面大多数代码都没问题，除了这一行：

``` test
# 抛出麻瓜处理器异常
cost_node_to_goal = magic(node, "shortest path to the goal")
```

所以，我们并不能用魔法来知道我们尚未探索的道路的成本。很好，那让我们来猜猜看，我们很乐观，所以我们假设当前节点到目标节点之间没有任何东西，我们可以直接走：

``` test
cost_node_to_goal = distance(node, goal_node)
```

注意，最短路径和最小距离是不同的：最小距离假定了当前节点和目标节点之间绝对没有障碍物。

这个假设当然非常简单。在我们基于网格的示例种，它是两个节点之间的曼哈顿距离，而所谓“曼哈顿距离”是指两点在在东西方向上的距离加上南北方向上的距离，即：abs(Ax - Bx) + abs(Ay - By)。如果你可以移动到对角线，则它的距离会是：sqrt( (Ax - Bx)^2 + (Ay - By)^2)。诸如此类，但最重要的是永远不要高估道路的成本。

所以，下面是一个没有魔法的 `choose_node` 版本：

``` test
function choose_node (reachable):
    min_cost = infinity
    best_node = None

    for node in reachable:
        cost_start_to_node = node.cost
        cost_node_to_goal = estimate_distance(node, goal_node)
        total_cost = cost_start_to_node + cost_node_to_goal

        if min_cost > total_cost:
            min_cost = total_cost
            best_node = node

    return best_node
```

预估从节点到目标节点的距离的函数成为**启发式算法**，观众朋友们，我们称这种算法叫做 _A*_。

## 在线演示

当你从意识到神秘的 _A*_ 算法实际上那么简单的震惊中恢复过来时，这里有一个你可以试玩的演示。与前面的示例不同，你会注意到整个搜索在错误的方向上浪费的时间非常少。

<script src="http://www.gabrielgambetta.com/js/path_helpers.js"></script>

<table><tr><td><canvas width="401" height="401" id="demo"></canvas></td><td><p id="reachable" style="color:green; font-family:courier;">reachable = []</p><p id="explored" style="color:red; font-family:courier;">explored = []</p><p id="path" style="font-family:courier;"></p><p><input type="button" value="Restart" onclick="stopRunSearch(); search.reset();"></input> <input type="button" value="Step" onclick="stopRunSearch(); search.step();"></input> <input type="button" value="Run" onclick="runSearch();"></input></p></td></tr></table>

<script>
// Choose the node with the lowest path cost.
Search.prototype.chooseNode = function() {
    // return this.reachable[Math.floor(Math.random() * this.reachable.length)];
    var min_cost = Infinity;
    var best_node = undefined;

    for (var i in this.reachable) {
        var node = this.reachable[i];

        // Compute Manhattan distance to goal.
        var idx = getNodeIndex(node, this.graph.nodes);
        var nr = Math.floor(idx / this.graph.cols);
        var nc = idx % this.graph.cols; 

        idx = getNodeIndex(this.goal_node, this.graph.nodes);
        var gr = Math.floor(idx / this.graph.cols);
        var gc = idx % this.graph.cols; 

        var node_to_goal_distance_estimate = Math.abs(nr - gr) + Math.abs(nc - gc);
        var total_cost = node.cost + node_to_goal_distance_estimate;

        if (total_cost < min_cost) {
            min_cost = total_cost;
            best_node = node;
        }
    }

    return best_node;
}

Search.prototype.addAdjacent = function(node, adjacent) {
    if (findNode(adjacent, this.explored)) {
        return;
    }

    if (!findNode(adjacent, this.reachable)) {
        this.reachable.push(adjacent);
    }

    if (adjacent.cost > node.cost + 1) {
        adjacent.cost = node.cost + 1;
        adjacent.previous = node;
    }
}

// Build the grid used in the example.
// "*" represent a blocked square, " " an open one.
var graph = new Graph(["          ",
                       "          ",
                       "          ",
                       "          ",
                       "          ",
                       "          ",
                       "        * ",
                       "       ** ",
                       "      **  ",
                       "          "]);

var stepDelay = 100;

var search = new Search(graph, "BN", "CK");
search.reset();

render(search);
</script>

## 结论

我们最终得到了 _A*_ 算法，它只不过是第一篇文章中描述的通用搜索算法，在第二篇文章的描述中加入了一些改进，使用 `choose_node` 函数选择我们预估的节点将使我们更加接近目标，仅此而已。

参照上面的结论，我们给出了主要方法的伪代码：

``` test
function find_path(start_node, goal_node):
    reachable = [start_node]
    explored = []

    while reachable is not empty:
        # 选择一个我们知道如何到达的节点
        node = choose_node(reachable)

        # 如果我们刚好到达了目标节点，则构建并返回路径。
        if node == goal_node:
            return build_path(goal_node)

        # 记录已经处理的节点，防止重复处理
        reachable.remove(node)
        explored.add(node)

        # 我们能从这个节点得到我们以前没有探索过的新节点
        new_reachable = get_adjacent_node(node) - explored
        for adjacent in new_reachable:
            # 是不是第一次抵达这个节点
            if adjacent not in reachable:
                reachable.add(adjacent)

            # 如果这是一个新路径，或者比现有的路径更短，则保留它
            if node.cost + 1 < adjacent.cost:
                adjacent.previous = node
                adjacent.cost = node.cost + 1

    # 如果我们到达了这里，则路径没有找到
    return None
```

`build_path` 方法实现：

``` test
function build_path(to_node):
    path = []
    while to_node != None:
        path.add(to_node)
        to_node = to_node.previous
    return path
```

使用 A* 算法实现的 `choose_node` 方法：

``` test
function choose_node(reachable):
    min_cost = infinity
    best_node = None

    for node in reachable:
        cost_start_to_node = node.cost
        cost_node_to_goal = estimate_distance(node, goal_node)
        total_cost = cost_start_to_node + cost_node_to_goal

        if min_cost > total_cost:
            min_cost = total_cost
            best_node = node

    return best_node
```

经此而已。

那么为什么本系列文章还有第四部分？

现在你已经了解了 _A*_ 算法的工作原理，我想探索一些令人难以置信的应用程序，而不仅仅是在一个正方形的网格中搜索路径。

---

* 系列文章目录：
  * [寻路算法揭秘（1）：通用搜索](/2019/02/18/pathfinding-demystified-01/)
  * [寻路算法揭秘（2）：搜索策略](/2019/02/20/pathfinding-demystified-02/)
  * [寻路算法揭秘（3）：A* 解密](/2019/02/23/pathfinding-demystified-03/)
  * [寻路算法揭秘（4）：实用 A* 算法](/2019/02/28/pathfinding-demystified-04/)

* [翻译原文](http://www.gabrielgambetta.com/generic-search.html)