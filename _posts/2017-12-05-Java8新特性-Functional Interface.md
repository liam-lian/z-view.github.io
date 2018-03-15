---
layout:     post
title:      Functional Interface
date:  2017-12-05
category:   Java8新特性
tags:   [Java8新特性]
---
### 函数式接口
FI实际上是一个接口，其中只能有一个方法（默认方法和静态方法不算）！  
Java中的lambda无法单独出现，它需要一个函数式接口来盛放，lambda表达式方法体其实就是函数接口中的那个唯一的方法的实现.    
针对与可能使用到的所有的lambda的类型，java 8中给出了几个全新的函数式接口，这样编写任何lambda表达式都事实上能够找到与之对应的FI  
  
#### Java 8中新增加的FI
- Predicate 接口只有一个参数，返回boolean类型
-  Function 接口有一个参数并且返回一个结果
- Supplier 接口返回一个任意范型的值，和Function接口不同的是该接口没有任何参数
- Consumer 接口表示执行在单个参数上的操作。
- Comparator 是老Java中的经典接口， Java 8在此之上添加了多种默认方法  
另外这些接口中还提供了一些静态方法或者是默认方法用于接口的增强，介绍如下：


#### 方法引用
如果Lambda表达式的全部内容就是调用一个已有的方法，那么可以用方法引用来替代Lambda表达式  
方法引用是lambda表达式的一种简写的形式，左边是容器（可以是类名，实例名），中间是"::"，右边是相应的方法名。如下所示：
- 如果是静态方法，则是ClassName::methodName。如 Object ::equals
- 如果是实例方法，则是Instance::methodName。如Object obj=new Object();obj::equals;
- 构造函数.则是ClassName::new