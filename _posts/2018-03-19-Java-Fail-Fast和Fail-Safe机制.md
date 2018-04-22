---
layout:     post
title:      Fail-Fast和Fail-Safe机制
date:      2018-03-19
category:   Java
tags:   [Java]
---

## Fail-Fast机制

**fail-fast 机制是java集合(Collection)中的一种错误机制。**当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。   
例如：当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；   
那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。    
就是说当一个线程遍历集合的过程中，集合发生了**被结构性修改**(主要表现在集合大小的变化)就会抛出此异常。

迭代器的快速失败行为无法得到保证，它不能保证一定会出现该错误，但是快速失败操作会尽最大努力抛出ConcurrentModificationException异常    
因此不应该为提高此类操作的正确性而编写一个依赖于此异常的程序是错误的做法，正确做法是：ConcurrentModificationException 应该仅用于检测 bug。

源码分析：

在集合框架的迭代器中定义了` protected transient int modCount = 0;`**用来表示list被结构性修改的次数**。    
在调用 next() 和 remove()时，都会执行 checkForComodification()。若 “**modCount 不等于expectedModCount**”，则抛出ConcurrentModificationException异常，产生fail-fast事件。   
其中expectedModCount实在迭代器初始化的时候赋值为当前modCount的值。因此如果在迭代过程中，发生了结构性修改，modCount 值发生变化，那么next()方法会检测到两个值不一样，抛出异常。    
迭代器的remove()方法，不会抛出异常的原因是，她在检查之前首先直接将当前的`modCount`复制给了`exceptModCount`，直接避免了异常的抛出 

### 安全失败Fail-safe

在迭代时候会在集合二层做一个拷贝，所以在修改集合上层元素不会影响下层。在java.util.concurrent下都是安全失败
