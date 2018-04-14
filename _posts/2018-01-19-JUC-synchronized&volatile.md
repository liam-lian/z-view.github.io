---
layout:     post
title:      synchronized&volatile
date:      2018-04-14
category:   JUC
tags:   [JUC]
---

synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种：
1.  修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象；
2.  修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象；
3.  修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象；
4.  修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象

---

1. 一个线程访问一个对象中的synchronized(this)同步代码块时，其他试图访问该对象的线程将被阻塞
2. .当一个线程访问对象的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该对象中的非synchronized(this)同步代码块
3. Synchronized修饰一个方法很简单，就是在方法的前面加synchronized，public synchronized void method(){//todo}; synchronized修饰方法和修饰一个代码块类似，只是作用范围不一样，修饰代码块是大括号括起来的范围，而修饰方法范围是整个函数
4. synchronized修饰的静态方法锁定的是这个类的所有对象。
5. synchronized作用于一个类T时，是给这个类T加锁，T的所有对象用的是同一把锁。


A. 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是对类，该类所有的对象同一把锁。  
B. 每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。  
C. 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。

---

volatile的作用就是1. 保证内存的可见性 2. 防止指令重排序。  
可以认为volatile关键字修饰的数据是直接在内存中处理的  
对于这个关键字无论读写，都会强制从缓存中写回或者重新从内存中读回到缓存  
这样确保多个进程中的数据的一致性。  

下面是单例模式中的双重检验的懒汉模式。
```Java
public class Singleton {
    private static volatile Singleton instance;
    private Singlton(){}
    public static Singleton getSingleton(){
        if(instance==null){
            synchronized (Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
上面的例子当中instance变量使用了volatile关键字。原因就是利用了防止指令重新排序。
因为`instance = new Singleton();`这条语句可以有一下的几个步骤：    

1. memory =allocate();    //1：分配对象的内存空间
2. ctorInstance(memory);  //2：初始化对象
3. instance =memory;     //3：instance指向刚分配的内存地址

其中2.3是可以重新排序的，如果重排序了的话，instance就不是null，但是还没有初始化，这样的话另外一个新的线程进来，会直接得到instance实例，
但是实际上得到的是半个实例，因为实例还没有初始化呢。

另外一个例子。
线程A
```Java
p=new Person();
boolean flag=true;
```
线程B
```Java
while(!flag){
  Thread.yeild();
}
p.work();
```
两个线程看似没有问题，但是线程A的两个指令可能发生重排，就是flag已经为true,但是p没有赋值，发生空指针异常。    
在这个时候应该把flag设置为volatile，就能防止指令重排。程序就不会有问题了。

## 总结
volatile 不能保证原子性 ,它本质是不是锁，而是一种保证内存可见行和防止指令重排的机制。
