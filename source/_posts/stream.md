---
title: stream
comments: true
toc: true
description: stream
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - java
  - stream
tags:
  - stream
  - 命令
date: 2020-12-25 16:00:00
---
# stream

Java8中提供了Stream对集合操作作出了极大的简化，学习了Stream之后，我们以后不用使用for循环就能对集合作出很好的操作。

## 一、流的初始化与转换

  Java中的Stream的所有操作都是针对流的，所以，使用Stream必须要得到Stream对象：
  1、初始化一个流：
    Stream stream = Stream.of("a", "b", "c");
  2、数组转换为一个流：
    String [] strArray = new String[] {"a", "b", "c"};
    stream = Stream.of(strArray);
    或者
    stream = Arrays.stream(strArray);
  3、集合对象转换为一个流（Collections）：
    List<String> list = Arrays.asList(strArray);
    stream = list.stream();

## 二、流的操作

流的操作可以归结为几种：

### 1.遍历操作(map)

使用map操作可以遍历集合中的每个对象，并对其进行操作，map之后，用.collect(Collectors.toList())会得到操作后的集合。

**map(T -> R)**

1. 遍历转换为大写：

   ``` java
   List<String> output = wordList.stream().
        map(String::toUpperCase).
        collect(Collectors.toList());
   ```

2. 平方数：

   ``` java
   List<Integer> nums = Arrays.asList(1, 2, 3, 4);
      List<Integer> squareNums = nums.stream().
            map(n -> n * n).
          collect(Collectors.toList());
   ```
   
3. 将流中的每一个元素 T 映射为 R（类似类型转换）

   ``` java
   List<String> newList = list.stream()
   							.map(Person::getName)
   							.collect(toList());   
   ```

### 2.过滤操作(filter)

使用filter可以对象Stream中进行过滤，通过测试的元素将会留下来生成一个新的Stream。

**`filter(T->boolean):`保留 boolean 为 true 的元素**

得到其中不为空的String:

``` java
List<String> filterLists = new ArrayList<>();
filterLists.add("");
filterLists.add("a");
filterLists.add("b");
List afterFilterLists = filterLists.stream().filter(s -> !s.isEmpty()) .collect(Collectors.toList());
```

保留 boolean 为 true 的元素

``` java
保留年龄为 20 的 person 元素
list = list.stream()
            .filter(person -> person.getAge() == 20)
            .collect(toList());
 
打印输出 [Person{name='jack', age=20}]
```

### 3.循环操作

如果只是想对流中的每个对象进行一些自定义的操作，可以使用forEach：

``` java
List<String> forEachLists = new ArrayList<>();
forEachLists.add("a");
forEachLists.add("b");
forEachLists.add("c");
forEachLists.stream().forEach(s-> System.out.println(s));
```

### 4.返回特定的结果结合

limit 返回 Stream 的前面 n 个元素；skip 则是扔掉前 n 个元素:

``` java
List<String> forEachLists = new ArrayList<>();
forEachLists.add("a");
forEachLists.add("b");
forEachLists.add("c");
forEachLists.add("d");
forEachLists.add("e");
forEachLists.add("f");
List<String> limitLists = forEachLists.stream().skip(2).limit(3).collect(Collectors.toList());
```

注意skip与limit是有顺序关系的，比如使用skip(2)会跳过集合的前两个，返回的为c、d、e、f,然后调用limit(3)会返回前3个，所以最后返回的c,d,e

### 5.排序(sort/min/max/distinct)

sort可以对集合中的所有元素进行排序。max，min可以寻找出流中最大或者最小的元素，而distinct可以寻找出不重复的元素

如果流中的元素的类实现了 Comparable 接口，即有自己的排序规则，那么可以直接调用 sorted() 方法对元素进行排序，如 Stream。反之, 需要调用 sorted((T, T) -> int) 实现 Comparator 接口

**sorted() / sorted((T, T) -> int)**

1. 对一个集合进行排序：

   ``` java
   List<Integer> sortLists = new ArrayList<>();
   sortLists.add(1);
   sortLists.add(4);
   sortLists.add(6);
   sortLists.add(3);
   sortLists.add(2);
   List<Integer> afterSortLists = sortLists.stream().sorted((In1,In2)->
          In1-In2).collect(Collectors.toList());
   ```

2. 得到其中长度最大的元素：

   ``` java
   List<String> maxLists = new ArrayList<>();
   maxLists.add("a");
   maxLists.add("b");
   maxLists.add("c");
   maxLists.add("d");
   maxLists.add("e");
   maxLists.add("f");
   maxLists.add("hahaha");
   int maxLength = maxLists.stream().mapToInt(s->s.length()).max().getAsInt();
   System.out.println("字符串长度最长的长度为"+maxLength);
   ```

3. 对一个集合进行查重：

   ``` java
   List<String> distinctList = new ArrayList<>();
   distinctList.add("a");
   distinctList.add("a");
   distinctList.add("c");
   distinctList.add("d");
   List<String> afterDistinctList = distinctList.stream().distinct().collect(Collectors.toList());
   ```

   4.根据年龄大小来比较：
   
   ```java
   根据年龄大小来比较：
   list = list.stream()
              .sorted((p1, p2) -> p1.getAge() - p2.getAge())
              .collect(toList());
   ```
   
   当然这个可以简化为，推荐使用写法简单明了:
   
   ``` java
   list = list.stream()      .sorted(Comparator.comparingInt(Person::getAge))
   .collect(toList());
   ```
   
   
   
   其中的distinct()方法能找出stream中元素equal()，即相同的元素，并将相同的去除，上述返回即为a,c,d。

### 6.匹配(Match方法)

   有的时候，我们只需要判断集合中是否全部满足条件，或者判断集合中是否有满足条件的元素，这时候就可以使用	   	match方法：
   allMatch：Stream 中全部元素符合传入的 predicate，返回 true
   anyMatch：Stream 中只要有一个元素符合传入的 predicate，返回 true
   noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true

1. 判断集合中没有有为'c'的元素：

   ``` java
   List<String> matchList = new ArrayList<>();
   matchList.add("a");
   matchList.add("a");
   matchList.add("c");
   matchList.add("d");
   
   boolean isExits = matchList.stream().anyMatch(s -> s.equals("c"));
   ```

2. 判断集合中是否全不为空：

   ``` java
   List<String> matchList = new ArrayList<>();
   matchList.add("a");
   matchList.add("");
   matchList.add("a");
   matchList.add("c");
   matchList.add("d");
   boolean isNotEmpty = matchList.stream().noneMatch(s -> s.isEmpty());
   ```

   则返回的为false

### 7.列表属性求和

```java
produceNum = jsonList.stream().map(IngredientsDetailEntity::getCheckQty).reduce(BigDecimal.ZERO, BigDecimal::add);
```

   

   

   

