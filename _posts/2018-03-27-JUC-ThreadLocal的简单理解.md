---
layout:     post
title:     ThreadLocal的简单理解
date:      2018-03-27
category:  JUC
tags:      [JUC] 
---

## ThreadLocal的简单理解

昨天去小米MIUI部门面试开发，被问到了ThreadLocal，直接懵掉，今天赶紧回来补课。

Thread类中保存了这样一个局部变量：

```Java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

threadLocals变量是当前线程下的一个Map，Map的key是ThreadLocal实例，value就是我们放在ThreadLocal中的值。在多线程环境下。相当于是每一个线程都能够独立的拥有一个Map【因为map是Thread中的变量】。由于ThreadLocal是通过map获取值的。也就相当于是每一个线程中的ThreadLocal的局部变量的值都是相互独立的【虽然他们是同一个ThreadLocal对象，但是在get和set值的时候操作的都是map，而map是从Thread中取出来的】。

ThreadLocal的代码分析：

```Java
//在当前的线程t中创建一个map【这直接导致了每一进程中的值是独立的】
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
//返回的是Thread中的map对象
ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
}
//得到的值是通过This在map中取出来的
public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
}
//如果map为空的话，创建map，并且要设置和返回初始值
//注意initialValue是Protected方法，可以自由设置。
private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
```

1. 每个线程中都有一个自己的ThreadLocalMap类对象，可以将线程自己的对象保持到其中，各管各的，线程可以正确的访问到自己的对象。 
2. 将ThreadLocal实例作为key，将不同对象的引用保存到不同线程的ThreadLocalMap中，然后在线程执行的各处通过这个静态ThreadLocal实例的get()方法取得自己线程保存的那个对象，避免了将这个对象作为参数传递的麻烦。 

ThreadLocal本身在整个流程中就是作为Map的key的价值存在的，真正的value的值是保存在Thread类的Field中的map里面。    
因此很多时候习惯将ThreadLocal变量设置为static的，没有必要让他创建很多个，因为你不同的Thrad中即使是key一样也没有什么关系。

###  ThreadLocalMap

ThreadLocalMap是ThreadLocal的内部类，但是他的使用实在Thread中。Thread通过维护这个map实现了对于多线程局部变量的隔离。

值得注意的是，这个Map中的项Entry是弱引用的！

WeakReference是Java语言规范中为了区别直接的对象引用（程序中通过构造函数声明出来的对象引用）而定义的另外一种引用关系。WeakReference标志性的特点是：reference实例不会影响到被应用对象的GC回收行为（即只要对象被除WeakReference对象之外所有的对象解除引用后，该对象便可以被GC回收），只不过在被对象回收之后，reference实例想获得被应用的对象时程序会返回null。

当ThreadLocal对象不再使用，可以被GC的时候，对应的在线程中ThreadLocalMap里的键值对也会失效。然而ThreadLocal对象回收后，所有对于该对象的弱引用只是变成了null。ThreadLocalMap在set的时候将键值为null的Entry替换掉。在resize的时候还会把null值的key对应的value置为null，方便gc。

```Java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

###  例子

在下面的例子中，5个线程中的拥有同一个LOCAL对象,但是他们的值不同。结构如下：

Thread0---->包含Map0，Map中的key是LOCAL，val是new Integer(0)。

Thread1---->包含Map1，Map中的key是LOCAL，val是new Integer(0)。

Thread2---->包含Map2，Map中的key是LOCAL，val是new Integer(0)。

每一次的set都是将val赋予了新的对象。也就是说每一个线程拥有一个map，虽然她们的key是相同的，但是是完全独立的。	

```Java
public class TestThreadLocal implements Runnable{

    private final static ThreadLocal<Integer> LOCAL=new InheritableThreadLocal<Integer>(){
        @Override
        protected Integer initialValue() {
            return new Integer(0);
        }
    };

    @Override
    public void run()  {
        for (int i = 0; i < 20; i++) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            LOCAL.set(LOCAL.get()+1);
        }
        System.out.println(Thread.currentThread().getName()+" "+LOCAL.get());
    }

    public static void main(String[] args) {
        Runnable r = new TestThreadLocal();
        for (int i = 0; i < 5; i++) {
            new Thread(r).start();
        }
    }
}

```
