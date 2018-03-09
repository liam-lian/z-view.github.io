---
layout:     post
title:      JCF-Stack and Queue
date:  2017-12-07
category:   JCF
tags:   [Java,JCF]
---

Java里有一个叫做*Stack*的类，却没有叫做*Queue*的类（它是个接口名字）。当需要使用栈时，Java已不推荐使用*Stack*，而是推荐使用更高效的*ArrayDeque*；既然*Queue*只是一个接口，当需要使用队列时也就首选*ArrayDeque*了（次选是*LinkedList*）。  
  
  
要讲栈和队列，首先要讲*Deque*接口。*Deque*的含义是“double ended queue”，即双端队列，它既可以当作栈使用，也可以当作队列使用。下表列出了*Deque*与*Queue*相对应的接口：

| Queue Method | Equivalent Deque Method | 说明 |
|--------|--------|--------|
| `add(e)` | `addLast(e)` | 向队尾插入元素，失败则抛出异常 |
| `offer(e)` | `offerLast(e)` | 向队尾插入元素，失败则返回`false` |
| `remove()` | `removeFirst()` | 获取并删除队首元素，失败则抛出异常 |
| `poll()` | `pollFirst()` | 获取并删除队首元素，失败则返回`null` |
| `element()` | `getFirst()` | 获取但不删除队首元素，失败则抛出异常 |
| `peek()` | `peekFirst()` | 获取但不删除队首元素，失败则返回`null` |

下表列出了*Deque*与*Stack*对应的接口：

| Stack Method | Equivalent Deque Method | 说明 |
|--------|--------|--------|
| `push(e)` | `addLast(e)` | 向栈顶插入元素，失败则抛出异常 |
| 无 | `offerLast(e)` | 向栈顶插入元素，失败则返回`false` |
| `pop()` | `removeLast()` | 获取并删除栈顶元素，失败则抛出异常 |
| 无 | `pollLast()` | 获取并删除栈顶元素，失败则返回`null` |
| `peek()` | `getLast()` | 获取但不删除栈顶元素，失败则抛出异常 |
| 无 | `peekLast()` | 获取但不删除栈顶元素，失败则返回`null` |

上面两个表共定义了*Deque*的12个接口。添加，删除，取值都有两套接口，它们功能相同，区别是对失败情况的处理不同。**一套接口遇到失败就会抛出异常，另一套遇到失败会返回特殊值（`false`或`null`）**。除非某种实现对容量有限制，大多数情况下，添加操作是不会失败的。**虽然*Deque*的接口有12个之多，但无非就是对容器的两端进行操作，或添加，或删除，或查看**。明白了这一点讲解起来就会非常简单。

*ArrayDeque*和*LinkedList*是*Deque*的两个通用实现，官方更推荐使用*AarryDeque*用作栈和队列  
  
    
*AarryDeque* 底层是使用一个循环数组实现的，这样才能够实现在在队列的头和队列的尾插入和删除数据
