---
layout:     post
title:      ReadWriteLock
category:   JUC
tags:   [JUC]
---
ReentrantReadWriteLock会使用两把锁来解决问题，一个读锁，一个写锁
### 线程进入读锁的前提条件：
- 没有其他线程的写锁，
- 没有写请求或者有写请求，但调用线程和持有锁的线程是同一个
### 线程进入写锁的前提条件：
 - 没有其他线程的读锁
 - 没有其他线程的写锁

就是说写-写、读-写互斥，读-读可以同时进行

```Java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReasWriteLockTest {

    public static void main(String[] args) {
        testReadWriteLock t = new testReadWriteLock();
        new Thread(()->t.write(12),"write1").start();
        new Thread(()->t.write(120),"write2").start();
        for (int i = 0; i < 10; i++) {
            new Thread(()->t.read(),"read"+i).start();
        }

    }

}

class testReadWriteLock {

    int val;
    ReadWriteLock lock = new ReentrantReadWriteLock();

    void write(int val) {

        lock.writeLock().lock();

        try {
            this.val = val;
            System.out.println(Thread.currentThread().getName());

        } finally {
            lock.writeLock().unlock();
        }

    }

    void read() {

        lock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + ":" + val);
        } finally {
            lock.readLock().unlock();
        }

    }
}

```
结果是很明确，10个读线程是可以同时获取val的值的。
