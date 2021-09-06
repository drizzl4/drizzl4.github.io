---
title: "Java Optional"
date: 2021-09-02T01:01:03+08:00
# aliases: ["/first"]
author: "drizzl4"
# author: ["Me", "You"] # multiple authors
TocOpen: false
draft: false
hidemeta: false
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
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

# weight: 1
tags: ["Java"]
categories: ["Java"]
showToc: true
comments: true
description: ""
searchHidden: false
summary: "Java Optional"
---

# Java Optional

## 使用Optional建模

```java
public class Person {
  //人可能有车，也可能没 有车，因此将这个字段 声明为Optional
  private Optional<Car> car;
  public Optional<Car> getCar() {return car;}
}
public class Car {
  // 车可能进行了保险，也可 能没有保险，所以将这个 字段声明为Optional
  private Optional<Insurance> insurance;
  public Optional<Insurance> getInsurance() { return insurance; }
}
public class Insurance {
  //保险公司必 须有名字
  private String name;
  public String getName() { return name; }
}
```

## Optional使用

### 创建**Optional**对象

#### 1. 声明一个空的Optional

```java
Optional<Car> optCar = Optional.empty();
```

#### 2. 依据一个非空值创建Optional

使用静态工厂方法Optional.of，依据一个非空值创建一个Optional对象:

```java
Optional<Car> optCar = Optional.of(car);
```

**如果car为null，则会抛出NullPointerException**

#### 3.可接受null的Optional

使用静态工厂方法Optional.ofNullable,允许null值的Optional对象

```java
Optional<Car> optCar = Optional.ofNullable(car);
```

### 使用map从Optional对象中提取和转换值

以前对null的判断：

```java
String name = null;
if(insurance != null) {
  name = insurance.getName();
}
```

使用Optinal提供的map方法：

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

### 使用flatMap链接Optional对象

**错误实现：**

```java
Optional<Person> optPerson = Optional.of(person);
    Optional<String> name =
        optPerson.map(Person::getCar)
                 .map(Car::getInsurance)
                 .map(Insurance::getName);
```

原因：

> optPerson是Optional<Person>类型的 变量， 调用map方法应该没有问题。但getCar返回的是一个Optional<Car>类型的对象(如代 码清单10-4所示)，这意味着map操作的结果是一个Optional<Optional<Car>>类型的对象。因 此，它对getInsurance的调用是非法的，因为最外层的optional对象包含了另一个optional 对象的值，而它当然不会支持getInsurance方法。

![image-20210906214507010](/static/images/image-20210906214507010.png)

**使用flatMap实现：**

```java
public String getCarInsuranceName(Optional<Person> person) {
  return person.flatMap(Person::getCar)
    .flatMap(Car::getInsurance)
    .map(Insurance::getName)
    .orElse("Unknown");
}
```

最终返回的为Optional<String>:

![image-20210906233028644](/static/images/image-20210906233028644.png)

### 默认行为及解引用**Optional**对象

> 1. get()是这些方法中最简单但又最不安全的方法。如果变量存在，它直接返回封装的变量 值，否则就抛出一个NoSuchElementException异常。所以，除非你非常确定Optional 变量一定包含值，否则使用这个方法是个相当糟糕的主意。此外，这种方式即便相对于 嵌套式的null检查，也并未体现出多大的改进。
>
> 2. orElse(T other)是我们在代码清单10-5中使用的方法，正如之前提到的，它允许你在 Optional对象不包含值时提供一个默认值。
>
> 3. orElseGet(Supplier<? extends T> other)是orElse方法的延迟调用版，Supplier 方法只有在Optional对象不含值时才执行调用。如果创建默认值是件耗时费力的工作， 你应该考虑采用这种方式(借此提升程序的性能)，或者你需要非常确定某个方法仅在 Optional为空时才进行调用，也可以考虑该方式(这种情况有严格的限制条件)。
>
> 4. orElseThrow(Supplier<? extends X> exceptionSupplier)和get方法非常类似， 它们遭遇Optional对象为空时都会抛出一个异常，但是使用orElseThrow你可以定制希 望抛出的异常类型。
>
> 5. ifPresent(Consumer<? super T>)让你能在变量值存在时执行一个作为参数传入的 方法，否则就不进行任何操作。

### 两个**Optional**对象的组合

假设有这样一个方法，它接受一个Person和一个Car对象，并以此为条件对外

部提供的服务进行查询，通过一些复杂的业务逻辑，试图找到满足该组合的最便宜的保险公司:

```java
public Insurance findCheapestInsurance(Person person, Car car) { 
  // 不同的保险公司提供的查询服务
  // 对比所有数据
  return cheapestCompany;
}
```

安全版本：

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(
Optional<Person> person, Optional<Car> car) {
        if (person.isPresent() && car.isPresent()) {
          return Optional.of(findCheapestInsurance(person.get(), car.get())); 				} else {
    			return Optional.empty();
				}
}
```

这样写的话，跟之前的判断null很相似，可以通过前面的map和flatMap方法进行改造：

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(
	Optional<Person> person, Optional<Car> car) {
	return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c))); 
}
```

> 这段代码中，你对第一个Optional对象调用flatMap方法，如果它是个空值，传递给它 的Lambda表达式不会执行，这次调用会直接返回一个空的Optional对象。反之，如果person 对象存在，这次调用就会将其作为函数Function的输入，并按照与flatMap方法的约定返回 一个Optional<Insurance>对象。这个函数的函数体会对第二个Optional对象执行map操 作，如果第二个对象不包含car，函数Function就返回一个空的Optional对象，整个 nullSafeFindCheapestInsuranc方法的返回值也是一个空的Optional对象。最后，如果 person和car对象都存在，作为参数传递给map方法的Lambda表达式能够使用这两个值安全 地调用原始的findCheapestInsurance方法，完成期望的操作。

### 使用filter过滤

正常代码：

```java
Insurance insurance = ...;
if(insurance != null && "CambridgeInsurance".equals(insurance.getName())){
	System.out.println("ok");
}
```

使用Optional重构：

```java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance -> 
                   "CambridgeInsurance".equals(insurance.getName()))
  					.ifPresent(x -> System.out.println("ok"));
```

### 总结

![image-20210906235251482](/static/images/image-20210906235251482.png)

![image-20210906235300902](/static/images/image-20210906235300902.png)

## 实际使用

### 用**Optional**封装可能为**null**的值

例如：

```java
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

### 异常与**Optional**的对比

在使用Integer.parseInt(String)时，如果无法解析到对应的整数，则会抛出异常，正常可以通过try/catch解决异常  

如果使用Optional的话，可以通过实现一个工具方法的方式  

```java
public static Optional<Integer> stringToInt(String s) {
  try{
    return Optional.of(Integer.parseInt(s));
  }catch(NumberFormatException e) {
    return Optional.empty();
  }
}
```

> 我们的建议是，你可以将多个类似的方法封装到一个工具类中，让我们称之为Optiona- **11** lUtility。通过这种方式，你以后就能直接调用OptionalUtility.stringToInt方法，将
>
> String转换为一个Optional<Integer>对象，而不再需要记得你在其中封装了笨拙的 try/catch的逻辑了。

