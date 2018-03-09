---
layout:     post
title:      CopyOnWrite容器
category:   JUC
tags:   [Java, JUC]
---

CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。  
Java中的CopyOnWrite容器有：CopyOnWriteArrayList和CopyOnWriteArraySet  

CopyOnWrite并发容器用于读多写少的并发场景    

需要注意的点  
1.  减少扩容开销。根据实际需要，初始化CopyOnWriteMap的大小，避免写时CopyOnWriteMap扩容的开销。

2.  使用批量添加。因为每次添加，容器每次都会进行复制，所以减少添加次数，可以减少容器的复制次数。如使用上面代码里的addBlackList方法。  
主要存在的问题
1. 内存占用。毕竟写的时候是由两块空间的
2. 数据一致性。写入的数据不是实时的能被读到的，因为需要copy之后，修改，然后指针引用修改了之后才能读到新的数据

源码

```
写时复制，且加锁
public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```