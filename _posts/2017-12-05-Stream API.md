---
layout:     post
title:      Java8新特性-Stream API
date:  2017-12-05
keywords:   Java8新特性
category:   java
tags:   [Java,Java8新特性]
---
stream并不是某种数据结构，它只是数据源的一种视图。这里的数据源可以是一个数组，Java容器或I/O channel等
- 调用Collection.stream()或者Collection.parallelStream()方法
- 调用Arrays.stream(T[] array)方法    
Stream的继承关系是：  
```
BaseStream
    - IntStream
    - LongStream
    - DoubleStream
    - Stream    
```
 
 虽然大部分情况下stream是容器调用Collection.stream()方法得到的，但stream和collections有以下不同：
- 无存储。stream不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，Java容器或I/O channel等。
- 为函数式编程而生。对stream的任何修改都不会修改背后的数据源，比如对stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新stream。
- 惰式执行。stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
- 可消费性。stream只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。  
  
对stream的操作分为为两类，中间操作(intermediate operations)和结束操作(terminal operations)，二者特点是：
- 中间操作总是会惰式执行，调用中间操作只会生成一个标记了该操作的新stream，仅此而已。
- 结束操作会触发实际计算，计算发生时会把所有中间操作积攒的操作以pipeline的方式执行，这样可以减少迭代次数。计算完成之后stream就会失效。
- 区分中间操作和结束操作最简单的方法，就是看方法的返回值，返回值为stream的大都是中间操作，否则是结束操作
  
中间操作：  
- void forEach(Consumer<? super E> action)
- Stream<T> filter(Predicate<? super T> predicate)  返回一个只包含满足predicate条件元素的Stream  
- Stream<T> distinct()
- Stream<T>　sorted()  使用自然比较器
- Stream<T>　sorted(Comparator<? super T> comparator)  使用自定义比较器
- <R> Stream<R> map(Function<? super T,? extends R> mapper)
- <R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)

结束操作
1. Reduce()===>stream->val
- Optional<T> reduce(BinaryOperator<T> accumulator)   基本形式
- T reduce(T identity, BinaryOperator<T> accumulator)      增加了初始值identity
- \<U\> U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator\<U\> combiner)  增加了combiner，只有并行操作才用到
2. collect()===>stream->Collection
<R> R collect(Supplier<R> supplier, BiConsumer<R,? super T> accumulator, BiConsumer<R,R> combiner)  
三个参数分别为，目标容器、如何添加（使用add还是put等）、如何合并。每次调用collect()都要传入这三个参数太麻烦，收集器Collector就是对这三个参数的简单封装,  因此简化为  
<R,A> R collect(Collector<? super T,A,R> collector)  
```
List<String> list = stream.collect(ArrayList::new, ArrayList::add, ArrayList::addAll);// 方式１
List<String> list = stream.collect(Collectors.toList());// 方式2
Set<String> set = stream.collect(Collectors.toSet()); 
//指定容器的底层实现
ArrayList<String> arrayList = stream.collect(Collectors.toCollection(ArrayList::new));// (3)
HashSet<String> hashSet = stream.collect(Collectors.toCollection(HashSet::new));// (4)
Map<String, Integer> map = stream.collect(Collectors.toMap(Function.identity(), String::length));
```
通常collect()结果就是一个Collection，一下情况会生成Map：  
- 使用Collectors.toMap()生成的收集器，用户需要指定如何生成Mapvalue。
- 使用Collectors.partitioningBy()生成的收集器，对元素进行二分区操作时用到。
- 使用Collectors.groupingBy()生成的收集器，对元素做group操作时用到。
举例子：
 ```
Map<Boolean, List<Student>> passingFailing = students.stream()
         .collect(Collectors.partitioningBy(s -> s.getGrade() >= PASS_THRESHOLD));
Map<Department, List<Employee>> byDept = employees.stream()
            .collect(Collectors.groupingBy(Employee::getDepartment));
 ```
 使用collect进行字符串的join：    
 使用Collectors.joining的收集器，有三种拼接的fangshi
 ```
Stream<String> stream = Stream.of("I", "love", "you");
String joined = stream.collect(Collectors.joining());// "Iloveyou"
String joined = stream.collect(Collectors.joining(","));// "I,love,you"
String joined = stream.collect(Collectors.joining(",", "{", "}"));// "{I,love,you}"
```
除了可以使用Collectors工具类已经封装好的收集器，我们还可以自定义收集器，或者直接调用
```
collect(Supplier<R> supplier, BiConsumer<R,? super T> accumulator, BiConsumer<R,R> combiner)
```
方法，收集任何形式你想要的信息。




