---
title: 寻路算法揭秘（2）：搜索策略
date: 2019-02-20 18:20:00
categories: 寻路算法
tags: [寻路算法, A*]
---

## 介绍

本系列第一篇文章介绍了通用寻路算法。如果你没有完全理解它，请返回并重新阅读，因为其对你了解后面的内容至关重要，一旦你掌握了它，A* 算法将是世界上最自然的事情。

## 秘制酱汁

我在上一章结尾处留下了两个悬而未决的问题：如果每一种寻路算法都使用相同的代码，是什么让 A* 的行为看起来像 A* 呢？而为什么上一章的演示实例会找出不同的路径呢？

这两个问题的答案是非常相关联的。尽管算法定义得很好，但有一个方面我故意含糊不清，事实证明，它是解释搜索算法行为的关键：

``` test
node = choose_node(reachable)
```

那些看起来很无辜的线使得每个搜索算法都与众不同。`choose_node` 所作的选择会让世界看起来不同。

那么为什么上一章的演示实例会找出不同的路径呢？答案是因为 `choose_node` 方法随机选择了一个节点。

<!--more-->

## 长度很重要

在深入研究 `choose_node` 函数的不同行为之前，根据前面的解释，我们需要对算法中存在的一点疏忽进行修正。

每当我们考虑与当前节点的相邻节点是，我们忽略了那些我们已经知道如何到达的节点：

``` test
if adjacent not in reachable:
    adjacent.previous = node  # 记住我们是如何抵达这个节点的
    reachable.add(adjacent)
```

这里犯了一个错误：如果我们刚好发现一个更好的到达此节点的方式呢？在这种情况下，我们应该应该调整这个节点的前驱节点来使到达此节点的路径更短。

为此，我们需要知道从起始节点到任何可达的节点之间的路径长度，我们称这种长度为 `cost`。现在让我们假设从一个节点移动到一个相邻节点的固定花费（cost）为 1。

在开始搜索之前，我们先设置每个节点的 `cost` 为值 `infinity`，这使得任何路径都比这个值要短。我们还需要设置到达 `start_node` 节点的 `cost` 为 0。

所以现在代码看起来是这样子：

``` test
if adjacent not in reachable:
    reachable.add(adjacent)

# 如果这是一条新路径，或者比现有的路径更断，则保留它。
if node.cost + 1 < adjacent.cost:
    adjacent.previous = node
    adjacent.cost = node.cost + 1
```

## 统一的搜索成本

现在让我们关注 `choose_node`(选择节点) 方法。如果我们想要得到尽可能短的路径，随机选择一个节点显然不是一个很好的主意。

一个更好的方法是选择从开始节点可以到达最短路径的节点。这通常会选择较短的路径，而不是较长的路径。当然，这并不意味着不考虑较长的路径，而是优先考虑较短的路径。因为一旦找到有效路径，算法就会停止，这应该就是我们的较短路径。

这里是一种可能的 `choose_node` 函数实现：

``` test
function choose_node (reachable):
    best_node = None

    for node in reachable:
        if beat_node == None or best_node > node.cost:
            best_node = node

    return best_node
```

直观的说，该算法的搜索从开始节点“径向”展开，直到抵达结束节点。下面是这种搜索行为的Demo演示:

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
        if (node.cost < min_cost) {
            min_cost = node.cost;
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

在选择下一个要考虑的节点方式上做一个非常简单的改变，就可以得到一个非常好的搜索算法：找到从开始节点到目标节点的最短路径。

在某种程度上，这仍然是个愚蠢的做法。算法会一直漫无目的的寻找，直到它偶然发现了目标节点。在上面的例子中，当我们在朝着 A 节点方向搜索时却距离目标节点越来越远，这有什么意义呢？

我们能让 `choose_node` 方法更加智能吗？即使事先不知道正确的方向，也能让它朝着目标不断迈进。

事实证明我们是可以做到的，在下一篇文章中，我们最终会得到更加完善的 `choose_node` 方法，它会使常规的寻路算法变成 A*。

---

* 系列文章目录：
  * [寻路算法揭秘（1）：通用搜索](/2019/02/18/pathfinding-demystified-01/)
  * [寻路算法揭秘（2）：搜索策略](/2019/02/20/pathfinding-demystified-02/)
  * [寻路算法揭秘（3）：A* 解密](/2019/02/23/pathfinding-demystified-03/)
  * [寻路算法揭秘（4）：实用 A* 算法](/2019/02/28/pathfinding-demystified-04/)

* [翻译原文](http://www.gabrielgambetta.com/generic-search.html)