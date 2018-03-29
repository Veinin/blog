---
title: Java 8编程：Lambda表达式
date: 2017-05-20 18:10:23
categories: Java
tags: [Java, Lambda]
---

## 简介
**Lambda**表达式是Java 8中包含的一个新的重要功能。它们提供了一种更加清晰简明的方法通过使用表达式来表示一个方法接口。将代码行为参数化，让代码更好的适应不断变化的需求，减轻程序员的工作量。它完全取代以往的匿名内部类功能，使代码更加简洁、灵活、易懂。此外，新的并发功能能让我们的代码充分利用多核优势。

<!--more-->

## 匿名内部类简化
**匿名内部类**表达式可以简单的理解为传递匿名函数的一种方式：它没有名称，只有参数列表、函数主体和返回类型。最常见的GUI编程中匿名内部类就经常出现，如使用一个EventHandler来处理Button的响应事件：

```
Button button = new Button("Send");
button.setOnAction(new EventHandler<ActionEvent>() {
    public void handle(ActionEvent event) {
        label.setText("Click!");
    }
});
```

使用Lambda表达式取代匿名内部类的话，它看起来会变成这样：

```
button.setOnAction((ActionEvent event) -> label.setText("Click!"));
```

## Lambda表达式语法
Lambda表达式通过一行代码来解决了匿名内部类的庞大性，我们可以理解为它解决了匿名内部类的“垂直为题”。

Lambda表达式由三部分组成：

``` java
    参数        箭头    主体
(int x, int y)	->    x + y
```

- **参数列表** ：这里有类型为int的x和y组成
- **箭头** ：箭头把参数与Lambda主体分隔开
- **Lambda主体** ：对x和y进行加法运算，并作为返回值返回   

Lambda基本语法可以理解为：

```
(parameters) -> expression
或
(parameters) -> {statements;}
```

使用实例：  

```
(int a, int b) -> a * b  // 组合两个值  
() -> "Veinin"  // 返回一个值
() -> { return "Veinin"; }  // 使用return
() -> new Object()  // 创建对象 
(String name) -> System.out.println(name);  // 消费一个对象
```

## 在哪里以及如何使用Lambda
我们可以在**函数式接口**中使用Lambda表达式，那么什么是函数式接口呢？函数式接口其实就是定义一个抽象方法的接口。如我们常用的Java API中的Comparator和Runnable接口:

```
public interface Comparator<T> {
    int compare(T o1, T o2);
}

public interface Runnable {
    void run();
}
```

Lambda允许你直接以内联的形式为函数接口的抽象提供实现，并把整个表达式作为函数式接口的一个实例。我们也可以用匿名内部类来实现相同的功能，但那相对来说说比较笨拙。

## Lambda使用函数式接口
为了应用不同的Lambda表达式，Java API已经为我们提供了几个函数式接口。比如我们之前熟悉的Comparable、Runnable和Callable。  
Java 8中也引入了几个新的函数式接口，他们都定义在了java.util.function中。

### Functionalinterface注解
如果我们去查询新的Java AP，会发现函数式接口通常都会带有@Functionalinterface标注。这个标注表示该接口被设计成了一个函数式接口。如果你使用@Functionalinterface定义一个接口，而它却不是函数式接口的话，编译器将会返回一个提示错误。

### Predicate
java.util.function.Predicate<T> 定义了一个test的抽象方法，它接收一个泛型T对象，并返回一个boolean值。在设计需要使用类型T对象的布尔值表达式时，我们可以使用这个接口。如判断String对象是否为空：

```
@(Blog)FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

Predicate<String> nonEmptyPredicate = (String s) -> s.isEmpty();

List<String> list = Arrays.asList("aaa", "", "bbb");
for (String str : list) {
	System.out.println(nonEmptyPredicate.test(str));
}
```

### Consumer
java.util.function.Consumer<T> 定义了一个accept的抽象方法，返回值为void，顾名思义，该接口可以接受一个对象，消费对象，对其进行某些操作。如打印List里面的所有对象：

```
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

Consumer<Integer> forEach = (Integer i) -> System.out.println(i);

List<Integer> numbers = Arrays.asList(1, 2, 3);
for (Integer number : numbers) {
	forEach.accept(number);
}
```

### Function
java.util.function.Function<T, R> 接口定义了一个apply抽象方法，它接收一个参数T，并返回一个R对象。如果你输入了一个对象，并需要把对象映射到其他对象上去，则可以使用该函数接口定义一个Lambda表达式来实现你的功能。

```
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

public static <T, R> List<R> map(List<T> list, Function<T, R> f) {
	List<R> result = new ArrayList<>();
   for (T s : list) {
   		result.add(f.apply(s));
   	}
   return result;
}

List<String> strings = Arrays.asList("aaa", "bb");
List<Integer> result = map(strings, (String s) -> s.length());
```

### Supplier
java.util.function.Supplier<T> 接口定义了一个get抽象方法，不接受参数，但会返回一个T对象，我们可以把它当做一个工厂方法，返回特定对象。

```
@FunctionalInterface
public interface Supplier<T> {
    T get();
}

public static Person produce(Supplier<Person> supp) {
	return supp.get();
} 

Person person = new Person();
Person sPerson = produce(() -> person);
```

### 其他函数式接口

- **UnaryOperator**，接收一种类型参数对象，并返回相同的类型的值。描述符为: `T -> T`
- **BinaryOperator**，接收两个相同类型的参数对象，并返回一个同类型的值。描述符为：`(T, T) -> T`
- **BiPredicate**，接收两个类型参数对象，并返回一个boolean值。描述符为：`(T, U) -> boolean`
- **BiConsumer**，接收两个输入参数，没有返回值。描述符为：`(T, U) -> void`
- **BiFunction**，接收两个输入参数，产生一个结果。描述符为：`(T, U) -> R`

### 原始数据类型特化
Java类型分为两类，一种是引用类型（如Integer、String、List），另外一种是原始数据类型（如int、double、float、byte等）。涉及原始数据类型对引用数据类型的转换时，Java有一个**自动装箱**机制，比如自动将int类型转换成Integer，反之亦然。  

Java 8也特地提供了避免自动装箱操作的相应的类型函数版本。如IntPredicate可以来解决Integer的自动装箱操作：

```
@FunctionalInterface
public interface IntPredicate {
    boolean test(int value);
}

IntPredicate evenNumbers = (int i) -> i % 2 == 0;
for (int i = 0; i < 1000; i++) {  // 避免1000此自动装箱操作
	evenNumbers.test(i);
}
```

## 方法引用
方法引用可以重复使用现有的方法，并可以像Lambda一样传递他们。在某些情况下，它比起Lambda表达式来说更加易读，也更加易懂。如下面对人的年龄排序例子：

```
persons.sort((Person p1, Person p2)
	-> p1.getAge().compareTo(p2.getAge()));
```

使用方法引用和java.util.Comparator.comparing，我们可以进一步简化为：

```
persons.sort(comparing(Person::getAge));
```
方法引用可以看作为调用特定方法的Lambda的快捷写法。它的基本思想是：如果你使用一个Lambda是直接去调用一个方法，那么最好是用名称来调用，而不是去描述如何调用。使用方法引用，将使你的代码可读性更好。如Person::getAge (**注意**:不需要括号，因为没有调用此方法) 就是指明引用了Person类中定义的getAge()方法，它实际上是Lambda表达式(Person person) -> person.getAge()的快捷写法。

### 方法引用分类

- **静态方法**的方法引用 : 如Long::parseLong
- **任意类型实例方法**的方法引用 : 如Stirng::length
- **现有对象的实例方法**的方法引用 : 如person对象的一个getValue方法，写作：person::getValue

### 构造函数引用
对于构造函数，我们可以通过类名称和关键字new来指明一个构造函数引用，如Person::new。  
如通过一个无参构造函数构造一个Person对象：

```
Supplier<Person> sup = Person::new;
Person p1 = sup.get();
```

等价于

```
Supplier<Person> sup = new Person();
Person p1 = sup.get();
```

如果你的构造函数是Person(Integer age)，那么Funciton函数接口适合它：

```
Funciton<Integer, Person> sup = Person::new;
Person p1 = sup.get(12);
```

相应的如果是三个参数，我们还可以使用BiFunciton函数接口。

### 方法引用例子

```
(Person person) -> person.getAge() 简化 Person::getAge
(String s) -> s.length() 简化 String::length
(String s) -> System.out.println(s) 简化 System.out::println
```

## 复合使用Lambda表达式
Java8中提供的很多函数式接口都提供了允许复合使用Lambda表达式的方法，其复合方法都使用了default关键字标识。

### 比较器复合
在进行排序功能时，如我们需要进行逆序排序，那么我们可使用Comparator函数中提供的reversed来排序：

```
default Comparator<T> reversed() {
	return Collections.reverseOrder(this);
}

persons.sort(comparing(Person::getAge).reversed());
```

在我们进行排序的同时，如果发现相同的值时，可能需要比较第二个值来进行排序，比如人的年龄相同的话则按身高排序，我们就可以使用符合语句thenComparing来进一步操作：

```
persons.sort(comparing(Person::getAge)
	.reversed()
	.thenComparing(Person::getHeight));
```

### 谓词复合
谓词复合Predicate提供了and、or和negate三个方法，这其实是我们常用的与、或、非操作。如我们需要判断一个人：未婚、年龄在20-25岁之间且职业是程序狗或教师。

```
Predicate<Person> married = Person::isMarried;
Predicate<Person> condition = married.negate()
		.and((s) -> s.getAge() >= 20)
		.and((s) -> s.getAge() <= 25)
		.and((s) -> s.getProfession().equeas("programmer"))
		.or((s) -> s.getProfession().equeas("teacher"));

Person person = new Person(20, false, "programmer");
condition.test(person)
```

### 函数复合
Funciton接口提供了andThean和compose两个默认方法，可以把Function相关的Lambda表达式复合起来。

```
Function<Integer, Integer> f = x -> x * x;
Function<Integer, Integer> g = x -> x - 1;
Function<Integer, Integer> t = f.andThen(g);
Function<Integer, Integer> c = f.compose(g);

int result = t.apply(2); // 先调用f，然后调用g，输出3
int result = c.apply(2); // 先调用c，然后调用f，输出1
```

## 总结

- Lambda表达式可以理解为一种匿名内部类，它没有名称，只有参数列表、函数主体和返回类型。  
- 只有一个抽象方法的接口我们称之为函数式接口，Lambda表达式与函数式接口配合使用。  
- Java 8自带了很多函数式编程接口，能满足我们大部分需求，这些函数定义在java.util.function包里面。为了避免Java的装箱操作，大部分通用的函数接口都提供了针对原始数据类型的特定接口。  
- Lambda表达式可以把行为参数化，我们可以通过参数传递方法引用。  
- 函数式接口很多默认的方法促使我们可以组合Lambda表达式，进行流水线式的操作。

## 参考
- [Java 8 in Action](https://book.douban.com/subject/25912747/)
- [java2s](http://www.java2s.com/Tutorials/Java/Java_Lambda/index.htm)
- [Lambda Expressions](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/Lambda-QuickStart/index.html#overview)