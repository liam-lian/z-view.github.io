
### Callable接口
```java
public interface Callable<V> {
    V call() throws Exception;
}
```
有返回值，并且可以抛出异常(相应的Runnable没有返回值,没有异常)
### Future接口
```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

`cancel()`方法用来取消异步任务的执行
- 如果异步任务已经完成或者已经被取消，或者由于某些原因不能取消，则会返回false
- 如果任务还没有被执行，则会返回true并且异步任务不会被执行
- 如果任务已经开始执行了但是还没有执行完成
    -   若参数为true，则会立即中断执行任务的线程并返回true，
    -   若参数g为false，则会返回true且不会中断任务执行线程。

`get()`:获取任务执行结果，如果任务还没完成则会阻塞等待直到任务执行完成。
- 如果任务被取消则会抛出CancellationException异常，
- 如果任务执行过程发生异常则会抛出ExecutionException异常，
- 如果阻塞等待过程中被中断则会抛出InterruptedException异常。

`get(long timeout,Timeunit unit)`:带超时时间的get()版本，如果阻塞等待过程中超时则会抛出TimeoutException异常

### FutureTask
FutureTask实现了RunnableFuture接口，RunnableFuture接口继承了Runnable接口和Future接口，所以FutureTask既能当做一个Runnable直接被Thread执行，也能作为Future用来得到Callable的计算结果。

构造函数：
```java
/*
*   直接构造，FutureTask中保存的是一个callable的实例
*/
public FutureTask(Callable<V> callable);
/*
 * 通过Runnable构造，通过一个适配器，将Runnable适配为Callable,返回的结果是一个死的result(不会变化)
 */
public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
}
```

FutureTask的内部状态

- NEW:表示是个新的任务或者还没被执行完的任务。这是初始状态。
- COMPLETING:任务已经执行完成或者执行任务的时候发生异常，但是任务执行结果或者异常原因还没有保存到outcome字段(outcome字段用来保存任务执行结果，如果发生异常，则用来保存异常原因)的时候，状态会从NEW变更到COMPLETING。但是这个状态会时间会比较短，属于中间状态。
- NORMAL:任务已经执行完成并且任务执行结果已经保存到outcome字段，状态会从COMPLETING转换到NORMAL。这是一个最终态。
- EXCEPTIONAL:任务执行发生异常并且异常原因已经保存到outcome字段中后，状态会从COMPLETING转换到EXCEPTIONAL。这是一个最终态。
- CANCELLED:任务还没开始执行或者已经开始执行但是还没有执行完成的时候，用户调用了cancel(false)方法取消任务且不中断任务执行线程，这个时候状态会从NEW转化为CANCELLED状态。这是一个最终态。
- INTERRUPTING: 任务还没开始执行或者已经执行但是还没有执行完成的时候，用户调用了cancel(true)方法取消任务并且要中断任务执行线程但是还没有中断任务执行线程之前，状态会从NEW转化为INTERRUPTING。这是一个中间状态。
- INTERRUPTED:调用interrupt()中断任务执行线程之后状态会从INTERRUPTING转换到INTERRUPTED。这是一个最终态。
有一点需要注意的是，所有值大于COMPLETING的状态都表示任务已经执行完成(任务正常执行完成，任务执行异常或者任务被取消)。
















