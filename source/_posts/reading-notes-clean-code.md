---
title: 《代码整洁之道》读书笔记
date: 2017-02-19 11:29:48
categories: 读书笔记
tags: [代码整洁, 编程]
---

什么是代码整洁？有多少程序员，可能就会就有多少定义。  
本书以详细到吓死人的程序告诉你对代码整洁的看法。它会告诉你关于整洁变量名的想法，关于整洁函数的想法，如此等等。  
它是一本教你如何编出写好程序的书籍。读完后，我们能知道许多关于代码的事，而且，我们还能说出好代码和糟糕代码之间的差异。我们将了解如何写出好代码。我们也会知道，如何将糟糕的代码改写成好代码。  

<!--more-->



## 有意义的命名
取名字最难的地方在于需要良好的描述技巧和共有的文化背景，与其说这是一种技术、商业或管理问题，倒不如说这是一种教学问题。好名字可以提升你的代码可读性，可以让你代码更易维护和重构。

### 名副其实
选好名字要花时间，但省下来的时间比花掉的时间多。注意命名，一旦发现更好的名称，就换掉旧的。这样做，读你代码的人（包括你自己）都会开心。

变量、函数或类名应该能答复所有大问题，它该告诉你，它为什么存在，它做什么事，应该怎么用。如果名称需要用注释来补充，那就不算名副其实。如：

``` java
int d; //消失的时间，以日记
```

名称d什么都没说嘛，我们应该指明对象和计量单位的名称：

``` java
int elspedTimeInDays;
int daysSinceCreation;
int fileAgeInDays;
```

选择体现本意的名称能让人更容易理解和修改。

``` java
public List<int[]> getThem() {
	List<int[]> list1 = new ArrayList<>();
	for (int[] x : theList)
		if (x[0] == 4)
			list1.add(x);
	return list1;
}
```

看完上面代码，我们可以产生以下问题：

- 它是什么东西？
- 它有什么意义？
- 值4是什么东西？
- 它返回列表用来干嘛？

问问开发它得程序员，程序员可能会说：我们正在开发扫雷游戏，theList是单元格列表，4是一种状态，我们需要从这里获取到关于这种状态的所有值。

当然，我们可以通过询问来读懂代码，但这不值得提倡，我们可以做的更好，只需要简单改下名字，就能轻易知道发生了什么。

``` java
public List<int[]> getFlaggedCells() {
	List<int[]> flaggedCells = new ArrayList<>();
	for (int[] cell : gameBoard)
		if (cell[STATUS_VALUE] == FLAGGED)
			flaggedCells.add(x);
	return flaggedCells;
}
```

### 做有意义的区分
如果程序员只是为了满足编译器或解释器两样不同的东西，从而写代码，那就会制造麻烦。  
以数字系列命名（a1、a2，...aN），这样纯属误导，完全可以提供更加正确信息。

``` java
copyChars(char a1[], char a2[])
```

上面函数的参数名称，如果改名为source和distination，这个函数会像样很多。

### 使用可搜索的名称
单字母名称和数字常量有个问题，就是很难在一大片文字中找出来。  
找MAX_LASSES_PER_STUDEN很容易，但找数字7很难。  
同样“e”也不是一个便于搜索的好变量名，它是英文中最常用的字母，每个程序、每段代码都可能出现。由此可见，**长名称胜于短名称。搜索可得到的名称胜于胡编乱造的名称。**

### 类名和方法名
类名和对象一个是**名词或短语**，如Customer、WikiPage。类名不应该是动词。  
方法名应当是**动词或动词短语**，如postPayment、deletePage。属性访问、修改和断言都应根据其值命名，如前缀加上get、set或is。




## 函数

### 短小
函数的第一规则是**短小**，第二条规则是还要更**短小**。应该有多短小？它看起来应该是这样子：

``` java
public static String renderPageWithSetupsAndTeardowns(
	PageData pageData, boolean isSuite) {
	if (isTestPage(pageData))
		includeSeteupAndTeardownPage(pageData, isSuite);
	return pageData.getHtml();
}
```

### 只做一件事
一个函数应该**做一件事，做好这件事，只做一件事。**函数遵循单一权责原则、开放闭合原则。  
如果函数只做该函数名下的同一抽象层上的步骤，则函数就是只做了一件事。

### 每个函数一个抽象层级
要确保函数只做一件事，那么函数语句都必须在同一抽象层级上。  
让代码拥有自顶向下的阅读顺序，每个函数后面都跟着下一抽象层级的函数。把函数分为：

- **高级抽象层**
- **中级抽象层**
- **还有低等抽象层**

``` java
getHtml() // 高
String pageName = PathParser.render(pagePath)。 // 中
.append("\n") // 低
```

函数中的语句都要在同一抽象层级上。

### 函数参数
函数参数最理想的数量是零，其次是一，再次是二，应尽量避免三，有足够的特殊理由才能用三个以上参数（所以无论如何都不要这么做）。

#### 一元函数普遍形式
单个参数有两种普遍理由，你也许会为关于那个参数的问题，就像：

``` java
boolean fileExists("MyFile")
```

也可能是操作该参数，将其转换成其他什么东西，再输出之，例如：

``` java
InputStream fileOpen("MyFile")
```

#### 标识参数
标识参数丑陋不堪，向函数传入布尔值简直骇人听闻的做法。比如: `render(true)`  
对于可怜的读者来说仍然摸不着头脑，看到 render(Boolean isSuite) 也许稍有帮助，但仍不够，应该一分为二：`reanderForSuite()` 和 `renderForSingleTest()`

#### 二元函数
两个参数的函数比一元函数难懂。例如`writeField(name`比`writeField(outputStream, name`好懂。尽管两种意义都很清楚，但扫一眼就明白还是第一种。  
应尽量利用一些机制将二元函数转成一元函数，例如 `writeField` 方法写成 `outputStream` 的成员之一，比如这样： `outputStream.writeField(name`。  
当然，有些时候两个参数刚好。如 `Point(0, 0)` 笛卡尔点天生就拥有两个参数。

#### 三元函数
比二元函数难懂的多，写三元函数前一定要想清楚。
设想 `assertEquals(message, expected, actual)` 有多少次你读到message时错以为expected呢？

#### 参数对象
如果函数看来需要两个、三个或三个以上的参数，就说明其中一些参数应该封装成类了。如:

``` java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```

### 分隔指令与询问
函数**要么做什么事，要么回答什么事，但二者不可兼得。**下面是设置某个属性，返回是否成功的代码语句：

``` java
boolean set(String attribute, String value);

if (set("username", "unclebob"))...
```

从读者的角度来说，并不一定能明白**set**的含义，你可以改名命名为 `setAndCheckIfExists`，但对if帮助也不大。解决方案是把指令与询问分隔开来：

``` java
if (attributeExists("username")) {
	setAttribute("username", "unclebob");
}
```

### 抽离try/cache代码块
try/cache代码块丑陋不堪，它搞乱了代码结构，把错误处理与正常的流程混为一谈。最好把try和cache代码块的主题部分抽离出来。 **函数只做一件事，而错误处理就是一件事。**

``` java
public void delete(Page page) {
	try {
		deletePageAndAllRefreces(page)
	} cache (Exception e) {
		logError(e);
	}
}

private void deletePageAndRefrences(Page page) throws Exceptio {
}
```

### Don't repeat yourself
一次且仅一次（once and only once，简称OAOO）又称为Don't repeat yourself（不要重复你自己，简称DRY）或一个规则，实现一次（one rule, one place）是面向对象编程中的基本原则，程序员的行事准则。旨在软件开发中，减少重复的信息。
DRY的原则是──系统中的每一部分，都必须有一个单一的、明确的、权威的代表──指的是（由人编写而非机器生成的）代码和测试所构成的系统，必须能够表达所应表达的内容，但是不能含有任何重复代码。当DRY原则被成功应用时，一个系统中任何单个元素的修改都不需要与其逻辑无关的其他元素发生改变。此外，与之逻辑上相关的其他元素的变化均爲可预见的、均匀的，并如此保持同步。 -- 摘录维基百科



## 注释
请记住，唯一真正好的注释就是**你想办法不去写注释。**  
注释的恰当用法是弥补我们用代码表达意图时遭遇的失败。注意，用了“失败”一词。  
注释总是一种失败。我们总无法找到不用注释就能表达自我的方法。所以要有注释，这并不值得庆贺。

### 注释不能美化糟糕的代码
写注释的动机之一是糟糕的代码存在。我们编写一个模块，发现它令人困扰、乱起八糟，它烂透了！我们告诉自己：“喔，最好写点注释”。与其花时间编写解释你搞出来的糟糕代码的注释，还不如花时间清洁那堆糟糕的代码。

### 用代码来阐述
**能用函数或变量时，就别用注释。**
你愿意看到：

``` java
// Check to see if the employee is eligible for full benifits
if (employee.flags & HOURLY_FLAG &&employee.age > 65)
```

还是这个？

``` java
if (employee.isEligibleForFullBenefits())
```

只要你想上几秒钟，就能用代码解释你大部分的意图。很多时候，简单到需要创建一个描述与注释所言同一事物的函数即可。

### 好注释

- **法律信息**，例如版权及著作权。
- **提供信息的注释**，例如描述抽象方法的返回值。
- **警告**，警告其他情况会出现某种后果的注释。
- **// TODO 注释**，解释为什么该函数现在无所作为，将来会怎么用。
- **公共AIP注释**，标准Java库中的javadoc就是一例。

### 坏注释

- **多余的注释**，如读注释的时间比读代码花的时间还长的注释。
- **循环式注释**，所谓每个函数、每个变量都要有个注释是愚蠢可笑的。这类注释徒然让代码变得散乱，满口胡言，让人迷糊不接。
- **注释掉的注释**，其他人不敢删除注释掉的代码，他们会想，代码依然放在那儿，一定有其原因，而且这段代码很重要，不能删除。注释掉的对堆积在一起，就像破酒瓶底的残渣一样。
- **日志式注释**，冗长的日志式记录只会让模块变得凌乱不堪，版本管理软件完全可以解决。
- **废话注释**，这类注释废话连篇，当代码修改后，这类注释边做了谎言一堆。如果你在写（或粘贴）注释时都没心思，怎么指望读者从中受益。

``` java
/** The name. */
private String name;

/** The version. */
private String name;

/** The name. */ 
private String info;

/*
* Returns the day of the month
* @return the day of the month
*/
public int getDayOfMonth() {
	return dayOfMonth;
}
```



## 格式
代码格式很重要，代码格式不可忽略，必须严肃对待。代码格式关乎沟通，而沟通是专业开发者的头等大事。原始代码修改很久之后，其代码风格和可读性仍会影响到可维护性和拓展性。

### 垂直格式
源代码当个文件应该有多大？例举了市面上常见的8个开源项目，其中大多数文件为200行，最长的500行单个文件组合起来就构建了出色的系统。

#### 垂直方向区隔
几乎所有代码都是从**上往下读，从左往右读**。每行展示一个表达式或一个子句，每组代码行展示一条完整的思路。这些思路用个空白行区间隔开。
在封包声明、导入声明和每个函数直接用空白行隔开，没一个空白行都是一条线索，标识出新的独立概念。

#### 垂直距离
变量声明应该尽可能的靠近其使用位置。实体变量应该房子类的顶部或底部声明。  
若一个函数调用了另外一个函数，就应该把他们放在一起，而且调用者应该尽可能的放在被调用者上面。这样，程序就自然有序了。  
你是否曾经在某个类中摸索，从一个函数跳到另外一个函数，上下求索，想要搞清楚他们的关系，最后却被搞糊涂了？  

- 关系密切的概念应该相互靠近。
- 变量声明应该尽可能的靠近其使用位置。本地变量应该在函数顶部出现，实体变量一个在类顶部声明。

### 横向格式
遵循**无需拖动滚动条到右边**原则。虽然近年来显示器屏幕越来越宽，但并不是所以人的显示器都和你一样，普遍来说推荐屏幕单行代码长度为80上限，当然也不用死守这个数值，推荐上限不超过120个字符。

#### 水平方向空格使用
使用空格字符将彼此紧密相关的事物联系到一起，也用空格字符把相关性较弱的事物分割开。

- 赋值操作符左右边、每一个函数参数右边加上空格
- 不在函数名与左圆括号之间加空格，因为函数与其参数密切相关
- 算数运算符优先级高的不用空格，优先级低的加空格

``` java
public static double determinant(double a, double b, double c) {
    return b*b - 4*a*c;
}
```

#### 水平对其
应该对其一组声明中的变量名，或一组复制语句中的右值：

``` java
private Socket 			 socket;
private InputStream		 output;
private FileNesseContext context;
private boolean 		 hasError;
```

### 团队规则
每个程序员都有自己喜欢的规则，但如果在一个团队中的工作，就是团队说了算。  
一组开发者应当认同一种格式风格，每个成员都应该采用那种风格。我们想要软件拥有一以贯之的风格。我们不想让它显得是由一大票意见相左的个人所写成。  
好的软件系统是由一些列读起来不错的代码文件组成的。它们拥有一致和顺畅的风格。读者要能确信它们在一个源文件中看到的风格在其他文件中也是同样的用法。绝对不要用各种不同的风格来编写源代码，这样会增加其复杂度。



## 对象和数据结构
将变量设置为私有（private）有一个理由：我们不想其他人依赖这些变量。我们还想在心血来潮时能自由修改其类型或实现。

### 数据抽象
隐藏实现并不只是在变量之间放上一个函数层那么简单。**隐藏实现关乎抽象！** 类并不简单的用取值器将其变量推向外间，而是暴露抽象接口，以便用户无需了解数据结构的实现就可以操作数据**本地**。

``` java
// 具体点
public class Point {
	public double x;
	public double y;
}

// 抽象点
public interface Point {
	double getX();
	double getY();
}
```

后者代码更佳，抽象的数据存取策略让你不知道该实现是在矩形坐标系中还是在极坐标系中。我们不愿意暴露数据细节，更愿意以抽象形态表述数据。

### 数据、对象的反对称性
过程式代码：

``` java
public class Square {
	public Point topLeft;
	public double side;
}

public class Rectangle {
	public Point topLeft;
	public double height;
	public double width;
}

public static double area(Object object) {
	if (object instanceof Square) {
		Square S = (Square) object;
		return s.side * s.side;
	} else if (shape instanceOf Rectangle) {
		Rectangle r = (Rectangle) object;
		return r.height * r.width;
	}
}
```

在面向对象方案中我们只需要一个area()多态方法，如果添加新形状只需要重新实现指定接口的一个类。

``` java
public interface Shape {
	double area();
}
```

过程式代码难以添加新数据结构，因为必须修改所有函数。  
面向对象代码难以添加新函数，因为必须修改所有类。  
在任何一个复杂的系统中，有需要**添加新数据类型而不是新函数**的时候，这时对象和面向对象就比较合适。  
另一方面，当想**添加新函数而不是新数据类型**的时候，过程式代码更合适。

### 数据传送对象
最为精炼的数据结构，是一个只有公共变量，没有函数的类。这种数据结构对于数据通信传送、解析是非常有用的。在应用程序代码中“豆”（bean）结构对半封装可能会对某些OO纯化论者感觉舒服些，不过通常没有其他好处。

``` java
public class Address {
	private String street;
	
	public void Address(String street) {
		this.street = street;
	}

	public String getStreet() {
		return street;
	}
}
```

往往我们会看到开发者往这类数据结构中塞进业务规则方法，把这类数据结构当做对象来用。这不是理智的行为，业务它导致了数据结构和对象的混杂体。



## 错误处理

### 别返回null值
你是否见过几乎每行代码都在检查null值得应用程序：

``` java
public void registerItem(Item item) {
	if (item != null) {
		ItemRgister registry = peristentStore.getItemRegistry();
		if (registry != null) {
			Item existing = registry.getItem(item.getId);
			// ...
		}
	}
}
```

这种代码看似不坏，其实糟糕透了！返回null值基本在给自己增加工作量，也是在给调用者添乱。只要有一处没有检查null值，应用程序就会失控。  
例如对于空指针异常，我们可能在运行时的得到异常，也许有人在代码顶端捕获异常，也可能没有捕获。  
**如果你打算在代码中返回null值，不如抛出异常，或返回特定对象，如Java 8 引入的Optional对象。**

### 别传递null值
返回null值是很糟糕的做法，但将null值传递给其他方法就更糟糕。

``` java
public double xProjection(Point p1, Point p2) {
	return (p2.x - p1.x) * 1.5;
}
```

上面代码如果我们传入null值，将得到一个空指针异常。当然我们也可以在函数里面进行错误处理：

``` java
public double xProjection(Point p1, Point p2) {
	assert p1 != null : "p1 should not be null";
	assert p2 != null : "p2 should not be null";
	return (p2.x - p1.x) * 1.5;
}
```

看上去很美，但认为解决问题，因为还是会得到运行时异常。最恰当的做法就是禁止传入null值。



## 单元测试

###  TDD
关于TDD相关可以参考文章[《TDD并不是看上去的那么美》](http://coolshell.cn/articles/3649.html)以及文章中的相关讨论。

###  整洁的测试
有三个要素：可读性、可读性、可读性。单元测试应该和其他代码中一样：明确、简介，还有足够的表达力。测试代码应该竟可能时候的位子表达大量内容。

- 每一个测试一个断言，或者是单个测试中断言的数量应该最小化。
- 每个测试一个概念，好测试应该每个测试一个概念，而不是把所有事情都混在一起。



## 类

### 类的组织
类应该从一组变量列表开始，如果有静态变量，一个先出现。然后是私有静态变量，以及私有实体变量。很少会有公共变量。  
函数紧跟变量列表之后。公共函数调用的私有工具函数紧随该调用函数后面（自顶向下原则）。

### 类应该短小
与函数一样，类的第一原则是应该短小。第二条原则是还要更短小。  
多小合适呢？对于类来说，采用不同的衡量方法，计算**权责**。  
类名应该描述其权责。命名可以用来帮助判断类的长度。类名越含混，该类拥有的权责就越多。

#### 单一权责原则
参考[维基百科](https://zh.wikipedia.org/wiki/%E5%8D%95%E4%B8%80%E5%8A%9F%E8%83%BD%E5%8E%9F%E5%88%99)

#### 内聚
参考[维基百科](https://zh.wikipedia.org/zh-cn/%E5%85%A7%E8%81%9A%E6%80%A7_(%E8%A8%88%E7%AE%97%E6%A9%9F%E7%A7%91%E5%AD%B8)



## Kent Beck关于**简单设计**的四条原则

- 运行所有测试
- 不可重复
- 表达了程序员的意图
- 尽可能减少类和方法的数量
- 以上规则按其重要程度排列



## 总结
习艺只要有二：知和行。你一个学习得到有关原则、模式和实践的知识，穷尽应知之事，并且要对其了如指掌，通过刻苦实践掌握它。  
学写整洁代码很难。它可不止要求掌握原则和模式。你得在这上面花功夫。你须自行实践，且体验自己的失败。你须观察他人的实践与失败。你须看看别人是怎样蹒跚学步，再转头研究他们的路数。你须看看别人是如何绞尽脑汁做出决策，又是如何为错误决策付出代码。  
如果你是一个程序员，或你想成为更好的程序员，都推荐你阅读这本书。  
能编写整洁代码的程序员就像艺术家，他能用一系列变换把一块白板变作由优雅代码构成的系统。