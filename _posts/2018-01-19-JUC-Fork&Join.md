---
layout:     post
title:      Fork&Join
date: 2018-01-19
category:   JUC
tags:   [JUC]
---

## Fork/Join框架
是Java7提供了的一个用于并行执行任务的框架， 是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。
## 工作窃取（work-stealing）算法
是指某个线程从其他队列里窃取任务来执行    
假如我们需要做一个比较大的任务，我们可以把这个任务分割为若干互不依赖的子任务，为了减少线程间的竞争，于是把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应，比如A线程负责处理A队列里的任务。但是有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有任务等待处理。干完活的线程与其等着，不如去帮其他线程干活，于是它就去其他线程的队列里窃取一个任务来执行。而在这时它们会访问同一个队列，所以为了减少窃取任务线程和被窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。  
## 框架介绍
第一步分割任务。首先我们需要有一个fork类来把大任务分割成子任务，有可能子任务还是很大，所以还需要不停的分割，直到分割出的子任务足够小。  

第二步执行任务并合并结果。分割的子任务分别放在双端队列里，然后几个启动线程分别从双端队列里获取任务执行。子任务执行完的结果都统一放在一个队列里，启动一个线程从队列里拿数据，然后合并这些数据。  

-  ForkJoinTask：我们要使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务中执行fork()和join()操作的机制，通常情况下我们不需要直接继承ForkJoinTask类，而只需要继承它的子类，Fork/Join框架提供了以下两个子类：
   - **RecursiveAction**：用于没有返回结果的任务。
   - **RecursiveTask** ：用于有返回结果的任务。
- ForkJoinPool ：ForkJoinTask需要通过ForkJoinPool来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。

```Java
public class ForkandJoinTest {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ForkJoinPool p=new ForkJoinPool();
        Future future=p.submit(new calBigSum(0,1000000000));
        System.out.println(future.get());


    }
}

class calBigSum extends RecursiveTask<Long>{


    long limit=1000;
    long start,end;

    public calBigSum(long start, long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if(end-start<10000){
            long sum=0;
            for (long i =start ; i <=end ; i++) {
                sum+=i;
            }
            return sum;
        }
        else {
            long mid=(start+end)/2;
            calBigSum left=new calBigSum(start,mid);
            calBigSum right=new calBigSum(mid+1,end);

            //fork就是去执行下面这个任务，其实就是执行compute这个函数
            left.fork();
            right.fork();

            //join等待fork的结果并返回
            return left.join()+right.join();
        }
    }
}
```
