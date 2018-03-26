---
layout:     post
title:      wait & notify & notifyAll
category:   JUC
tags:   [JUC]
---
 
 wait与notify是java同步机制中重要的组成部分。结合与synchronized关键字使用，可以建立很多优秀的同步模型。  
 
 同步分为类级别和对象级别，分别对应着类锁和对象锁。类锁是每个类只有一个，如果static的方法被synchronized关键字修饰，则在这个方法被执行前必须获得类锁；对象锁类同。  
 
首先，调用一个Object的wait与notify/notifyAll的时候，必须保证调用代码对该Object是同步的，也就是说必须在作用等同于synchronized(obj){......}的内部才能够去调用obj的wait与notify/notifyAll三个方法，否则就会报错：
  java.lang.IllegalMonitorStateException:current thread not owner  
  
在调用wait的时候，线程自动释放其占有的对象锁，同时不会去申请对象锁。当线程被唤醒的时候，它才再次获得了去获得对象锁的权利。
所以，notify与notifyAll没有太多的区别，只是notify仅唤醒一个线程并允许它去获得锁，notifyAll是唤醒所有等待这个对象的线程并允许它们去获得对象锁，只要是在synchronied块中的代码，没有对象锁是寸步难行的。其实唤醒一个线程就是重新允许这个线程去获得对象锁并向下运行。  
  
wait和notify是实现进程之间通信的重要手段，wait和notify、notifyall都是在Object类的方法，因此是在两个进程之间的共享对象山调用。  

##### 永远在循环（loop）里调用 wait 和 notify，不是在 If 语句
否则可能会出现一个虚假唤醒的问题。下面是wait的一个通用的范式。  
就是说在满足条件的时候，应该等待，但是在多线程条件下，可能会有虚假唤醒，就是说在某一时刻满足了唤醒条件，但实际上下一时刻就不满足了，通过循环可以 进行一个二次的condition的验证
```
// The standard idiom for calling the wait method in Java 
synchronized (sharedObject) { 
    while (condition) { 
    sharedObject.wait(); 
        // (Releases lock, and reacquires on wakeup) 
    } 
    // do action based upon condition e.g. take or put into queue 
}
```

1. 你可以使用wait和notify函数来实现线程间通信。你可以用它们来实现多线程（>3）之间的通信。

2. 永远在synchronized的函数或对象里使用wait、notify和notifyAll，不然Java虚拟机会生成 IllegalMonitorStateException。

3. 永远在while循环里而不是if语句下使用wait。这样，循环会在线程睡眠前后都检查wait的条件，并在条件实际上并未改变的情况下处理唤醒通知。

4. 永远在多线程间共享的对象（在生产者消费者模型里即缓冲区队列）上使用wait。

5. 基于前文提及的理由，更倾向用 notifyAll()，而不是 notify()。

下面给出一个生产者、消费者的例子
```Java
public class Wait_Notify {

    public static void main(String[] args) {

        List<Object> list=new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            new  Thread(new procuder(list,10),"Threadprodecer"+i).start();
        }
        for (int i = 0; i < 3; i++) {
            new  Thread(new customer(list),"Threadcustomer"+i).start();
        }
    }
}


class procuder implements Runnable {
    List<Object> list;
    int max;

    public procuder(List<Object> list, int max) {
        this.list = list;
        this.max = max;
    }

    @Override
    public void run() {
        while (true) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (list) {
                while (list.size() == max) {
                    try {
                        list.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                Integer i = new Random().nextInt();
                list.add(i);
                System.out.println(Thread.currentThread().getName() + ":" + i);
                list.notifyAll();
            }
        }
    }
}

class customer implements Runnable{
    List<Object> list;

    public customer(List<Object> list) {
        this.list = list;
    }

    @Override
    public void run() {
        while (true) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (list) {
                while (list.size() == 0) {
                    try {
                        list.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + ":" + list.get(0));
                list.remove(0);
                list.notifyAll();
            }
        }
    }

}

```





