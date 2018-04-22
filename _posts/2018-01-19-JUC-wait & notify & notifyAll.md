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
/*
* 简单实现
* 基于生产这消费者模式的容器,list中可以存在的数据个数是有限的
* 当list为空,get操作阻塞,直到有数据可以取出为止
* 当list满了(达到最大的限制),put操作阻塞,直到有空位为止
* */
public class Wait_Notify {

    //使用LinkedList,基于链表,删除第一个元素更快
    final private List<Object> list = new LinkedList<>();
   //限定最大长度
    final private int MaxSize;

    public Wait_Notify(int maxSize) {
        MaxSize = maxSize;
    }

    //加锁进入函数,即每次只能有一个线程调用get或者put方法,锁加在方法上,就是获得锁才能执行方法,离开方法时自动释放锁
    //注意全文使用的都是同一把所(作用在this上的对象锁,是互斥的)
    public synchronized void put(Object item) throws InterruptedException {
        //当list满了的时候阻塞,直到收到get函数中消耗了元素之后发出的notifyAll消息为止
        while (list.size() == MaxSize) {
            //wait就是阻塞,当前的锁是作用在当前对象上的(this),调用wait函数之后,就会把持有锁释放掉,
            //之后线程进入对于this锁消息的等待队列中,直到this.notifyAll()调用之后才能唤醒
            //但是线程唤醒之后还是需要重新竞争获得锁才能继续向下执行
            //因此wait必须在获得锁之后调用
            this.wait();
        }
        list.add(item);
        //生产一个对象之后,发出通知唤醒处于等待状态的线程
        //事实上就是唤醒等在get里面的线程

        System.out.println("put--Size="+list.size());
        this.notifyAll();
    }

    public synchronized Object get() throws InterruptedException {

        /*
        * 这里使用where不能使用if,是为了避免虚假唤醒
        * 所谓虚假唤醒就是。。
        * 假设从容器为空开始：
        * Thread-A,获得锁,执行get,但是当前list为空,执行wait,释放锁,进入等待队列中.
        * 然后Thread-B,获得锁,执行get,但是当前list为空,,执行wait释放锁,也进入等待对类中.
        * 然后Thread-C,获得锁,执行put,正常执行完毕,调用了this.notifyAll()将AB两个线程唤醒
        * AB两个线程都进入Runable状态,两个线程开始竞争锁,假设B竞争到了,则B继续从刚才的this.wait()之后开始执行,消耗了一个元素,结束函数,释放锁.
        * 最后剩下A终于可以获得锁继续从this.wait()开始执行了!此时如果是if,就是直接往下走去消耗元素了,
        * 但实际上list已经空了,B应该继续等待,不应该被唤醒去往下执行,这就是虚假唤醒.
        * 改用while可以在验证一遍条件！B会继续wait
        * 因此wait必须在while中调用才安全
        * */
        while (list.size() == 0) {
            this.wait();
        }
        Object val = list.remove(0);
        //生产一个对象之后,发出通知唤醒处于等待状态的线程
        //事实上就是唤醒等在set里面的线程
        this.notifyAll();
        System.out.println("get--Size="+list.size());
        return val;
    }

    //主函数测试
    public static void main(String[] args) {

        Wait_Notify container=new Wait_Notify(10);

        Runnable producer= () -> {
            try {
                for (int i = 0; i < 10; i++) {
                    container.put(100);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };

        Runnable consumer= () -> {
            try {
                for (int i = 0; i < 10; i++) {
                    container.get();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };

        //开启三个消费者三个生产者
        new Thread(producer).start();
        new Thread(consumer).start();
        new Thread(producer).start();
        new Thread(consumer).start();
        new Thread(producer).start();
        new Thread(consumer).start();
    }
}
```

