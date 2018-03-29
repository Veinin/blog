---
title: Java8编程：Streams API
date: 2017-05-26 00:32:05
categories: Java
tags: [Java, Streams API]
---

在Java日常编程中，我们使用的最多的API可能是集合了，集合几乎在所有的单元模块中都会出现。而如果使用集合就必须对集合进行处理，往往开发人员可能需要使用循环去进行重复检查处理。为了简化流程，我们使用SQL查询语句进行数据分组来的更加简单。例如：  

```
SELECT name FROM person WHERE age > 20;
```

上诉表达式可以快速的帮你找出年龄大于20的人，但当数据量大的时候，类似的处理效率又成了问题，有些开发人员会想到使用多核进行数据处理，缺乏相关经验的Java开发人员却是非常容易编写出错百出的并行处理代码的。  
流是Java 8引入的一组新API，我们可以使用流像使用SQL语句一样声明性的方式的进行数据处理。此外，使用流还可以在无须编写额外多线程代码的情况下透明的并行处理。

<!--more-->


## 简介
什么是流？流不是数据结构，也不保存数据，流只是一些了的算法和计算的操作序列。其定义包含以下几种元素：

- **元素序列**，以顺序方式提供给流的一组数据，流通过这组数据进行处理计算。
- **源**，提供给流的一个数据源，如集合、数组或I/O资源。
- **聚合操作**，流支持使用顺序或并行进行一系列诸如筛选、查找、匹配、分组、截取的聚合元素操作。
- **流水线**，很多流操作会返回另外一个流，这些操作组合起来形成一个流水线。
- **自动迭代**，流在元素上面的迭代操作是*内部迭代*进行的，*流只能被遍历一次*。与其相反，我们常用的如`for-each`操作被称为外部迭代。

本文大部分例子都是一系列的Person类集合操作，代码清单如下：

```
public class Person {
	private final String name;
	private final boolean married;
	private final int age;
	private final Sex sex;

	public Person(String name, boolean married, int age, Sex sex) {
		this.name = name;
		this.married = married;
		this.age = age;
		this.sex = sex;
	}

	public String getName() {
		return name;
	}

	public boolean isMarried() {
		return married;
	}

	public int getAge() {
		return age;
	}
	
	public Sex getSex() {
		return sex;
	}

	public enum Sex {MALE, FEMALE}
	public enum AgeDistribution {YOUTH, MIDDLE, ELDERLY}
}

List<Person> persons = Arrays.asList(
	new Person("Mathews", false, 10, Sex.Male),
	new Person("Silvia", true, 30, Sex.FEMALE),
	new Person("Veinin", false, 25, Sex.Male),
	new Person("Bales", true, 60, Sex.FEMALE),
	new Person("Baldry", true, 40, Sex.FEMALE),
	new Person("Sims", true, 55, Sex.Male),
);
```


## 流操作
java.util.stream.Stream接口定义了许多流操作方法，我们把他们分为两大类：

- **中间操作流**，操作完后会返回另一个流`Stream<T>`，可以跟其他流处理操作连接起来。连接起来的中间操作不会立即执行。
| 操作     | 操作参数       | 函数描述符   |
| :--: | :: | :--: |
| filter   | Predicate<T>   | T -> boolean |
| distinct | | |
| limit    | | |
| skip     | | |
| map      | Function<T, R> | T -> R       |
| sorted   | Comparator<T>  | (T, T) -> int|
- **终端操作流**，执行所有流水线操作，并关闭流操作生成结果，其结果不是任何流的值，比如Integer、List、Map、Void等。
| 操作    | 目的                                           |
| :-: | :: |
| forEach | 消费流中的元素，应用于制定Lambda操作，返回void |
| count   | 返回流操作结果的个数                           |
| collect | 把流操作结果归纳成一个集合                     |
 
**中间操作**与**终端操作**结合后，看起来会是这样子：

```java
long count = persons.stream()
    .filter(Person::isMarried)  // 中间操作
    .limit(3)                   // 中间操作
    .count();                   // 终端操作
```

上面的流操作包含以下元素：
- **数据源**，persons集合
- **中间操作链**，filter与limit构成一条流水线
- **终端操作**, 执行流水线，并调用count生成结果。


## 构建流
创建一个流有许多种方式，大部分需要流操作的对象都有提供构建流的API。但总体来说可以归纳成以下几种：
- **通过集合生成流**，Collection接口提供了一个默认方法 `stream()` 用来返回一个流对象 `Stream<T>` ,如果需要并行处理数据，可以通过 `paralleStream()` 返回一个并行流。
- **通过值创建流**，`Stream` 接口中提供了 `Stream.of(T)` 静态方法，也可以通过 `Stream.empty()` 返回一个空的流对象。
- **通过数组创建流**，静态方法 `java.util.Arrays.stream()` 可以从数组中创建一个流。如 `IntStream stream = Arrays.stream(new int[]{1, 2, 3})` 。
- **通过文件生成流**， `java.nio.file.Files` 中提供了多个静态方法可以从文件中生成一个流。
- **创建无限流**，所谓无限流，是指不像上面的流创建方法从指定大小的数据源中得到一个流，它通过给定函数创建一个值，并可以永久的执行下去不断产生新值，一般来说我们通过 `limit(n)` 来限制这种流。`Stream` 提供了两个静态方法： `Stream.iterate` 和 `Stream.generate` 来产生一个无限流。

```
// 迭代，从0开始，对每次生产的整数n做加1运算，生产10个数后输出。
Stream.iterate(0, n -> n + 1)
    .limit(10)
    .forEach(System.out::println);

// 生成，接受一个Supplier<T>类型的Lambda表达式来提供新值，生产5个值后打印输出
Stream.generate(Math::random)
    .limit(5)
    .forEach(System.out::println);
```


## 使用流

### 筛选
`Stream` 接口提供了 `filter()` 方法接受一个谓词参数 `Predicate<T>`  ，返回一个包含所有符合谓词筛选条件的元素的流。例如，我们需要筛选所有已婚人士：

```
List<Person> marriedPersons = persons.stream()
    .filter(Person::isMarried)
    .collect(toList());
```

### 筛选各异元素
`Stream` 还提供了一个 `distinct()` 的方法，通过调用元素的 `hashCode()` 和 `equals()` 方法来实现元素各异的对比，产生一个没有重复值的流。例如，筛选列表中不重复的值：

```
List<Integer> numbers = Arrays.asList(1, 2, 1, 2, 3);
numbers.stream()
    .distinct()
    .forEach(System.out::println);  // 输出 1, 2, 3
```

### 截短流
`Stream.limit(n)`、 `Stream.skip(n)` 可以对筛选过的流元素进行截短， `limit` 和 `skip` 是互补的， `limit` 截取前n个元素，而skip则是跳过前n个元素，如果元素不足，会返回一个空流。

```
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.stream()
    .limit(2)
    .forEach(System.out::println);  // 输出 1, 2
    
numbers.stream()
    .skip(2)
    .forEach(System.out::println);  // 输出 3, 4, 5
```

### 映射
有时候我们可能需要从某组元素中提取一组特定对象，比如上Person列表中提取每个人的名字。  `Stream.map()` 方法能满足我们的需求。 `map` 方法接受一个 `Function<T, R>` Lambda表达式作为参数。

```
List<String> personNames = persons.stream()
    .map(Person::getName)
    .collect(toList());
```

### 扁平化流
流的扁平化与映射采用一对一映射关系不同，使用 `flatMap` 会将map映射时生成的单个流都合并起来。假如给你一个 `Stream<List<String>>` 流，需要生成一个 `Stream<String>` 流，并且去除重复的 `String` 元素时， `flatMap` 就能派上用场。

```
Stream<List<String>> integerListStream = Stream.of(
  Arrays.asList("Mathews", "Veinin"), 
  Arrays.asList("Veinin", "Baldry"), 
  Arrays.asList("Sims")
);

Stream<Integer> stringStream = integerListStream .flatMap(Collection::stream).distinct();
stringStream.forEach(System.out::println);
```

输出：

```
Mathews  
Veinin  
Baldry  
Sims
```

### 流的查找与匹配
对数据处理的常用功能离不开查找与匹配，Stream API提供了对流的查找匹配相关函数：
- **anyMatch**，流中是否有一个元素匹配，返回一个boolean值
- **allMatch**，流中的所有元素是否都匹配，返回一个boolean值
- **noneMatch**， 流中没有任何一个匹配的元素，返回一个boolean值
- **findAny**，返回流中的任一一个元素，返回一个Optional<T>对象。
- **findFirst**，返回流中的第一个出现的元素，返回一个Optional<T>对象。

```
persons.stream().anyMatch(Person::isMarried);       // 是否有已婚人士
persons.stream().allMatch(p -> p.getAge() < 20);    // 是否所有人都小于20岁
persons.stream().noneMatch(p -> p.getAge() > 30);   // 是否没有一个大于30岁的人

// 找出任一一个已婚人士，如果有则输出
persons.stream()
    .filter(Person::isMarried)
    .findAny()
    .ifPresent(System.out::println);
    
// 找出元素序列中的第一个人，如果找到则输出
persons.stream()
    .findFirst()
    .ifPresent(System.out::println);
```

> 关于`Optional`类
`Optional` 是Java8新引入的一个类，它是一个可以为null值的容器对象。如果值存在， `isPresent()` 将会返回 `true`，并可以通过 `get()` 方法获取到这个值。
关于 `Optional`的设计思想在 [Google Guava](https://github.com/google/guava/blob/d9f088a61c80fa47d09ccf05a8c3608a47c5e4c0/guava/src/com/google/common/base/Optional.java) 代码库中其实早就已经存在了。当我们在调用函数后，其返回值我们无法判断是否为null时，就可以返回一个 `Optional<T>` 对象，来代替你的返回值，提示调用者，此返回值可能为空。其使用语法大致为这样：
```
Optional<String> name = person.getName();
if (name.isPresent()) {
    System.out.println(name.get());
}
```

### 归纳
`map` 操作是将一组元素映射成一组新的值，而 `reduce` 操作则是把这些映射过的元素进行组合操作，通过指定运算规则产生另一个结果。如计算一组数值的平均值、最大、最小值，这些操作都可以归类的`归纳`操作。常用的 `归纳` 操作包括：sum、min、max、average、count。
比如，我们需要对所有人计算年龄和、最大年龄、最小年龄：

```
int totolAge = persons.stream()
    .map(Person::getAge)
    .reduce(0, Integer::sum);
    
Optional<Integer> maxAge = persons.stream()
    .map(Person::getAge)
    .reduce(Integer::max);

Optional<Integer> minAge = persons.stream()
    .map(Person::getAge)
    .reduce(Integer::min);
```

`reduce` 接受两个参数：

- 初始值
- 计算用的lambda表达式，类型为BinaryOperator<T>，讲两个元素结合起来，产生一个新值。



### 数值流

#### 基本数据类型操作流
对基本数据类型的装箱、拆箱操作是非常耗时的操作。Stream API提供了3种流接口来解决这个问题：`IntStream`、`LongStream`、`DoubleStream`。
上面的`归纳`操作我们可以使用原始数据类型流来优化：

```
int totalAge = persons.stream()
    .mapToInt(Person::getAge)     // 返回一个 IntStream
    .sum();

OptionalInt maxAge = persons.stream()
    .mapToInt(Person::getAge)
    .max();
```

#### 数值范围生成流
有时我们需要生成一窜制定范围内的数值，并进行相关操作，比如对1-100范围内的数求和，对于这种操作我们可以使用 `range` 和 `rangeClosed`， `range` 对于 `rangeClosed` 来说， 前者的结束值将不被包含。

```
int total = IntStream.range(1, 100).sum();          // 1-99  数值求和
int total = IntStream.rangeClosed(1, 100).sum();    // 1-100 数值求和
```

### 分组
分组是一个常见的数据库操作，在Java 8之前，我们用代码对数据分组很麻烦，并且容易出错。但如果使用Java 8提供的函数式接口 `Collectors.groupingBy` 这将变得很简单。如我们将人的性别进行分组，传统的Java操作看起来是这样的：

```
Map<Sex, List<Person>> personBySex = new HashMap<>();
for (Person person : persons) {
    List<Person> personByList = personBySex.get(person.getSex());
    if (personByList == null) {
        personByList = new ArrayList<>();
        personBySex.put(person.getSex(), personByList);
    }
    personByList.add(person);
}
```

使用 `Collectors.groupingBy` 后，我们的代码将大大简化：

```
Map<Sex, List<Person>> personBySex = persons.stream().collect(groupingBy(Person::getSex));
```

有时候，还需要进一步进行多级分组，如除了性别外，我们还需要对年龄分布进行分组，把人分为年轻组、中年组和老年组：

```
Map<Sex, Map<AgeDistribution, List<Person>> personBySex = persons.stream().collect(
    groupingBy(Person::getSex),
        groupingBy(person -> {
            if (person.getAge <= 30) {
                return AgeDistribution.YOUTH;
            } else if (person.getAge <= 50) {
                return AgeDistribution.MIDDLE;
            } else {
                return AgeDistribution.ELDERLY;
            }
        })
    );
```

### 分区
分区是分组的一种特殊处理方式，分区函数讲返回一个boolean值，并把元素分为两类（true or false），存储于 `Map<Boolean, T>` 当中。`Collectors.partitioningBy` 提供了这种分区功能，如我们需要对已婚、未婚人士分区时：

```
Map<Boolean, List<Person>> partitionedPerson = persons.stream()
    .collect(partitioningBy(Person::isMarried));
    
List<Person> marriedPersons = partitionedPerson.get(true);      // 获得已婚组
```

### 并行流
并行流是把一个问题分解成多个子问题，通过多个线程分别处理每个子问题的流。在处理问题的时候，使用并行流，可以充分利用多核CPU的优势，让任务分摊到每个CPU，让所有CPU都忙起来。

求和运算就是个很好的利用并行处理的例子，传统的Java代码看起来是这样的：

```
public static long sum(long n) {
    long result = 0;
    for (int i = 1; i <= n; i++) {
        result += i;
    }
    return result;
}
```

通过流，我们可以简化其操作：

```
public static long sequentialSum(long n) {
    return LongStream.rangeClose(1L, n)
        .reduce(0L, Long::sum);
}
```

使用parallel()方法，我们可以把上面的顺序计算的流转换成并行计算的流：

```
public static long parallelSum(long n) {
    return LongStream.rangeClose(1L, n)
        .parallel()
        .reduce(0L, Long::sum);
}
```

### 高效的使用并行流
并不是说任何流操作，当使用并行的时候都会提升性能的，相反，如果使用不当，并行流效率将会大打折扣，甚至效率也会更加底下。例如如果我们使用 `Stream.iterator` 进行累计并行计算：

```
public static long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
        .limit(n)
        .parallel()
        .reduce(0L, Long::sum);
}
```

上面代码运行效率和传统的for循环对比，运行时间可能要慢上几倍，究其原因是因为 `iterate` 生成的是进行过**装箱操作**的对象，`iterate` 操作原理也很难使其分成多个子任务来单独运行。

对于是否并行流操作，可以先考虑一下几个问题：

- 留意装箱操作，自动装箱、拆箱操作性能将大大降低。  
- 有些操作顺序流天生就比并行流要快，如limit和findFirst操作都依赖于元素顺序。  
- 对于较小的数据量，并行流不是一个好的方式。  
- 考虑流背后的数据结构是否利于分解，如ArrayList拆分效率比LinkedList要高很多。 
- 考虑流水线的中间操作改变分解、合并过程后是否会降低性能。


## 总结

- 一个流操作包含数据源、中间操作链和终端操作。
- 我们可以通过值、集合、数组、文件以及 `iterate` 和 `generate` 生成一个流。
- 使用 `filter` 、 `distinct` 、 `limit` 、 `skip` 对流进行筛选和切片。
- 使用 `map` 和 `reduce` 进行映射和归纳操作。
- 使用 `anyMatch` 、 `allMatch` 、 `noneMatch` 进行流匹配操作，使用 `findAny` 和 `findFirst` 进行流查找操作。
- 对于需要装箱、拆箱的流操作，我们可以使用 `IntStream` 、 `DoubleStream` 、 `LongStream` 。
- 可以用 `groupingBy` 对流元素进行分组、用 `partitioningBy` 进行分区。
- 通过 `parallel` 我们可以很容易让一个流操作并行化，但是否选择并行流，我们需要考虑很多因素。


## 参考
- [Java 8 in Action](https://book.douban.com/subject/25912747/)
- [java2s](http://www.java2s.com/Tutorials/Java/Stream_How_to/index.htm)
- [stackoverflow](http://stackoverflow.com/questions/22382453/java-8-streams-flatmap-method-example/26106616)