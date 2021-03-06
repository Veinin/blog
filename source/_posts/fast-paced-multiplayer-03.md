---
title: 快节奏多人游戏（3）：实体插值
date: 2019-03-10 20:05:00
categories: 多人游戏
tags: [多人游戏, 多人游戏网络同步]
---

## 介绍

在本系列的第一篇文章中，我们介绍了权威服务器的概念及其防止客户欺骗的用处。然而，天真地使用这种技术可能会导致关于可玩性和响应性的潜在问题。在第二篇文章中，我们提出了客户端预测作为克服这些限制的方法。

这两篇文章的最终结果传递的一组概念与技术是允许玩家通过互联网连接到权威的服务器，并在传输延迟的情况下，让玩家以一种完全像玩单机游戏的感觉方式控制游戏中的角色。

在本文中，我们将对连接到同一服务器的其他玩家进行探索。

<!--more-->

## 服务器时间步长

在上一篇文章中，我们描述服务器的行为非常简单：它读取客户的输入，更新游戏状态，并发送结果给客户端。但当多个客户端连接时，服务器的主循环将会有所不同。

在多个客户端同时连接同一个服务器的情况下，多个客户端可能同时发送输入信息，并且速度将非常快（玩家可以快速的发送诸如按箭头按键、移动鼠标或者点击屏幕的输入操作）。服务器接受每个客户端的输入操作然后马上广播更新后的游戏世界状态将是非常消耗CPU和网络带宽的。

一个更好的方式是对接受的每个客户端输入进行排队，而不需要接受一个就马上进行处理。相反，游戏世界以一个较低的频率周期性的更新。例如每秒更新 10 次，每次更新之间产生的延迟（这种情况下为 100 毫秒），我们称之为步长。在每次更新循环的迭代中，所有未处理的客户都输入指令都将应用到游戏世界中去（处理这些指令的时间增量可能比时间步长更小一些），并将新状态广播给客户端。

总之，服务器游戏世界里面的更新步长速率是根据客户端的输入而独立存在的，并且是可以被预测的。

## 处理低频率更新

从客户的角度来看，上面提到的方案可以很容易的进程预测，客户都的预测工作因为与延迟无关，因此在相对不频繁的状态更新的中也是可以进行预测的。但是，由于客户端的低频率广播（继续使用100毫秒的示例），客户端得到的关于其他实体可能在游戏中移动的信息是非常少的。

在下面的第一个实现中，我们将在接受到服务器的状态更新后再更新角色的位置信息，但这回导致角色的运动非常不稳定，你可能看到的是每 100 毫秒玩家就瞬移一次，而不是很平滑的移动到目的地。

![fpm](/images/fpm/fpm3-01.png)

根据你正在开发的游戏类型中，有很多方法可以解决这个问题。一般来说，你的游戏实体越可预测，那么就越容易做到正确。

## 航位（Dead reckoning）推算

假设你正在制作一款赛车游戏，一辆正在快速行驶的汽车是非常容易预测的。例如，如果它以每秒100米的运行速度，一秒钟后它距离开始的地点大约是100米。

为什么只能是概略的计算呢？因为一秒钟内，汽车可能会加速或者减速，抑或是转向左侧或右侧一点，这里的关键字是“一点”。不管玩家实际在做什么，汽车的可操作性使得其在高速运行时的任何一个时间点的位置高度依赖于其之前的位置、速度和方向
。换句话说，赛车是不可能立即就旋转 180 度的。

这种情况下，我们应该如何在每 100 毫秒的步长情况下发送服务器的更新呢？客户端每收到一次其他车辆的速度和方向后，在接下来的 100 毫秒它将无法再接收到任何新的信息，但客户端却依旧需要显示车辆正在运行。最简单的解决方案是假设汽车的方向和加速度都会在 100 毫秒内保持不变，我们可以使用这个参数在本地运行汽车的物理模拟。然后，100 毫秒后，当服务器的更新到达时，再将汽车的位置进行校正。

在很多变动的因素下，这些校正的参数可能会很大，当然也可能相对较小。如果玩家确实把汽车保持在直线上并且不改变汽车的速度，那么预测的位置将和校正位置完全相同。另外，如果玩家遭受到了撞击，那么预测的位置可能会有非常大的错误。

需要注意的是航行位置推算可以应用于低速的情况，例如战舰。事实上，“航位推算”一词来源于海上航行。

## 实体插值

在某些情况下，根本无法运用航位推算，特别是在玩家的方向和速度可以可以立刻改变的情况下。例如，在 3D 射击游戏中，玩家通常会以非常高的速度进行跑动、停止和转向操作，这使得航位推算根本无法应用，因为根据玩家之前的位置和速度的数据根本无法预测接下来会发生什么。

当服务器发送权威数据时，你不能只更新玩家的位置，这样你只会让玩家每 100 毫秒只更新一次位置，这会让游戏无法继续玩下去。

你需要解决的是在你拥有的每 100 毫秒的权威位置数据里面，如何和向玩家展示中间发生了什么事情。解决这个问题的关键是展示其他玩家相对当前玩家来说的过去时间。

假设你在 t = 1000 时收到位置数据，并且你已经在 t = 900 时也收到了数据，因此你现在知道玩家在 t = 900 和 t = 1000 时的位置数据。你可以在 t = 1000 和 t = 1100 显示其他玩家从 t = 900 到 t = 1000 所做的事情。在“延迟”显示100毫秒的情况下，通过这种方式，你始终会显示用户的实际移动数据。

![fpm](/images/fpm/fpm3-02.png)

用于从 t = 900 到 t = 1000 进行插值的位置数据取决于游戏。通常插值效果以及不错了，如果觉得还不够好，你可以让服务器在每次更新时发送更详细的移动数据。例如，玩家紧随一系列的直线段，或每隔 10 ms 采样一次的位置，这样的插值看起来会更好（在你发送一些增量的小规模移动数据时，你可以针对此特殊情况对线路上的数据格式进行大量优化，因此你不太需要为此发送10倍数量级的数据）。

请注意，使用这种技术，每个玩家看到游戏世界的渲染都会略有不同，因为每个玩家虽能看到现在的自己，但看到的其他实体却是过去的行为。然而，即使对于快节奏的游戏，看到具有 100 毫秒延迟的其他实体通常也是显而易见的。

这里有一些例外，当你需要很多空间和时间精度时，例如当玩家向其他玩家射击时，由于看的是过去的其他玩家，你的目标是延迟 100 毫秒的，也就是说，你的射击的是 100 毫秒前目标！我们将在下一篇文章中讨论这个问题。

## 总结

在具有权威服务器的客户端-服务器环境中，因为不太频繁的更新与网络延迟，你仍然必须给予每个玩家显示连续性和平稳移动的假象。在本系列文章的第2部分中，我们探索了一种使用客户端预测和服务器协调来实时显示用户控制的玩家的移动方法，这可确保用户输入会立即对本地玩家产生影响，消除因为延迟而导致游戏无法继续下去的影响。

但是，游戏中的其他实体仍然是个问题。在本文中，我们探讨了两种处理它们的方法。

第一种方法，航位推算，适用于实体的位置可以从先前的实体数据（例如位置，速度和加速度）进行预估的游戏。当不满足这些条件时，此方法将不可行。

第二个方法，实体插值，当你根本无法预测实体未来的位置时，且它仅能使用服务器提供的真实实体数据时，你可以显示其他实体时在时间上稍微做一些延迟让步。实际效果是用户能看到玩家自己在当前的实际数据，但其他实体看到的却是过去。这通常会带来难以置信的无缝体验。

但是，如果不做其他事情，当需要高分辨率和时间精度时，例如对一个移动的目标进行瞄准射击，上面的做法将不可用：当客户端2渲染客户端1的位置与服务器端客户端1的位置不匹配时，这会让爆头射击变得不可用。由于没有了爆头射击功能，游戏也会变得不完整，我们将在下一篇文章中处理这个问题。

---

* 系列文章目录：
  * [快节奏多人游戏（1）：客户端与服务器架构](/2019/03/02/fast-paced-multiplayer-01/)
  * [快节奏多人游戏（2）：客户端预测与服务器协调](/2019/03/06/fast-paced-multiplayer-02/)
  * [快节奏多人游戏（3）：实体插值](/2019/03/10/fast-paced-multiplayer-03/)
  * [快节奏多人游戏（4）：延迟补偿](/2019/03/14/fast-paced-multiplayer-04/)

* [翻译原文](http://www.gabrielgambetta.com/entity-interpolation.html)