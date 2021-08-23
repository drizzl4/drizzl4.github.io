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
![image-20210819143355223](/static/images/image-20210819143355223.png)
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
![](https://init.best/post-images/1628839778350.png)
```java
Optional<Integer> min = number.stream().reduce(Integer::min);
```
## 小总结
![image-20210819143428774](/static/images/image-20210819143428774.png)
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

