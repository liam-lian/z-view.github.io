---
layout:     post
title:      Semaphore.md
category:   JUC
tags:   [JUC]
---

## 信号量(Semaphore)
信号量是一个非负整数，所有通过它的线程都会将该整数减一（通过它当然是为了使用资源），当该整数值为零时，所有试图通过它的线程都将处于等待状态。在信号量上我们定义两种操作： Wait（等待） 和 Release（释放）。 当一个线程调用Wait（等待）操作时，它要么通过然后将信号量减一，要么一直等下去，直到信号量大于一或超时。Release（释放）实际上是在信号量上执行加操作，加操作实际上是释放了由信号量守护的资源。  
  
  Semaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源

---
方法
- int availablePermits() ：返回此信号量中当前可用的许可证数。
- int getQueueLength()：返回正在等待获取许可证的线程数。
- boolean hasQueuedThreads() ：是否有线程正在等待获取许可证。
- void reducePermits(int reduction) ：减少reduction个许可证。是个protected方法。
- Collection getQueuedThreads() ：返回所有等待获取许可证的线程集合。是个protected方法。
**重要的**
- acquire()
- release()

```
public class SemaphoreTest {


    public static void main(String[] args) {

        ExecutorService executorService= Executors.newFixedThreadPool(20);
        Semaphore semaphore=new Semaphore(5);

        for (int i = 0; i < 100; i++) {
            Runnable r=()->{
                try {
                    semaphore.acquire();
                    System.out.println(semaphore.availablePermits()+":"+semaphore.getQueueLength());
                    semaphore.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
            executorService.submit(r);
        }
        executorService.shutdown();
    }
}
```

## TimeUnit
枚举类型
- MICROSECONDS  
- MILLISECONDS     
- NANOSECONDS
- SECONDS          
- MINUTES     
- HOURS      
- DAYS      
方法:
- convert(long duration, TimeUnit unit)的意思将duration这个时间转化为本对象（this）所表示的时间形式。
-  void  sleep(long timeout)
          使用此单元执行 Thread.sleep.这是将时间参数转换为 Thread.sleep 方法所需格式的便捷方法。  
          这样的可读性更强