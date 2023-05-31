## 1. Executor框架模型
<img src="D:\Project\IT-notes\Java\并发\img\Executor框架模型.png" style="width:700px;height:600px;" />

## 2. Executor框架结构
Executor框架由三大部分组成：
1. 任务：实现了`Runnable`或`Callable`接口
2. 任务的执行：核心接口`Executor`、继承自`Executor`的`ExecutorService`接口
3. 异步计算：接口`Future`与`FuturnTask`类

<img src="D:\Project\IT-notes\Java\并发\img\Executor类与接口的关系谱.png" style="width:700px;height:600px;" />

<img src="D:\Project\IT-notes\Java\并发\img\Executor使用示意.png" style="width:700px;height:500px;" />

1. 主线程首先要创建实现`Runnable`或者`Callable`接口的任务对象
2. 然后可以把`Runnable`对象直接交给`ExecutorService`执行
3. 如果执行`ExecutorService.submit（…）`，`ExecutorService`将返回一个实现`Future`接口的对象
4. 最后，主线程可以执行`FutureTask.get()`方法来等待任务执行完成

## 3. Executor框架成员
通常使用工厂类`Executors`来创建。`Executors`可以创建3种类型的`ThreadPoolExecutor`：`SingleThreadExecutor`、`FixedThreadPool`和`CachedThreadPool`

### 1. FixedThreadPool
```java
//创建使用固定线程数的FixedThreadPool
public static ExecutorService newFixedThreadPool(int nThreads)
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactorythreadFactory)
```

### 2. SingleThreadExecutor
```java
//创建使用单个线程的SingleThreadExecutor
public static ExecutorService newSingleThreadExecutor()
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory)
```

### 3. CachedThreadPool
```java
//创建一个会根据需要创建新线程的 CachedThreadPool
public static ExecutorService newCachedThreadPool()
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory)
```

### 4. ScheduledThreadPoolExecutor
```java
//包含若干个线程的ScheduledThreadPoolExecutor
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) 
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize,ThreadFactory)

//创建单个线程 的SingleThreadScheduledExecutor
public static ScheduledExecutorService newSingleThreadScheduledExecutor()
public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory)
```

### 5. Future接口
```java
Future submit(Callable task)
Future submit(Runnable task, T result)
Future<> submit(Runnable task)
```

### 6. Runnable与Callable
```java
public static Callable callable(Runnable task) // 假设返回对象Callable
public static Callable callable(Runnable task, T result) // 假设返回对象Callable
```

## 4. 相关ThreadPoolExecutor详解
### 1. ThreadPoolExecutor详解
#### 1. FixedThreadPool
```java
public static ExecutorService newFixedThreadPool(int nThreads)
{
    return new ThreadPoolExecutor(nThreads, nThreads, 0 L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue < Runnable > ());
}
```

`FixedThreadPool`的`corePoolSize`和`maximumPoolSize`都被设置为创建`FixedThreadPool`时指定的参数`nThreads`

当线程池中的线程数大于`corePoolSize`时，`keepAliveTime`为多余的空闲线程等待新任务的最长时间，超过这个时间后多余的线程将被终止。这里把`keepAliveTime`设置为0L，意味着多余的空闲线程会被立即终止

**执行大致过程原理**：
1. 如果当前运行的线程数少于`corePoolSize`，则创建新线程来执行任务
2. 在线程池完成预热之后（当前运行的线程数等于`corePoolSize`），将任务加入`LinkedBlockingQueue`
3. 线程执行完任务后，会在循环中反复从`LinkedBlockingQueue`获取任务来执行

`FixedThreadPool`使用无界队列`LinkedBlockingQueue`作为线程池的工作队列的影响：
1. 当线程池中的线程数达到`corePoolSize`后，新任务将在无界队列中等待，因此线程池中的线程数不会超过`corePoolSize`
2. 由于使用无界队列，运行中的`FixedThreadPool`（未执行方法`shutdown()`或` shutdownNow()`）不会拒绝任务（不会调用`RejectedExecutionHandler.rejectedExecution`方法）

#### 2. SingleThreadExecutor
`SingleThreadExecutor`是使用单个`worker`线程的`Executor`
```java
public static ExecutorService newSingleThreadExecutor()
{
    return new FinalizableDelegatedExecutorService(new ThreadPoolExecutor(1, 1, 0 L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue < Runnable > ()));
}
```

**执行大致过程原理**：
1. 如果当前运行的线程数少于`corePoolSize`（即线程池中无运行的线程），则创建一个新线程来执行任务
2. 在线程池完成预热之后（当前线程池中有一个运行的线程），将任务加入`LinkedBlockingQueue`
3. 线程执行完任务后，会在循环中反复从`LinkedBlockingQueue`获取任务来执行

`SingleThreadExecutor`的`corePoolSize`和`maximumPoolSize`被设置为1，其他参数与`FixedThreadPool`相同
`SingleThreadExecutor`使用无界队列`LinkedBlockingQueue`作为线程池的工作队列（队列的容量为`Integer.MAX_VALUE`）
`SingleThreadExecutor`使用无界队列作为工作队列对线程池带来的影响与`FixedThreadPool`相同

#### 3. CachedThreadPool
`CachedThreadPool`是一个会根据需要创建新线程的线程池
```java
public static ExecutorService newCachedThreadPool()
{
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60 L, TimeUnit.SECONDS, new SynchronousQueue < Runnable > ());
}
```

`CachedThreadPool`的`corePoolSize`被设置为0，即`corePool`为空
`maximumPoolSize`被设置为`Integer.MAX_VALUE`，即`maximumPool`是无界的
这里把`keepAliveTime`设置为60L，意味着`CachedThreadPool`中的空闲线程等待新任务的最长时间为60秒，空闲线程超过60秒后将会被终止

`CachedThreadPool`使用没有容量的`SynchronousQueue`作为线程池的工作队列，但`CachedThreadPool`的`maximumPool`是无界的。这意味着，如果主线程提交任务的速度高于`maximumPool`中线程处理任务的速度时，`CachedThreadPool`会不断创建新线程

**执行大致过程原理**：
1. 首先执行`SynchronousQueue.offer(Runnable task)`。如果当前`maximumPool`中有空闲线程正在执行`SynchronousQueue.poll(keepAliveTime，TimeUnit.NANOSECONDS)`，那么主线程执行`offer`操作与空闲线程执行的`poll`操作配对成功，主线程把任务交给空闲线程执行，`execute()`方法执行完成；否则执行下面的步骤2
2. 当初始`maximumPool`为空，或者`maximumPool`中当前没有空闲线程时，将没有线程执行`SynchronousQueue.poll(keepAliveTime，TimeUnit.NANOSECONDS)`。这种情况下，步骤1将失 败。此时`CachedThreadPool`会创建一个新线程执行任务，`execute()`方法执行完成
3. 在步骤2中新创建的线程将任务执行完后，会执行`SynchronousQueue.poll(keepAliveTime，TimeUnit.NANOSECONDS)`。这个`poll`操作会让空闲线程最多在`SynchronousQueue`中等待60秒钟。如果60秒钟内主线程提交了一个新任务，那么这个空闲线程将执行主线程提交的新任务；否则，这个空闲线程将终止。由于空闲60秒的空闲线程会被终止，因此长时间保持空闲的`CachedThreadPool`不会使用任何资源

### 2. ScheduledThreadPoolExecutor详解


### 3. FutureTask详解
`FutureTask`除了实现`Future`接口外，还实现了`Runnable`接口。因此，`FutureTask`可以交给`Executor`执行，也可以由调用线程直接执行（`FutureTask.run()`）

<img src="D:\Project\IT-notes\Java\并发\img\FutureTask状态迁移.png" style="width:700px;height:400px;" />

<img src="D:\Project\IT-notes\Java\并发\img\FutureTask方法调用.png" style="width:700px;height:500px;" />

- `Callable`：`Callable`是一个接口，一个函数式接口，也是个泛型接口。`call()`有返回值，且返回值类型与泛型参数类型相同，且可以抛出异常。`Callable`可以看作是`Runnable`接口的补充
- `Future`：`Future`是为了配合`Callable/Runnable`而产生的，既然有返回值，那么返回什么？什么时候返回？这些问题其实都可以算在`Future`机制里
- `FutureTask`：`Future`机制就是为了解决多线程返回值的问题。但是`Callable`、`Future`、`RunnableFuture`都是接口；`FutureTask`实现了`RunnableFuture`接口，同时具有`Runnable`、`Future`的能力，即既可以作为`Future`得到`Callable`的返回值，又可以作为一个`Runnable`

```java
private static ExecutorService pool = Executors.newFixedThreadPool(nThreads,5);
public static void main(String]args){
	//方式一：
	MyTask myTask = new MyTask();
	FutureTask<Integer> futureTask = new FutureTask<>(myTask);
	pool.submit(futureTask);
	//方式二
	//new Thread(futureTask).start();
	try {
		//拿结果方式一
		Integer result = futureTask.get();
		//拿结果方式二，设置超时时间，最多等5秒
		Integer result1 = futureTask.get(timeout:5,TimeUnit.SECONDS);
		System.out.println(result);
		System.out.println(result1);
	} catch (Exception e)
		e.printStackTrace();
	}
}
static class MyTask implements Callable<Integer> {
	@0verride
	public Integer call() throws Exception {
		int sum =0;
		for (int i=0;i<1000;i++)
			sum +=i;
		TimeUnit.SECONDS.sleep(timeout:5);
		return sum;
	}
}
```

