---
title: "Java stream"
date: 2021-08-16T22:40:45+08:00
# weight: 1
# aliases: ["/first"]
tags: ["Java"]
author: "drizzl4"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Java stream part"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: false
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/drizzl4/drizzl4.github.io/blob/master/content"
    Text: "Edit" # edit text
    appendFilePath: true # to append file path to Edit link
summary: "Java stream apply"
categories: ["Java"]
---

## 基础类

Dish
```java
public class Dish {
    private final String name;
    private final boolean vegetarian;
    private final int calories;
    private final Type type;
public Dish(String name, boolean vegetarian, int calories, Type type) { this.name = name;
this.vegetarian = vegetarian;
this.calories = calories;
        this.type = type;
    }
    public String getName() {
        return name;
}
    public boolean isVegetarian() {
        return vegetarian;
}
    public int getCalories() {
        return calories;
       } 
public Type getType() {
    return type;
}
@Override
public String toString() {
    return name;
}
public enum Type { MEAT, FISH, OTHER }
}
```
menu
```java
List<Dish> menu = Arrays.asList(
new Dish("pork", false, 800, Dish.Type.MEAT), 3 new Dish("beef", false, 700, Dish.Type.MEAT),
new Dish("chicken", false, 400, Dish.Type.MEAT),
new Dish("french fries", true, 530, Dish.Type.OTHER),
new Dish("rice", true, 350, Dish.Type.OTHER),
new Dish("season fruit", true, 120, Dish.Type.OTHER),
new Dish("pizza", true, 550, Dish.Type.OTHER),
new Dish("prawns", false, 300, Dish.Type.FISH), 5 new Dish("salmon", false, 450, Dish.Type.FISH) );
```

## 基础功能
### 筛选某一属性
```java
List<Dish> vegetarianMenu = menu.stream()
								.filter(Dish::isVegetarian)
                                .collect(toList());
```
### 筛选去重
```java
List<Integer> nembers = Arrays.asList(1,2,1,3,3,2,4);
numbers.stream()
       .filter(i -> i%2==0)
       .distinct()
       .forEach(System.out::println);
```
### 限制条数
limit可以用在无序流或者有序流，无序流不会按照任何顺序排序。
```java
List<Dish> dishes = menu.stream()
                        .filter(d -> d.getCalories() > 300)
                        .limit(3)
                        .collect(toList());
```
### 跳过前N个元素
```java
List<Dish> dishes = menu.stream()
                        .filter(d -> d.getCalories >300)
                        .skip(2)
                        .collect(toList());
```
## 映射
### 流的扁平化
例如["Helllo","World"]，将字母去重之后，返回["H","e","l", "o","W","r","d"]  
如果使用：  
```java
words.stream()
        .map(word -> word.splot(""))
        .distinct()
        .collect(toList());
```
map返回的其实是Stream<String[]>类型，比较的是Hello与World的不同  
如果使用：  
```java
words.stream()
	 .map(word -> word.split(""))
	 .map(Arrays::stream)
	 .distinct()
	 .collect(toList());
```
这代表，map将所有单个的字符转化为一个流，每个流中只存在一个字符  
使用flatMap得到正确结果
```java
words.stream()
	 .map(word -> word.split(""))
	 .flatMap(Array::stream)
	 .distinct()
	 .collect(Collectors.toList());
```
flatMap将会将所有单个的流合成一个流  

## 查找和匹配
使用allMatch、anyMatch、noneMatch、findFirst和findAny方法进行匹配查找
### anyMatch方法
流中是否有一个元素能够匹配给定的谓词
```java
if(menu.stream().anyMatch(Dish::isVegetarian)){
	//anymatch方法返回一个boolean类型
	System.out.println("The menu is (somewhat) vegetarian friendly!!");
}
```
### allMatch方法
流中的所有元素是否都能匹配给定的谓词  
```java
boolean isHealthy = menu.stream()
						.allMatch(d -> d.getCalories <1000);
```
### noneMatch方法
流中所有元素都无法与给定的谓词匹配  
```java
boolean isHealthy = menu.stream()
						.noneMatch(d -> d.getCalories <1000);
```
>短路求值  
       有些操作不需要处理整个流就能得到结果。例如，假设你需要对一个用and连起来的大布 尔表达式求值。不管表达式有多长，你只需找到一个表达式为false，就可以推断整个表达式 将返回false，所以用不着计算整个表达式。这就是短路。  
       对于流而言，某些操作(例如allMatch、anyMatch、noneMatch、findFirst和findAny) 不用处理整个流就能得到结果。只要找到一个元素，就可以有结果了。同样，limit也是一个 短路操作:它只需要创建一个给定大小的流，而用不着处理流中所有的元素。在碰到无限大小 的流的时候，这种操作就有用了:它们可以把无限流变成有限流。我们会在5.7节中介绍无限 流的例子。  
### findAny方法
返回流中任意元素，一般与其他流操作结合使用，则立即利用**短路**返回
```java
Optional<Dish> dish = 
	menu.stream()
		.filter(Dish::isVegetarian)
		.findAny();
```
#### 关于Optiona简单介绍
> Optiona
		isPresent()将在Optional包含值的时候返回true, 否则返回false。
		ifPresent(Consumer<T> block)会在值存在的时候执行给定的代码块。Consumer函数式接口让你传递一个接收T类型参数，并返回void的Lambda表达式。  
		T get()会在值存在时返回值，否则抛出一个NoSuchElement异常。 
		T orElse(T other)会在值存在时返回值，否则返回一个默认值。  
例如：
```java
menu.stream()
		 .filter(Dish::isVegetarian)
		 .findAny()
		 .isPresent(d -> System.out.println(d.getName));
```
### findFirst()方法
查找第一个元素，例如List中数据已经排好序，则根据短路返回第一个  
```java
List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> firstSquareDivisibleByThree =
		someNumbers.stream()
							 	 .map(x -> x*x)
								 .filter(x -> x%3 ==0)
								 .findFirst();
```
#### findfirst与findAny
> 你可能会想，为什么会同时有findFirst和findAny呢?答案是并行。找到第一个元素
在并行上限制更多。如果你不关心返回的元素是哪个，请使用findAny，因为它在使用并行流 时限制较少。  

## 归约
使用reduce操作来表达更复杂的查询，比如“计算菜单中的总卡路里”或“菜单中卡路里最高的菜是哪一个”。此类查询需要将流中所有元素反复结合起来，得到一个值，比如一个Integer。这样的查询可以被归类为归约操作 (将流归约成一个值)。用函数式编程语言的术语来说，这称为折叠(fold)，因为你可以将这个操
作看成把一张长长的纸(你的流)反复折叠成一个小方块，而这就是折叠操作的结果。

### 元素求和
```java
//求和
int sum = numbers.stream().reduce(0, (a, b) -> a+b);
//相乘
int product = numbers.stream().reduce(1,(a, b) -> a*b);
```
![image-20210819143355223](/images/image-20210819143355223.png)
Java 8中，Integer类增加sum静态方法来求和

```java
int sum = numbers.stream().reduce(0, Integer::sum);
```
**无初始值**
```java
Optional<Integer> sum = numbers.stream().reduce((a, b) -> a+b);
```
因为可能不存在值，所以返回的为Optional对象。
### 最大值和最小值
```java
//最大值
Optional<Integer> max = number.stream().reduce(Integer::max);
```
![image-20210911160446202](/images/image-20210911160446202.png)
```java
Optional<Integer> min = number.stream().reduce(Integer::min);
```
## 小总结
![image-20210819143428774](/images/image-20210819143428774.png)
## 数值流
Java 8引入三个原始类型特化流接口，IntStream、DoubleStream、LongStream. 从而避免暗中装箱的成本。  
### 映射到数值流
通过mapToInt、mapToDouble、mapToLong，例如：
```java
int calories = menu.stream().mapToInt(Dish::getCalories).sum();
```
此处mapToInt并不是返回一个Stream<Integer>,而是IntStream，最终调用IntStream接口的sum()方法。  
### 数值流到非特化流

例如：
```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
List<Integer> stream  = intStream.boxed();
```
### 默认值OptionalInt
Optional同样有特化版本：OptionalInt、OptionalDouble和OptionalLong  
```java
Optional maxCalories = menu.stream()
                            .mapToInt(Dish::getCalories)
                            .max();
//如果没有最大值，显式提供一个默认最大值
int max = maxCalories.orElse(1);
```
### 数值范围
Java 8 引入了两个用于IntStream和LongSteram的静态方法生成一段连续数字，例如1-100  
其中range是不包含结束值的，rangeClosed则包含结束值  
```java
//从1到100的50个偶数
IntStream evenNumbers = IntStream.rangeClosed(1,100);
                    .filter(n -> n%2 ==0);
```
## 构建流

### 值创建流
```java
Stream<String> str = Stream.of("Java 8", "Lambda","In","Action");
str.map(String::toUpperCase).forEach(System.out.println);
```
生成一个空流  
```java
Stream<String> str = Stream.empty();
```
### 数组创建流

使用静态方法Arrays.stream从数组创建一个流，它接受一个数组作为参数  

```java
int[] numbers = {2, 3, 4, 5, 6};
int sum = ArrayList.stream(numbers).sum();
```

### 文件生成流

```java
//筛选不同的单词的总数
long uniqueWords = 0;
//流会自动关闭
try(Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())) {
  uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" "))).distinct()
    .count();
}catch(IOException e) {
  //如果打开文件时出现异常
}
```

**注意：flatMap的使用**

### 无限流简单使用

#### 使用iterate生成

```java
Stream.iterate(0,n -> n+2)
  .limit(10)
  .foreach(System.out.println);
```

#### 使用generate生成

```java
Stream.generate(Math::random)
          .limit(5)
          .forEach(System.out::println);
```



## 用流收集数据

### 生成Map

```java
Map<Currency, List<Transaction>> transactionsByCurrencies = transactions.stream().collect(groupingBy(Transaction::getCurrency));
```

### 查找流中最大值和最小值

```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalories = menu.stream().collect(maxBy(dishCaloriesComparator));
```

### 汇总

#### 求和

```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

类似的方法还有：summingLong、summingDouble

#### 平均数

```java
int averageCalories = menu.stream().collect(averagingInt(Dish::getCalories));
```

#### 个数、总和、平均值、最大值、最小值

```java
IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
```

同样存在DoubleSummaryStatistics、LongSummaryStatistics

### 连接字符串

```java
String shortMenu = menu.stream().map(Dish::getName).collect(join());
```

如果Dish类存在一个toString方法返回菜肴名称，则不需要进行map筛选名称变量  

```java
String shortName = menu.stream().collect(join());
```

分割方法：

```java
String shortName = menu.stream().map(Dish::getCalories).collect(join(","));
```

## 分组

通过条件划分：

```java
public enum CaloricLevel  { DIEF, NORMAL, FAT}
Map<CaloricLevel, List<Dish> dishesByCaloriesLevel = menu.stream().collect(
	groupBy(dish -> {
    if(dish.getCalories()<=400) return CaloricLevel.DIEF;
    else if(dish.getCalories()<=700) return CaloricLevel.NORMAL;
    else return CaloricLevel.FAT;
  }))
```

### 多级分组

```java
public enum CaloricLevel  { DIEF, NORMAL, FAT}
Map<Dish.Type, Map<CaloricLevel, List<Dish>> dishesByTypeCaloricLevel = 
  menu.stream().collect(
		groupingBy(Dish::getType, 
              groupingBy(dish -> {
                if(dish.getCalories()<=400) return CaloricLevel.DIEF;
    						else if(dish.getCalories()<=700) return CaloricLevel.NORMAL;
    						else return CaloricLevel.FAT;
              })))
```

### 按子组收集数据

#### 1. 把收集器的结果转换为另一种类型

```java
Map<Dish.Type, Long> typesCount = menu.stream().collect(
groupingBy(Dish::getType, counting()));

//结果： {MEAT=3, FISH=2, OTHER=4}
```

此外，gorupingBy(x)是groupingBy(x, toList())的简便写法。

```java
//按类型分类，得出分组中热量最高的
Map<Dish.Type, Optional<Dish>> mostCaloricByType =
    menu.stream()
        .collect(groupingBy(Dish::getType,
                            maxBy(comparingInt(Dish::getCalories))));

//结果：{FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[pork]}
```

**注意：**

> 这个Map中的值是Optional，因为这是maxBy工厂方法生成的收集器的类型，但实际上， 如果菜单中没有某一类型的Dish，这个类型就不会对应一个Optional. empty()值， 而且根本不会出现在Map的键中。groupingBy收集器只有在应用分组条件后，第一次在 流中找到某个键对应的元素时才会把键加入分组Map中。这意味着Optional包装器在这 里不是很有用，因为它不会仅仅因为它是归约收集器的返回类型而表达一个最终可能不 存在却意外存在的值。

去掉Optional：

```java
Map<Dish.Type, Dish> mostCaloricByType = 
  menu.stream().collect(groupingBy(Dish::getType,
                                  collectingAndThen(
                                  	maxBy(comparingInt(Dish::getCalories)),
                                  Optional::get)));
//结果：{FISH=salmon, OTHER=pizza, MEAT=pork}
```

> 这个工厂方法接受两个参数——要转换的收集器以及转换函数，并返回另一个收集器。这个 收集器相当于旧收集器的一个包装，collect操作的最后一步就是将返回值用转换函数做一个映 射。在这里，被包起来的收集器就是用maxBy建立的那个，而转换函数Optional::get则把返 回的Optional中的值提取出来。前面已经说过，这个操作放在这里是安全的，因为reducing 收集器永远都不会返回Optional.empty()。

#### 2. 与**groupingBy**联合使用的其他收集器的例子

将List变为Set

```java
Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType = 
  menu.stream().collect(
		groupingBy(Dish::getType, mapping(
    		dish -> {
           if (dish.getCalories() <= 400) return CaloricLevel.DIET;
           else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
           else return CaloricLevel.FAT;
        }, toSet())));
```

如果需要设置Set的类型，可以通过toCollection  

```java
Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType = 
  menu.stream().collect(
		groupingBy(Dish::getType, mapping(
    		dish -> {
           if (dish.getCalories() <= 400) return CaloricLevel.DIET;
           else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
           else return CaloricLevel.FAT;
        }, toCollection(HashSet::new))));
```

## 分区

分区为分组的特殊情况，其会分为true和false两组  

```java
Map<Boolean, List<Dish>> partitionedMenu = 
  		menu.stream().collect(partitioningBy(Dish::isVegetarian));

//结果：{false=[pork, beef, chicken, prawns, salmon],true=[french fries, rice, season fruit, pizza]}
```

分区进阶  

```java
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = 
  menu.stream().collect(
		partitioningBy(Dish::isVegetarian, groupingBy(Dish::getType)));

//结果：{false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]},true={OTHER=[french fries, rice, season fruit, pizza]}}
```

得到热量最高  

```java
Map<Boolean, Dish> mostCaloricPartitionedByVegetarian = 
  menu.stream().collect(
		partitioningBy(Dish::isVegetarian,
                  collectingAndThen(
                    maxBy(comparingInt(Dish::getCalories)),
                  	Optional::get)));
```

## 将数字按质数和非质数分区

有需要可继续看此部分

## 收集器接口

有需要可继续看此部分



## 并行流

```java
public static long parallelSum(long n) {
  return Stream.iterate(1L, i -> i+1)
    .limit(n)
    .parallel()
    .reduce(0L, Long::sum);
}
```

并行流线程池：  

并行流内部使用了默认的ForkJoinPool，它默认的线程数量就是你的处理器的数量，这个值是由Runtime.getRuntime().available-Processors()得到的。  

可以通过：

```java
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism","12");
```

> 这是一个全局设置，因此它将影响代码中所有的并行流。反过来说，目前还无法专为某个 并行流指定这个值。一般而言，让ForkJoinPool的大小等于处理器数量是个不错的默认值， 除非你有很好的理由，否则我们强烈建议你不要修改它。

**并行流注意：**

1. 留意装箱。自动装箱和拆箱操作会大大降低性能。Java 8中有原始类型流(IntStream、 LongStream、DoubleStream)来避免这种操作，但凡有可能都应该用这些流。

2. 有些操作本身在并行流上的性能就比顺序流差。特别是limit和findFirst等依赖于元 素顺序的操作，它们在并行流上执行的代价非常大。例如，findAny会比findFirst性 能好，因为它不一定要按顺序来执行。你总是可以调用unordered方法来把有序流变成 无序流。那么，如果你需要流中的*n*个元素而不是专门要前*n*个的话，对无序并行流调用 limit可能会比单个有序流(比如数据源是一个List)更高效。

3. 对于较小的数据量，选择并行流几乎从来都不是一个好的决定。并行处理少数几个元素 的好处还抵不上并行化造成的额外开销。

4. 要考虑流背后的数据结构是否易于分解。例如，ArrayList的拆分效率比LinkedList 高得多，因为前者用不着遍历就可以平均拆分，而后者则必须遍历。另外，用range工厂方法创建的原始类型流也可以快速分解。

5. 考虑终端操作中合并步骤的代价是大是小(例如Collector中的combiner方法)。如果这一步代价很大，那么组合每个子流产生的部分结果所付出的代价就可能会超出通过并行流得到的性能提升。

   ![image-20210901231238662](/images/image-20210901231238662.png)
