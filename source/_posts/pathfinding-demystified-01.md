---
title: 寻路算法揭秘（1）：通用搜索
date: 2019-02-18 21:13:37
categories: 多人游戏
tags: [多人游戏, 多人游戏网络同步]
---

## 介绍

寻路算法通常是游戏开发者一个比较困惑的主题。特别是 A* 算法常常令人费解，人们普遍会认为这是一种神秘的魔法。

本系列文章的目的是以一种非常清晰易懂的方式解释常规的寻路与 A* 算法，你不必误解这是一个非常困难的话题，如果解释的当，这将是非常简单明了的问题。

请注意，我们的重点将放在游戏领域的寻路，与学术化的方法不同，我们将跳过诸如深度优先（Depth-First）或广度优先（Breadth-First）搜索算法的介绍，并尽快进从0到1的介绍 A* 算法。

第一篇文章将要解释寻路的基本概念，一旦你掌握了这些基本概念，你会发现 A* 算法非常简单。

## 简单的第一步

虽然你能够将这些概念任意复杂的 3D 环境中去，但首先我们将从一个非常简单的设置开始：一个 5x5 的方格。为了方便起见，我们用大写字母标记了每一个方块。

![pd](/images/pathfinding/path1-01.png)

<!--more-->

我们要做的第一件事就是通过图形来表示环境。我不会详细介绍图表的内容，直观地说，它是由箭头连接的一组气泡。我们把气泡称为“节点”，箭头称为“边缘”。

每个节点代表一个角色可以的一种“状态”。在这种情况下，角色的状态就是它的位置，所以我们要为每个方块的网格创建一个节点：

![pd](/images/pathfinding/path1-02.png)

现在让我们添加边缘，它代表了你可以从其他的给定状态“达到”哪些状态。在下面的例子中，除了被阻挡的方格外，你可以从任何方格走到其相邻的方格：

![pd](/images/pathfinding/path1-03.png)

如果你你能从 A 节点到达 B 节点，我们可以说 B 是 A 的邻边。

注意，边缘是有方向的，我们需要一个从 A 到 B 的边，也需要一个从 B 到 A 的边。这看起来可能有点多余，但当你考虑复杂的“状态”时，就不会是这样了。例如，你可以从屋顶掉落到地面，但你不能从地面再跳到屋顶；你可以从“活着”变成“死亡”，但反过来则不行。

## 第一个例子

当我们向从 A 节点到 T 节点时，在第一步中我们可以从两个方向进行选择：走到 B 节点或走到 F 节点。

假设我们先走到 B 节点，现在你又有了两个选择：回到 A 节点或去到 C 节点。我们记得我们已经去过了 A 节点，这是一个我们已经考虑过的选择，所以没必要再选择一次（否则我们可能整天都在走 A -> B -> A -> B...），所以下一步我们将去到 C 节点。

现在我们到达了 C 节点，但是我们发现我们已经无路可走了，而再回到 B 节点是没有任何意义的。所以我们到了一个死胡同，当我们在 A 节点选择 B 节点时就不是一个好主意，也许我们应该尝试下 F 节点。

我们继续不断的重复这个过程，直到我们发现了 T 节点。我们只是通过回溯我们的步骤，重新构建了路径。当我们在 T 节点时，我们怎么来到这里的？那么终点的路径是 O -> T 吗？那我们怎么走到 O 节点？

请记住，我们实际上根本没有移动，所有的这些都只是一次思考练习。我们仍然站在 A 节点，在我们弄清楚整个路径之前，我们根本不会移动。当我们说“移动到B节点”，我们实际上是在说“想象我们到达了B节点”。

## 通用算法

**本章节是本系列文章最重要的部分**。这是你为了寻路必须理解的部分，其他（包括A*）只是一些细节而已。一旦你了解了这一部分你将得到启蒙。

这一章节当然也很简单。

让我们将上面的例子形式化成一些伪代码。

我们需要追踪我们知道的从起点可以到达的节点。开始时，我们只有一个起始节点，但是我们可以对网格进行“探索”，并搞清楚如何到达其他节点。我们把这个列表叫做 `reachable`：

``` test
reachable = [start_node]
```

另外，我们需要追踪我们已经到达过的节点，因为我们不想再次前往，我们把这部分节点列表叫做 `explored`:

``` test
explored = []
```

这里我们来到了**算法的核心部分**：在搜索的每一个步骤时，我们先选择一个我们已知可达但尚未探索过的节点，然后在此节点检索我们可以到达的新节点。一旦发现我们可以到达目标节点，我们就完成了！否则，继续寻找。

这听起来好像很简单？是的，但这还不是全部，让我们一步步的用伪代码写出来。

我们一直寻找，直到我们到达目标节点（在这种情况下，我们已经找到了从开始节点到目标节点的路径），或者直到没有其他节点可以继续探索（在这种情况下，没有从开始节点到达目标节点的路径）:

``` test
while reachable is not empty:
```

我们选择一个我们已知的但没有探索过的节点：

``` test
    node = choose_node(reachable)
```

如果我们已经直到如何到达目标节点，我们就完成了！然后只需要建立一条路径，根据当前节点的前驱(previous )节点回到开始节点：

``` test
    if node == goal_node:
        path = []
        while node != None:
            path.add(node)
            node = node.previous

        return path
```

我们多次的去往同一个节点是没有意义的，所以我们需要追踪那些已经去过的节点：

``` test
    reachable.remove(node)
    explored.add(node)
```

我们可以算出从当前节点可以到达的新节点。我们继续从当前节点的相邻节点开始，并删除我们已经探索过的节点：

``` test
    new_reachable = get_adjacent_nodes(node) - explored
```

然后取出可探索的每一个新节点：

``` test
    for adjacent in new_reachable:
```

如果我们已经探索过此节点，则忽略它。否则，把此节点加入到已经探索过的节点列表中，记录我们已经探索过它：

``` test
        if adjacent not in reachable:
            adjacent.previous = node  # Remember how we got there.
            reachable.add(adjacent)
```

找到目标节点是退出循环的一种方式。另外一种则是当可探索(reachable)的节点为空时：如果我们已经没有可检查的节点，并且没有任何一个节点是我们的目标节点，这意味着没有从开始节点到达目标节点的路径：

``` test
return None
```

然后...也就这样了。这已经是一个完整的流程了，我们把上面的代码提取到一个单独的方法中去：

``` test
function find_path (start_node, end_node):
    reachable = [start_node]
    explored = []

    while reachable is not empty:
        # Choose some node we know how to reach.
        node = choose_node(reachable)

        # If we just got to the goal node, build and return the path.
        if node == goal_node:
            return build_path(goal_node)

        # Don't repeat ourselves.
        reachable.remove(node)
        explored.add(node)

        # Where can we get from here?
        new_reachable = get_adjacent_nodes(node) - explored
        for adjacent in new_reachable:
            if adjacent not in reachable
                adjacent.previous = node  # Remember how we got there.
                reachable.add(adjacent)

    # If we get here, no path was found :(
    return None
```

下面这个函数通过前驱节点构建返回起始节点的路径：

``` test
function build_path (to_node):
    path = []
    while to_node != None:
        path.add(to_node)
        to_node = to_node.previous
    return path
```

就这样，这是每一个寻路算法的伪代码，包括 A*。

## 在线演示

下面是一个在线演示和上面算法的实现示例。`choose_node` 只是一个随机选择的节点。你可以一步步的运行它，并查看 `可达(reachable)` 和 `已探索(explored)` 节点的变化情况，以及 `前驱(previous)` 节点的连接情况。

<script src="http://www.gabrielgambetta.com/js/path_helpers.js"></script>
<table><tr><td width="201" height="201"><canvas width="201" height="201" id="demo"></canvas></td><td><p id="reachable" style="color:green; font-family:courier;">reachable = []</p><p id="explored" style="color:red; font-family:courier;">explored = []</p><p id="path" style="font-family:courier;"></p><p><input type="button" value="Restart" onclick="stopRunSearch(); search.reset();"></input> <input type="button" value="Step"onclick="stopRunSearch(); search.step();"></input> <input type="button" value="Run" onclick="runSearch();"></input></p></td></tr></table>
<script>
// Choose a random node from the reachable list.
Search.prototype.chooseNode = function() {
    return this.reachable[Math.floor(Math.random() * this.reachable.length)];
}

Search.prototype.addAdjacent = function(node, adjacent) {
    if (findNode(adjacent, this.explored) || findNode(adjacent, this.reachable)) {
        return;
    }

    adjacent.previous = node;
    this.reachable.push(adjacent);
}

// Build the grid used in the example.
// "*" represent a blocked square, " " an open one.
var graph = new Graph(["   * ",
                       " *** ",
                       "     ",
                       "* ** ",
                       "*    "]);

var stepDelay = 200;

var search = new Search(graph, "A", "T");
search.reset();

render(search);
</script>

## 总结

上述算法是每一种寻路算法的通用算法。

那么，你知道是什么使得每一种寻路算法彼此都不相同吗？为什么 `A*` 叫做 `A*`。

这里有个小提示：如果你多次运行上面的演示搜索，你会发现算法每次找到的路径并不一定总是相同。它找到了一些其他路径，但其并不是最短路径，为什么？