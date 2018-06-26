---
layout:     post
title:      CountDownLatch&&CyclicBarrier
category:   JUC
tags:   [JUC]
---
### [CountDownLatch](http://www.importnew.com/15731.html)
CountDownLatch这个类能够使一个线程等待其他线程完成各自的工作后再执行。
例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。  

  CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。
  每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。  
  构造器中的计数值（count）实际上就是闭锁需要等待的线程数量。这个值只能被设置一次，而且CountDownLatch没有提供任何机制去重新设置这个计数值。

与CountDownLatch的第一次交互是主线程等待其他线程。主线程必须在启动其他线程后立即调用CountDownLatch.await()方法。这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。

其他N 个线程必须引用闭锁对象，因为他们需要通知CountDownLatch对象，他们已经完成了各自的任务。
这种通知机制是通过 CountDownLatch.countDown()方法来完成的；
每调用一次这个方法，在构造函数中初始化的count值就减1。
所以当N个线程都调 用了这个方法，count的值等于0，然后主线程就能通过await()方法，恢复执行自己的任务。  


### CyclicBarrier

CyclicBarrier 的字面意思是可循环使用(Cyclic)的屏障(Barrier)。
它要做的事情是，让一组线程到达一个屏障(也可以叫同步点)时被阻塞，直到最后一个线程到达屏障时，
屏障才会开门，所有被屏障拦截的线程才会继续干活。
CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，
其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，
然后当前线程被阻塞。

就是说CyclicBarrier所关联的所有的线程在调用await()方法的时候对被阻塞掉，知道每一个线程都执行了await()方法之后才被唤醒。   
这样的话，确保了一组线程能够同时进入到程序的指定位置之后才能继续执行。

```java
//构造函数
CyclicBarrier(int parties)
CyclicBarrier(int parties, Runnable barrierAction)
#  parties表示这一组线程的个数
#  barrierAction当所有线程都执行了await()方法的时候，优先调用barrierAction的执行，然后在唤醒所有的线程
#  在await()方法中检查当前的计数器的值，如果每一个线程都通过barrier，就直接执行barrierAction.run()  
#  因此需要注意barrierAction是在最后一个通过栅栏的线程中执行的。
```


实例程序
```Java
public class CyclicBarrierT {
    static class NextStop implements Runnable{

        private CyclicBarrier barrier;

        public NextStop(CyclicBarrier barrier) {
            this.barrier = barrier;
        }
        @Override
        public void run() {
            try {
                System.out.println("玩家"+Thread.currentThread().getName()+"正在第一关");
                Thread.sleep(new Random().nextInt(1500) + 500);
                System.out.println("玩家"+Thread.currentThread().getName()+"结束第一关");
                barrier.await();
                System.out.println("玩家"+Thread.currentThread().getName()+"进入第二关");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {

        CyclicBarrier barrier=new CyclicBarrier(4,()->{
            System.out.println("所有玩家都已经完成第一关,最后一个完成的是:玩家"+Thread.currentThread().getName());
        });

        for (int i = 0; i < 4; i++) {
            new Thread(new NextStop(barrier),String.valueOf(i)).start();
        }
    }
}

```


### CyclicBarrier和CountDownLatch的区别
- CountDownLatch: 一个线程,等待另外N个线程完成某个事情之后才能执行。
- CyclicBarrier: N个线程相互等待，任何一个线程完成之前，所有的线程都必须等待。
- CountDownLatch的计数器只能使用一次。而CyclicBarrier的计数器可以使用reset() 方法重置。所以CyclicBarrier能处理更为复杂的业务场景，比如如果计算发生错误，可以重置计数器，并让线程们重新执行一次。
 
就是说CountDownLatch是一个线程等待多个线程调用countDown()方法，当计数器归零就才能继续执行   
CyclicBarrier是多个线程互相等待，大家全部都被阻塞，直到所有线程都到达指定位置为止。










