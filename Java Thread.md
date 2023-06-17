# 进程和线程

**什么是进程？什么是线程？**

进程是程序的一次执行过程，操作系统以进程为单位为执行的程序分配资源和进行调度，每个进程都有独立的内存空间和一套变量。

而线程，它是进程内部的一个执行流程，每个进程至少有一条线程，进程内的多个线程共享进程的资源。多个线程同时运行，就称为并发。

系统资源虽然是分配给进程，但真正使用的却是线程，CPU 资源甚至以线程为单位进行分配。设计线程，是为充分地使用多核的资源。

**为什么设计线程？**

每创建一个进程，都要分配大量资源，并建立众多的数据表进行维护，开销较大。线程虽然也有自己的空间，但它能使用进程的共享资源，相比于进程，线程要轻量的多。

进程相互隔离，彼此都无法访问对方的资源，交互较复杂。而线程可通过共享资源，很方便地进行通信，影响对方的执行过程。

若没有线程，以进程为单位执行程序，那么进程只有一条执行流程，所以只能使用单个内核。面对多任务，将不得不创建多个进程。相比于多线程，多进程占用更多资源，通信复杂，无法共享资源，最重要的是，不能充分使用系统的 CPU 资源。

**主线程**

运行 Java 程序的 `main()` 方法，就是启动一个 JVM 进程。而 `main()` 所在的线程，是这个进程的第一个线程，称为主线程，在 JVM 这条线程的名字就是 *main*。

# 线程 Thread

## Thread

`Thread` 类表示线程，它包含许多线程相关的方法。

> **java.lang.Thread**
>
> * `Thread([Runnable target])`：构造器。
>
> * `start()`：启动新线程并执行 `run()` 方法，每个线程只能调用一次该方法。
>
> * `boolean isAlive()`：线程是否活着，线程新建或终结时，返回 `false`。
>
> * `static Thread currentThread()`：返回当前线程的实例。

**线程启动**

`Thread` 实现 `Runnable` 接口，所以它有 `run()` 方法。调用 `Thread::start()`，JVM 将为这个 `Thread` 对象创建一个新线程，并执行它的 `run()` 方法。

直接调用 `run()`，只是一次普通的方法调用，与线程无关。

线程只能调用一次 `start()`，多次调用抛出 IllegalThreadStateException 异常。

**线程名字**

在 JVM，线程有自己的名字，默认名字简单易记，比如 *main*、*Thread-1*。

> **java.lang.Thread**
>
> * `setName(String name)`：设置线程名。
> * `String getName()`：返回线程名。

## 线程状态

操作系统为线程定义 7 种状态，不过 Java 只定义 6 种。

### 新建 New

构造后尚未调用 `start()` 方法的线程。

### 可运行 Runnable

线程调用 `start()`，立即进入可运行状态。该状态的线程可能正在运行，也可能在等待 CPU 时间片。线程即使获得 CPU 执行时间，也不一定能直接执行到结束，执行中途可能需要暂停以让步给其它线程，这经常发生。

**Java 把 Ready 和 Running 两种系统的线程状态合并为 Runnable**

现代操作系统架构使用 "时间分片" 调度线程。每个时间片只能让线程使用 CPU 很短时间（Running），然后线程就被放到调度队列等待再次获得分片（Ready）。这里的时间量级很低，线程状态切换的太过频繁，以至于没有必要再区分 Ready 和 Running。

### 等待 Waiting

等待状态的线程不会被分配 CPU 时间片，它需要被其它线程显式唤醒，否则将一直处于该状态。

`Thread::wait()`、`Thread::join()`、`LockSuppor::lock()` 可以使线程进入等待状态。

### 限时等待 Timed Waiting

与等待状态类似，但限时等待的线程在经过指定时间后，会被自动唤醒。

`Thread::wait(long)`、`Thread::sleep(long)`、`Thread::join(long)`  可以使线程进入限时等待状态。

### 阻塞 Blocked

如果线程试图获得某个 `synchronized` 锁，而这把锁正被其它线程占有，它将进入阻塞状态。当锁被其它线程释放，并且线程调度器允许，它将占有这把锁并进入可运行状态。

### 终止 Terminated

运行结束的线程处于这种状态，线程会因为以下原因而终止：

* `run()` 执行结束，正常退出。
* `run()` 抛出一个未被捕获的异常。

不论线程如何终止，它都不会影响其它线程的正常运行。

## 创建线程

### 实现 Runnable

`Runnable` 接口表示线程的任务，它有一个 `run()` 方法，线程应该执行的逻辑就放在这里。

使用 `Runnable` 实例构造 `Thread` 对象，以此为线程设置任务。

```
Runnable r = () => {...};
Thread t = new Thread(r);
t.start();
```

### 实现 Callable

`Callable ` 接口表示可获得返回结果线程任务，它需要被 `FutrueTask` 封装才能使用。

`FutrueTask` 实现 `Runnable` 接口，所以可用它构造线程，以执行 `Callable ` 任务。

```
Callable<Integer> task = ...;
FutureTask<Integer> futureTask = new FutureTask<>(task);
new Thread(futureTask).start();
Integer result = futureTask.get(); // get the callable compute result
```

### 继承 Thread

其实 `Thread` 本身就是 `Runnable` 的实现，所以可通过继承和重写为线程设置任务。

```
public class A extends Thread {
    public void run() {
        ...
    }
}
```

### 比较与选择

根据需求，如果需要获得线程的执行结果，就用 `Callable` 创建线程。否则，推荐使用 `Runnable`，它把线程和任务解耦。

## 线程机制

### 线程控制

> **java.lang.Thread**
>
> * `static sleep(long millis)`：使当前线程进入限时等待，单位毫秒，不释放锁。
> * `static yield()`：当前线程请求放弃获得的 CPU 时间片，不一定生效，调度器可以忽略这个请求，该方法不阻塞线程，也不释放锁。
> * `join([long millis])`：使线程进入等待，直到当前线程终止，或等待超时，单位毫秒。

### 线程中断

若 `run()` 正常返回，或抛出未被捕获的异常，线程终止。以前可以调用 `Thread::stop()` 终止线程，但此方法已被废弃，当前没有任何方法可以强制终止线程。不过，可以使用线程中断建议线程终止。

`Thread::interrupt()` 设置中断状态，所谓中断，是线程的一个标记，所有线程在运行时都应该时不时地检查这个标记，以进行某些处理。

```
while (!Thread.currentThread().isInterrupted() && more work to do)
{
    do more work
}
```

如果线程调用 `sleep()`、`join()`、`wait()` 等方法进入等待，此时调用 `Thread::interrupt()` 设置中断，使线程进入等待的调用会立即抛出 `InterruptedException` 异常，线程转为可运行状态。但是，线程中断无法影响由于尝试获取 `synchronized` 锁而阻塞的线程。

如果处于中断状态的线程调用进入等待的方法，比如 `sleep()`、`join()`、`wait()`，这次调用不会生效，方法抛出异常，同时清除中断状态。

其实，中断状态只是一个线程的标记，如何响应，完全由程序的设计者而定。常见的 `sleep()`、`join()`，以及工具类 `LockSupport` 挂起线程的方法，都响应中断。

> **java.lang.Thread**
>
> * `void interrupt()`：设置中断，如果线程处于等待状态，则抛出 `InterruptedException` 异常。
> * `boolean isInterrupted()`：线程是否处于中断状态。
> * `static boolean interrupted()`：线程是否处于中断状态，同时清除中断状态。

### 守护线程

守护线程是在后台提供服务的线程，它与普通线程没什么差别。所有普通线程结束后，程序就要终止，它同时会终止所有的守护进程。

可通过 `Thread::setDaemon(boolean)` 设置守护线程，该方法需在 `start()` 前调用。

> **java.lang.Thread**
>
> * `setDaemon(boolean on)`：设置守护线程。
>
> * `boolean isDaemon()`：是否为守护线程。

### 线程优先级

`Thread` 有一个属性 `priority`，表示线程被调度器分配 CPU 时间片的优先程度。线程默认使用构造它的线程的优先级。

Java 线程的优先级范围是 `[1,10]`，JVM 把 Java 线程映射到内核线程。不同操作系统的优先级范围也不同，对于范围较小的系统，可能发生不同的 Java 线程优先级映射到相同的内核线程优先级。

推荐使用 `Thread` 提供的 3 个静态常量设置线程优先级，避免优先级映射重叠：

* `static int MIN_PRIORITY = 1`：最低优先级。

* `static int NORM_PRIORITY = 5`：默认优先级。

* `static int MAX_PRIORITY = 10`：最高优先级。

> **java.lang.Thread**
>
> * `setPriority(int newPriority)`：设置线程优先级。
> * `int getPriority()`：返回线程优先级。

### 未捕获异常的处理

并非所有的异常都能被捕获，对于 `run()`，当它抛出一个无法捕获的非检查型异常，线程会在终止前会把这个异常交给一个处理器。

`Thread` 有一个内部接口 `UncaughtExceptionHandler`，表示未捕获异常的处理器。

```
public interface UncaughtExceptionHandler 
{
	void uncaughtException(Thread t, Throwable e);
}
```

`ThreadGroup` 是线程分组，它实现 `UncaughtExceptionHandler`，每个线程都有自己的分组。它是线程对未捕获异常的默认处理器，以下是线程分组处理异常的逻辑：

* 如果有父线程分组，则把异常传给它。
* 如果 `Thread` 设有默认处理器，则把异常传给它。
* 如果异常是 `ThreadDeath` 类型，则什么也不做。
* 把 `Throwable` 的栈轨迹通过 `System.err` 输出。

> **java.lang.Thread**
>
> * `setUncaughtExceptionHandler(UncaughtExceptionHandler eh)`：设置处理器。
>
> * `static setDefaultUncaughtExceptionHandler(UncaughtExceptionHandler eh)`
>
>   设置全局默认处理器。
>

### 线程分组

`ThreadGroup` 是线程分组，以前用于管理一组线程，现在更倾向使用线程池管理多个线程。每个线程都有自己的分组，默认与创建它的线程同组，可在构造时指定分组。线程一旦被分组，就不能改变，直至终结。

第一个线程 `main` 属于名为 `main` 的分组，这个分组由 JVM 创建。对于用户自己创建的线程分组，它都有一个父线程分组，默认是创建它的线程的分组，可在构造时指定父分组。

# 线程池 Executor

虽然线程比进程轻量很多，但频繁地创建、销毁线程依然会带来很大的开销。原因是，大多 JVM 实现把线程映射到内核线程，那就需要进行系统调用，这会占用大量的 CPU 执行时间。

线程池维护多个可复用的线程，我们只需提交任务，线程池就会调用空闲线程或创建新线程来执行任务。任务结束后，线程不会终止，而是等待被分配下一个任务。

所以，线程池能带来以下好处：

* 减少开销，不用反复创建、销毁线程。
* 加快响应，复用已有线程，不用等待新线程的创建。
* 可管理性，线程池能对多个线程进行统一的分配、调优和监控。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/java-8fce42a6.png)

## 线程池接口

### Executor

`Executor` 接口表示线程池，只有一个提交任务的方法。

> **java.util.concurrent.Executor**
>
> * `execute(Runnable command)`：根据实现，复用线程，或创建线程，在未来执行提交的任务。
>

### ExecutorService

`ExecutorService` 是 `Executor` 的子接口，增加线程池的使用方法。

> **java.util.concurrent.ExecutorService**
>
> * `Future<T> submit(Callable<T> task)`
>
>   `Future<T> submit(Runnable task[, T result])`
>
>   提交任务，返回 `Future` 对象用于获取执行结果，第二个方法可选指定结果，不指定则为 `null`。
>
> * `void shutdown()`：关闭线程池，完成已提交的任务，拒绝新任务。
>
> * `List<Runnable> shutdownNow()`：关闭线程池，停止正在执行的任务，返回还没执行的任务。
>

`ExecutorService` 还定义有一些特殊方法，用于执行任务集合。

> **java.util.concurrent.ExecutorService**
>
> * `T invokeAny(Collection<Callable> tasks[, long timeout, TimeUnit unit])`
>
>   提交多个任务，其中任意一个结束，无论正常退出、取消或异常，都返回，可选限时完成。
>
> * `List<Future> invokeAll(Collection<Callable> tasks[, long timeout, TimeUnit unit])`
>
>   提交多个任务，返回对应的 `Future` 集合，可选限时完成。

### ScheduledExecutorService

`ScheduledExecutorService` 是 `ExecutorService` 的子接口，增加定时任务的方法。

> **java.util.concurrent.ScheduledExecutorService**
>
> * `ScheduledFuture schedule(Runnable command, long delay, TimeUnit unit)`
>
> * `ScheduledFuture schedule(Callable callable, long delay, TimeUnit unit)`
>
>   延迟指定时间，然后执行一次任务。
>
> * `ScheduledFuture scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)`
>
>   延迟 *initialDelay* 时间后周期执行任务，每次执行的开始间隔 *period* 时间。如果发生异常，禁止后续的执行。如果超过周期，延迟后续的执行。
>
> * `ScheduledFuture scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)`
>
>   延迟 *initialDelay* 时间后周期执行任务，每次执行结束后 *delay* 时间才进行下次执行。如果发生异常，禁止后续的执行。

## ThreadPoolExecutor

`ThreadPoolExecutor` 实现 `ExecutorService` 接口，是最核心的线程池。它的工作原理较简单，包括一个线程集合 workers 和一个阻塞队列 workQueue。用户向线程池提交的任务，先被插入阻塞队列，workers 的线程不断从 workQueue 获取任务然后执行，如果阻塞队列为空，workers 的线程会被阻塞。

### 线程池的状态

**线程池状态**

线程池有 5 种状态。

```
private static final int RUNNING    = -1 << COUNT_BITS;

private static final int SHUTDOWN   =  0 << COUNT_BITS;

private static final int STOP       =  1 << COUNT_BITS;

private static final int TIDYING    =  2 << COUNT_BITS;

private static final int TERMINATED =  3 << COUNT_BITS;
```

| 状态       | 数值 | 说明                                                    |
| ---------- | ---- | ------------------------------------------------------- |
| RUNNING    | -1   | 接收新任务，完成已提交的任务。                          |
| SHUTDOWN   | 0    | 拒绝新任务，完成已提交的任务。                          |
| STOP       | 1    | 拒绝新任务，抛弃阻塞队列的任务，中断正在执行的任务。    |
| TIDYING    | 2    | 所有任务都被完成，线程池为空，将要调用 `terminated()`。 |
| TERMINATED | 3    | 终止，`terminated()` 执行后的状态。                     |

线程池状态转换的时机：

* RUNNING -> SHUTDOWN：显式调用 `shutdown()`，或隐式调用 `finalize()` 内的 `shutdown()`。
* RUNNING 或 SHUTDOWN -> STOP：显式调用 `shutdownNow()`。
* SHUTDOWN -> TIDYING：线程池和阻塞队列都为空。
* STOP -> TIDYING：线程池为空。
* TIDYING -> TERMINATED：`terminated()` 执行完后。

**状态变量 ctl**

属性 ctl 记录线程池的信息，其中，最高 3 位表示线程池状态，剩余 29 位表示工作线程数。

```
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

// 根据线程池状态和工作线程数计算ctl
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

静态属性 CAPACITY 表示线程池的线程数的最大限制。

```
// 00011111111111111111111111111111
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
```

### 7 个核心参数

线程池的核心参数就是它参数最多的构造器的 7 个参数。

```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

| 参数                                | 说明                                                         |
| :---------------------------------- | :----------------------------------------------------------- |
| `int corePoolSize`                  | 核心线程数，线程池最少有这么多个线程。                       |
| `int maximumPoolSize`               | 最大线程数，线程池最多有这么多个线程。                       |
| `long keepAliveTime`                | 存活时间，非核心线程的最长闲置时间。                         |
| `TimeUnit unit`                     | 存活时间的单位。                                             |
| `BlockingQueue<Runnable> workQueue` | 阻塞队列，任务被线程处理前的存放地点。                       |
| `ThreadFactory threadFactory`       | 线程工厂，负责创建 `Thread` 对象。                           |
| `RejectedExecutionHandler handler`  | 饱和策略，阻塞队列满又没有空闲线程时，对新提交任务的处理方式。 |

饱和策略有 4 个选项，用枚举变量表示：

* `ThreadPoolExecutor.AbortPolicy`：默认，拒绝新任务，提交抛出异常。
* `ThreadPoolExecutor.CallerRunsPolicy`：用提交者的线程执行任务，若这个线程已关闭，则丢弃任务。
* `ThreadPoolExecutor.DiscardPolicy`：丢弃新任务，不做处理。
* `ThreadPoolExecutor.DiscardOldestPolicy`：丢弃阻塞队列最早的任务，接收新任务。

推荐使用构造器而非工具类 `Executors` 创建线程池，以定义更合适更安全的线程池运行方式。

### 部分源码解析

线程池初始为空，后面处理任务时才创建线程，调用 `prestartAllCoreThreads()` 提前实例化所有核心线程。

**提交任务 `execute()`**

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/java-5feba488.png)

```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get(); // 根据ctl计算线程数
    if (workerCountOf(c) < corePoolSize) { // 线程数<核心线程数，创建新线程执行任务
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 线程数>=核心线程数，如果线程池处于RUNNING，则把任务插入阻塞队列
    if (isRunning(c) && workQueue.offer(command)) {
    	// 二次检查，如果线程池处于非RUNNING，则把任务从阻塞队列剔除，并执行拒绝策略
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0) // 如果线程池为空，增加一个非核心线程
            addWorker(null, false);
    }
    // 任务插入队列失败，可能由于队满，创建非核心线程执行任务
    else if (!addWorker(command, false))
        reject(command); // 失败，说明线程池非RUNNING，或达到最大线程数，执行拒绝策略
}
```

**添加线程 `addWorker()`**

```
// core设置核心线程，firstTask是初始任务
private boolean addWorker(Runnable firstTask, boolean core) {
    // 更新线程数
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c); // 计算线程池状态

        // 线程池状态是否允许添加线程
        if (rs >= SHUTDOWN &&
                ! (rs == SHUTDOWN && firstTask == null && !workQueue.isEmpty()))
            return false;

		// 使用CAS更新线程数
        for (;;) {
            int wc = workerCountOf(c); // 计算线程数
            // 是否达到线程数限制，超过则返回失败
            if (wc >= CAPACITY || wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs) // 如果线程池状态发生变化，回到标记点重新计算
                continue retry;
        }
    }

	// 创建添加线程
    boolean workerStarted = false; // 线程启动标记
    boolean workerAdded = false; // 线程添加标记
    ThreadPoolExecutor.Worker w = null;
    try {
        w = new ThreadPoolExecutor.Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock; // mainLock是全局独占锁
            mainLock.lock();
            try {
            	// 再次检查线程池状态
                int rs = runStateOf(ctl.get());
                if (rs < SHUTDOWN || (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // 如果线程已启动，抛出异常
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true; 
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) { // 启动线程
                t.start();
                workerStarted = true;
            }
        }
    } finally { // 添加失败，进行相关处理，比如恢复线程数
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

**工作线程 `Worker`**

线程池不是直接使用 `Thread` 对象，而是把它封装成 `ThreadPoolExecutor.Worker` 类型，保存到 `HashSet` 集合，然后复用。对线程池来说，"线程" 既是指 `Thread`，也指 `Worker`。

```
private final HashSet<Worker> workers = new HashSet<Worker>();
```

`Worker` 是线程池的内部类，实现 `Runnable` 接口，并继承 AQS 实现为不可重入的独占锁，还封装 `Thread` 对象用于执行任务。

```
private final class Worker extends AbstractQueuedSynchronizer implements Runnable {
		
	final Thread thread;
    
	Runnable firstTask; // 初始任务
    
	volatile long completedTasks; // 完成的任务数量
```

`Worker` 调用 `run()` 进行启动，这个方法只是简单地调用线程池的 `runWorker()`，它不断地从队列获取任务并执行，使用独占锁保证线程安全。

```
final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // 释放锁
        boolean completedAbruptly = true;
        try {
            while (task != null || (task = getTask()) != null) { // 获取任务
                w.lock();
                ...
                } finally {
                    task = null;
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }
}
```

## ScheduledThreadPoolExecutor

`ScheduledThreadPoolExecutor` 实现 `ScheduledExecutorService` 接口。

## Callable 和 Future

**Callable**

`Callable ` 接口表示可获得返回结果线程任务，它的 `run()` 有返回值，还能抛出异常。

```
public interface Callable<V> 
{
    V call() throws Exception;
}
```

`Thread` 没有接收 `Callable` 任务的方法，可通过 `FutrueTask` 对象间接向 `Thread` 设置 `Callable` 任务。

**Future**

每个 `Callable ` 任务都有对应的 `Future` 对象，用于获取任务结果，或调控任务的执行。

> **java.util.concurrent.Future**
>
> * `V get([long timeout, TimeUnit unit])`：获取结果，如果还没完成，阻塞或限时阻塞。
> * `boolean cancel(boolean mayInterruptIfRunning)`：尝试取消任务，如果任务还未开始，则直接取消，如果任务正在执行，且 *mayInterruptIfRunning* 为 `true`，则中断线程。
> * `boolean isCancelled()`：如果任务在完成前被取消，返回 `true`。
> * `boolean isDone()`：如果任务结束，包括正常完成、发生异常、中途取消，返回 `true`。

## ExecutorCompletionService

对于批量执行 `Callable ` 任务，`ExecutorService.invokeAll()` 返回所有任务的 `Future` 列表，遍历这个列表获取结果，中间可能由于某个任务未完成而长时间阻塞。

可用 `ExecutorCompletionService` 批量执行 `Callable ` 任务，它并不实现线程池，而是在底层封装一个线程池实例，并提供非阻塞地获取结果的功能。原理很简单，它维护一个 `Future` 阻塞队列，不断的轮询每个任务是否完成，如果是则把该对象插入阻塞队列。

> **java.util.concurrent.ExecutorCompletionService**
>
> * `ExecutorCompletionService(Executor executor)`：构造器，*executor* 是线程池实例。
>
> * `Future submit(Callable task)`
>
>   `Future submit(Runnable task, V result)`
>
>   提交任务。
>
> * `Future take()`：获取一个已完成任务的结果，没有则阻塞。
>
> * `Future poll([long timeout, TimeUnit unit])`
>
>   获取一个已完成任务的结果，没有则返回 `null`，第二个方法将等待一段时间才认为没有结果。

## 工具类 Executors

`Executors` 提供许多线程池相关的方法，其中的线程池工厂方法，虽然很方便，但实践中常被禁止，原因如下：

* `FixedThreadPool()` 和 `SingleThreadExecutor()`，阻塞队列长度是 `Integer.MAX_VALUE`，可能堆积大量任务，导致 OOM。

* `CachedThreadPool()` 和 `ScheduledThreadPool()`，最大线程数是 `Integer.MAX_VALUE`，可能创建大量线程，导致 OOM。

**固定线程池**

```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

**缓存线程池**

```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

**单例线程池**

```
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

**调度线程池**

```
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
	return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

```
// 上面代码使用的构造器
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```

**单例调度线程池**

```
public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
    return new DelegatedScheduledExecutorService
        (new ScheduledThreadPoolExecutor(1));
}
```

**Runnable 转换 Callable**

```
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}
```

# 互斥同步 synchronized

同步指多个线程并发访问共享数据时，保证每个时刻至多有一条线程操作数据。互斥，也就是独占锁、悲观锁，这是实现同步的一种方式。

## 基本内容

**使用方式**

关键字 `synchronized` 是 Java 提供的互斥同步手段，它可以修饰实例方法、静态方法，以及块。

调用 `synchronized` 实例方法，需要获得对象的锁。调用 `synchronized` 静态方法，需要获得类的锁。

```
public [static] synchronized void method() {
    method body
}
```

修饰块时，需指定锁所属的对象，线程要进入这个块，必须先获得这个对象的锁。

```
synchronized (this) {
	method body
}
```

**线程释放锁的时机**

* 执行完所有代码，正常退出块或方法。
* 发生异常，导致线程终止。
* 调用 `wait()`，线程释放锁并进入等待状态。

**synchronized 的局限**

* 中断状态无法影响正在尝试获取锁而被阻塞的线程。
* 线程得到锁之前，会一直阻塞，不能设置超时时间。
* 每个对象锁只有一个条件。

**选择 synchronized 或 ReentrantLock？**

优先使用 `synchronized`，它的代码精简，且由 JVM 实现，出错的可能性更低，JDK 6 还对 `synchronized` 进行大量优化，相比于 `ReentrantLock` 已不存在性能上的差异。

仅当需要 `ReentrantLock` 的额外功能时，才选择它。

## 等待交互

线程进入同步块/方法后，可调用获得的锁所属对象的 `wait()` 方法，使当前线程释放锁并进入等待，以把锁资源让给其它线程。`wait()`、`notify()` 等方法的实现，依赖于 Monitor 对象。

> **java.lang.Object**
>
> * `wait([long timeout[, int nanos]])`
>
>   使当前线程进入等待并释放对象的锁，直到被其它线程唤醒，或等待超时，或线程中断，*timeout* 单位毫秒，*nanos* 单位纳秒，*timeout* 为 0 表示永久等待。
>
> * `notify()`：随机唤醒一个等待获取该对象的锁的线程。
>
> * `notifyAll()`：唤醒所有等待获取该对象的锁的线程。

## 实现原理

**对象锁 ObjectMonitor**

查看字节码（`javap`），发现同步块的内容被 monitorenter 和 monitorexit 两个指令包裹，从名字看，它们都与 monitor 相关。其实，Monitor 就是锁资源，由 C++ 的 `ObjectMonitor` 类实现，每个 Java 对象都有一个自己的监视器。指令 monitorenter 和 monitorexit 操作监视器，分别是获取锁和释放锁。

> `synchronized` 方法的字节码稍有不同，但都是通过 Monitor 对象实现。

因此，`synchronized` 是由 JVM 在字节码层面实现，从而保证线程安全。

**计数器 Monitor**

监视器 Monitor 有一个计数器，值为 0 时表示锁空闲，可以获取，线程通过 monitorenter 指令获取锁，监视器记录线程并把计数器+1。此后，线程可对这个锁重复执行 monitorenter 指令，每次都 +1 计数器。所以，对象锁是可重入锁。每次执行 monitorexit 指令，就 -1 计数器，直到计数器的值为 0，才认为线程释放锁，最后清除记录。

## 原子，可见，有序

关键字 `synchronized` 非常强大，它能同时解决并发的 3 个核心问题：原子性，可见性，有序性。

**原子性**

指令 monitorenter 和 monitorexit 会隐式调用 JMM 的 `lock` 和 `unlock` 操作，所以，任意时刻至多有一条线程进入同步块，自然可以保证原子性。

**可见性**

Happens-Before 规定："对一个变量执行 `unlock` 操作之前，必须先把这个变量同步回主内存"。

**有序性**

JMM 规定："每个变量，在同一时刻，只能被一条线程执行 `lock` 操作"，所以，要获取锁的线程只能串行地进入同步块。而在单个线程内，即使发生重排序，也没有线程安全问题，因为线程内表现为有序。

不同的是，关键字 `volatile` 是通过内存屏障的方式，禁止重排序，从而保证程序执行的有序性。

## JDK 6 的 synchronized 优化

大多 JVM 实现把 Java 线程映射为内核线程，所以 Java 线程的挂起、恢复等最终都要进行系统调用，而这就会在内核态与用户态之间来回切换，耗费大量的 CPU 执行时间。这两状态切换的开销主要来自响应中断、保护和恢复执行现场。

所以，`synchronized` 锁的持有，是一个重量级操作，因为线程遇到锁不空闲时会被挂起。为此，HotSpot 虚拟机实现在 JDK 6 为 `synchronized` 提供大量的优化，这些都是在 JVM 层面。

### 锁消除

JVM 在运行过程中，如果监测到某段同步块的代码不可能存在共享数据竞争，那么就忽略这个锁。这种优化是有必要的，尽管开发人员不会滥用 `synchronized`，但许多 JDK 类库都有使用对象锁，所以，锁消除仍能明显地提升程序的性能。

判断是否可能存在共享数据竞争的依据，来源于逃逸分析。比如，如果在一段代码中发现，其操作的堆内数据都不会逃逸出去被其它线程访问，那就直接把这些数据看作栈内数据，这自然就不可能有线程安全问题。

### 锁粗化

平时编码，同步块的范围应该尽可能小，因为需要同步的操作越少，其占用锁的时间也越少，发生线程竞争的概率也就越低。但是，如果有一连串操作，都分别去获取同一个锁，甚至在循环体内获取锁，那即便没有竞争，反复地加锁解锁也会耗费大量资源。

如果 JVM 发现这样的一串操作，就把它们的锁合并，同步的范围是整个序列的外部。这样，就只有一次加锁和解锁。下例，虽然 `append()` 是同步方法，但这一连串的调用，只会进行一次加锁和解锁。

```
StringBuffer sb = new StringBuffer();
sb.append(1).append(2).append(3).append(4).append(5);
```

### 自旋锁

因为 JVM 把 Java 线程映射到内核线程，那么由于 `synchronized` 而导致的阻塞和恢复都要进行系统调用，这会耗费大量的 CPU 执行时间。

**什么是自旋？**

锁被占用的时间往往很短，如果每次获取锁失败都挂起线程，为这点时间去系统调用很不值得。因为现在大多主机都是多核处理器，支持多线程并发执行。那么可以这样做，如果某个线程获取锁失败，不是立即挂起它，而是先让其执行一个忙循环，这个忙循环就是自旋，从而避免系统调用的开销。

自旋锁在 JDK 1.4.2 就已经引入，但默认关闭，JDK 6 改为默认开启。

可以使用 JVM 参数 `-XX:+/-UseSpinning` 开启或关闭自旋锁。

**自旋次数有限**

自旋虽然可以避免线程切换的开销，但它一直占用 CPU，如果锁很快就被释放，那自旋的效果就非常好。如果锁长时间没有释放，那自旋就会耗费大量的 CPU 执行时间，得不偿失。

所以，自旋的次数有上限，如果超过限定次数还未获得锁，那就挂起线程，自旋上限默认是 10 次。

可以使用 JVM 参数 `-XX:PreBlockSpin` 修改自旋上限。

**自适应自旋**

不同代码的执行速度千差万别，如果所有的锁的自旋上限都相同且固定，这当然不合理。JDK 6 对自旋锁增加自适应的功能，这表示锁的自旋上限会动态变化，根据锁之前的自旋时间和持有线程的状态调整。如果这个锁上次自旋成功，并且持有线程正在运行，JVM 就认为这次自旋也很有可能成功，所以提高它的自旋上限。反之，如果这个锁很少自旋成功，就降低它的自旋上限，甚至不允许它自旋。

随着程序运行时间的增加，以及性能监控信息的不断完善，JVM 对锁的状况预测就越来越准，自适应自旋的效果会越来越好。

### 轻量级锁

轻量级锁是 JDK 6 引入的新型锁机制，它的目的是 "在没有多线程竞争的前提，减少传统的重量级锁使用操作系统互斥量的性能消耗"。

**对象头信息**

要理解轻量级锁和偏向锁，需要知道 HotSpot 虚拟机对象的内存布局，HotSpot 把对象头分为两部分。

* 第一部分保存对象的运行数据，比如哈希码。这部分在 32 位和 64 位的虚拟机分别占用 32 和 64 比特，官方称它为 "Mark Word"。这部分是实现轻量锁和偏向锁的关键。
* 另一部分保存指向方法区类型数据的指针，如果是数组对象，对象头还要记录数组的长度。

对象头信息是与对象数据无关的额外存储成本，为提高 JVM 的空间使用效率，Mark Word 被设计成动态的数据结构，以便在小空间保存尽量多的信息。下表是 32 位的 HotSpot 虚拟机在不同状态的 Mark Word 结构。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/java-264ad76e.png)

**轻量级锁的加锁**

线程即将进入同步块时，如果锁还未被获取，即 Mark Word 的标志位是 "01"，JVM 会在当前线程的栈帧内创建一个名为 Lock Record 的空间，保存着锁所属对象的 Mark Word 拷贝。

此时，线程堆栈和对象头的状态如下所示：

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/java-a13c43c9.png)

然后，JVM 使用 CAS 操作，尝试把 Mark Word 更新为指向 Lock Record 的状态，如果成功，表示当前线程获取到这个对象锁，同时，标志位也更新为 "00"，对象处于轻量级锁定。

此时，线程堆栈和对象头的状态如下所示：

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/java-5f5d8b3b.png)

如果 CAS 更新失败，首先 JVM 检查 Mark Word 是否指向当前线程的 Lock Record，如果是，说明当前线程已经持有这个锁，直接进入同步块。否则，说明锁被其它线程获取，存在线程竞争，轻量级锁不再适用，膨胀为重量级锁，标志位更新为 "10"，Mark Word 转而存储指向重量级锁的指针，当前线程自旋或挂起。

**轻量级锁的解锁**

JVM 使用 CAS 操作释放轻量级锁，尝试用 Lock Record 的拷贝覆盖 Mark Word，如果成功，Mark Word 还原成未锁定的状态。如果失败，说明存在线程竞争，锁已经成为重量级锁，此时丢弃 Lock Record，并唤醒被这个锁阻塞的线程。

**轻量级锁的效果**

经验法则 "绝大多数的锁，在整个同步周期内都不存在线程竞争"，只有在这个前提，轻量级锁才能提升程序的运行效率。

如果，当前程序的锁普遍存在线程竞争，轻量级锁只会使程序更慢，因为它额外增加了许多 CAS 操作。

### 偏向锁

偏向锁是 JDK 6 引入的新型锁机制，它的目的是 "在没有多线程竞争的前提，消除同步原语，进一步提高程序的运行性能"。

对比来看，轻量级锁使用 CAS 操作消除同步使用的互斥量，而偏向锁甚至把 CAS 操作都省略，相当什么都不做。

可以使用 JVM 参数 `-XX：+/-UseBiased Locking` 开启或关闭偏向锁，默认开启。

**对象锁的升级**

对象锁从偏向锁开始，只能单向升级：偏向锁 -> 轻量级锁 -> 重量级锁，不允许回退。

**偏向锁的加锁**

如果偏向锁开启，Mark Word 初始的标志位为 "01"，偏向模式为 "1"，即"无锁可偏向"状态。

线程即将进入同步块时，如果锁可偏向，比较 Mark Word 保存的线程 ID 与当前线程，相等则进入同步块，否则使用 CAS 操作向 Mark Word 写入当前线程 ID，成功说明锁处于无锁可偏向，进入同步块，失败说明偏向锁已被其它线程获取，开始撤销偏向锁。

从这里看，偏向锁只有更新线程 ID 这一个 CAS 操作，效率更高。

**偏向锁的撤销**

偏向锁没有解锁的概念，因为它偏向于锁只被一个线程获取过，即使线程已经退出同步块，这个偏向锁也不该被其它线程获取，除非允许重偏向。

偏向锁的撤销需要在全局安全点，这个时刻没有正在执行的字节码，即所有线程都停止工作，要求这么苛刻，必然影响性能。

JVM 暂停所有线程后，再根据锁定状态，决定是否撤销偏向锁，撤销后 Mark Word 的标志位变为未锁定或轻量级锁定，后续的步骤就按照轻量级锁进行。

* 未锁定，持有偏向锁的线程要么终结，要么退出同步块。如果允许重偏向，使用 CAS 操作更新 Mark Word 的线程 ID，否则，把锁升级为轻量级锁，状态为"未锁定"。
* 锁定，持有偏向锁的线程正在执行同步块的代码。偏向锁升级为轻量级锁，状态为"已锁定"。

最后，恢复所有线程的运行。

偏向锁、轻量级锁、重量级锁的转化，以及 Mark Word 的变化如下所示：

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/java-5b817e00.png)

**偏向锁的效果**

同样，偏向锁只有在线程竞争较少时，才会明显地提升程序性能。如果程序普遍存在线程竞争，它会在轻量级锁之上进一步扩大性能开销。

**偏向锁与 HashCode**

查看 Mark Word，当处于可偏向状态，便不再记录 HashCode 哈希码。Java 对象如果计算过哈希码，就应该永远保持不变，否则很多依赖 HashCode 的 API 可能出错。方法 `Object.hashCode()` 是大多数对象 HashCode 字段的来源，它返回对象的一致性哈希，这个值的不可变性通过在 Mark Word 保存来实现。

所以，如果对象调用过 `hashCode()`，那么它的锁就永远无法进入偏向模式。如果锁正处于偏向，对象此时收到计算其哈希码的请求，偏向锁就升级为重量级锁，Mark Word 会保存 `Monitor` 对象的指针，而 `Monitor` 内有非加锁状态时 Mark Word 的拷贝（类似 Lock Record），自然也有 HashCode。

如果用户重写对象的 `hashCode()` 方法，Mark Word 不会保存这个方法的返回值，因为无法保证其一致性。

# 原子操作 CAS

## CAS 概念

**指令集的支持**

虽然锁可以解决线程安全问题，但它的开销较大。随着硬件指令集的发展，可以把某些操作在硬件层面转换为一个原子操作，从而保证线程安全，这种方式的效率远远高于锁机制。

**CAS 操作**

CAS（Compare-and-Swap）是基于硬件的非阻塞原子性更新操作，基本所有指令集都支持这个功能。

CAS 有 3 个参数：内存位置 V、预期的旧值 A、准备设置的新值 B，它的逻辑是：如果 V 的值为 A，就把 V 的值更新为 B，不论是否更新，都返回 V 的旧值。

**CAS 缺陷**

* ABA 问题

  CAS 获取到旧值后，另一个线程修改值，然后又把值恢复，期间 CAS 无法察觉这些修改，后续的步骤依然照常执行。对此，可以为变量的每次修改关联一个版本号，JDK 1.5 的 `AtomicStampedReference` 原子类，使用时间戳关联每次修改。

* 操作数量

  CAS 每次只能操作单个变量，对此，JDK 1.5 提供引用类型的原子类 `AtomicReference`，可以把多个变量放到一个对象，通过原子性的修改对象实现原子性地修改多个变量。

## Unsafe

`sun.misc.Unsafe` 类有一些 `native` 方法，它们与 CAS 操作相关，使用 JNI 方式访问本地的 C++ 库实现。因为这个类可以直接操作 JVM 内存，为了安全，Java 限制它只能被 `Bootstrap` 加载的类访问。如果确实需要，可以通过反射获取 `Unsafe` 实例。

> **sun.misc.Unsafe**
>
> * `native park(boolean isAbsolute, long time)`：
>
>   挂起当前线程。*time* 超时时间，如果 *isAbsolute* 是 `false`，那么 *time* 表示相对时间，单位纳秒，否则表示绝对时间，单位毫秒。*time* 为 0 表示永久阻塞。
>
>   `park()` 挂起的线程，可被 `unpark()` 唤醒，响应中断但不抛出异常，如果之前调用过 `unpark()` 或处于中断状态，`park()` 立即返回。
>
> * `native unpark(Object thread)`：恢复当前线程。

## 原子类

基于 `Unsafe`，JDK 提供许多原子操作的类。

* `AtomicInteger`：整型
* `AtomicLong`：长整型
* `AtomicBoolean`：布尔类型
* `AtomicReference`：引用类型
* `AtomicIntegerArray`：整型数组
* `AtomicLongArray`：长整型数组
* `AtomicReferenceArray`：引用类型数组
* `AtomicStampedReference`：引用类型，内部使用 `Pair` 存储元素及其版本号
* `AtomicMarkableReferce`：原子更新带有标记位的引用类型

# 抽象同步队列 AQS

## 工具类 LockSupport

`LockSupport` 提供许多线程控制相关的方法，包括挂起/唤醒线程。查看源码，它基本就是直接调用 `Unsafe` 的方法，也是因此，`LockSupport` 挂起的线程会响应线程中断。

> **java.util.concurrent.locks.LockSupport**
>
> * `static park()`：`Unsafe::park(false, 0L)`。
>
> * `static unpark(Thread thread)`：`Unsafe::unpark(thread)`。
>
> * `static parkNanos(long nanos)`：`Unsafe::park(false, nanos)`。
>
> * `static park(Object blocker)`：
>
>   `Unsafe::putObject(t, parkBlockerOffset, arg)` + `Unsafe::park(false, 0L)`。
>
>   绑定对象 *blocker* 用于传递信息，设置在线程对象属性。
>
> * `static parkNanos(Object blocker, long nanos)`：
>
>   `Unsafe::park(false, nanos)` + `Unsafe::putObject(t, parkBlockerOffset, arg)`。
>
> * `static parkUntil(Object blocker, long deadline)`：
>
>   `Unsafe::park(true, deadline)` + `Unsafe::putObject(t, parkBlockerOffset, arg)`。
>
> * `static Object getBlocker(Thread t)`：获取指定线程的 *blocker* 对象。

## AQS 同步器模板

### 基本概念

`AbstractQueuedSynchronizer` 抽象类是设计锁以及其它同步器的基类，它使用模板模式，把代码的运行流程完全固定，只有少数方法允许重写，以让用户自定义功能，这使同步器的实现简单且规范。![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/java-94264b32.png)

**AQS 队列**

AQS 维护一个虚拟的双端队列，虚拟的意思是没有具体的队列实例，只靠结点间相互关联组成队列，属性 head 和 tail 指向队头和队尾。

内部类 `Node` 表示队列结点，每个结点代表一个进入队列的线程，属性 thread 指向线程实例，prev 和 next 关联前、后结点，SHARED 和 EXCLUSIVE 标记当前线程由于获取共享\独占资源而进入阻塞队列，waitStatus 是线程的等待状态：CANCELLED（-1）线程被取消，SIGNAL（-1）线程等待唤醒，CONDITION（-2 ）线程在条件队列等待，PROPAGATE（-3）释放资源时需要通知其它线程。

**state**

属性 state 是 AQS 的核心，记录 AQS 当前的状态。其实，state 就是 AQS 所谓的资源，获取资源，就是给 state 设值，释放资源，则是还原 state 的值。在不同的实现，state 的意义不同。比如在 `ReentrantLock`，state 表示锁定状态和重入次数，而在 `CountDownlatch`，则表示计数器。

```
private volatile int state;
```

方法 `getState()`、`setState()`、`compareAndSetState()` 访问和修改 state 属性。

```
protected final int getState() {return state;}

protected final void setState(int newState) {state = newState;}

protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

**ConditionObject**

内部类 `ConditionObject` 能与同步器合作，实现条件等待功能，它的作用和 `wait()` 和 `signal()` 相同，每个 AQS 可以有多个条件对象。`ConditionObject` 在内部也维护一个双端队列，即等待队列，结点类型也是 `Node`，表示因调用这个条件的 `await()` 方法而进入等待状态的线程

> **java.util.concurrent.locks.Condition**
>
> * `await()`：使当前线程释放关联的锁，并进入等待状态。
> * `boolean await(long time, TimeUnit unit)`：超时等待。
> * `signal()`：把条件队列的队头从等待队列移动到 AQS 队列，并唤醒。
> * `signalAll()`：把条件队列的所有结点从等待队列移动到 AQS 队列，并唤醒。

### 模板方法

AQS 的思想是：如果资源空闲，就把当前请求资源的线程设为有效线程，并锁定资源。如果资源已被占用，就把请求资源的其它线程挂起并放到 AQS 队列。

与独占资源相关的可重写方法：

```
protected boolean tryAcquire(int arg) {
	throw new UnsupportedOperationException();
}
```

```
protected boolean tryRelease(int arg) {
	throw new UnsupportedOperationException();
}
```

与共享资源相关的可重写方法：

```
protected int tryAcquireShared(int arg) {
	throw new UnsupportedOperationException();
}
```

```
protected boolean tryReleaseShared(int arg) {
    throw new UnsupportedOperationException();
}
```

另外，还有 `isHeldExclusively()` 方法，判断资源是否是独占方式。

```
protected boolean isHeldExclusively() {
    throw new UnsupportedOperationException();
}
```

### 独占资源

独占方式获取的资源是与具体线程绑定，即如果某个线程获取到资源，AQS 就记录资源被这个线程占有。后续其它的线程再请求获取资源，就会被阻塞并加入 AQS 队列。

独享方式相关方法：`acquire()`、`acquireInterruptibly()`、`release()`

**`acquire()`**

首先调用 `tryAcquire()` 获取资源，成功则返回，否则把当前线程封装为 `EXCLUSIVE` 结点插入 AQS 队列，然后调用 `LockSupport::park()` 挂起线程。

```
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

**`acquireInterruptibly()`**

与 `acquire()` 类似，但会响应中断状态，获取资源的过程中，如果线程被中断，就抛出异常。

```
public final void acquireInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (!tryAcquire(arg))
        doAcquireInterruptibly(arg);
}
```

**`release()`**

首先调用 `tryRelease()` 释放资源，然后使用 `unparkSuccessor()` 恢复 AQS 队列的头结点，激活的线程再次获取资源，如果失败，还是会被插入 AQS 队列。

```
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

### 共享资源

共享方式获取的资源与具体线程不相关，某个线程请求获取资源，即使资源正被占用，但 state 仍满足需求，则成功获取资源。

共享方式相关方法：`acquireShared()`、`acquireSharedInterruptibly()`、`releaseShared()`

**`acquireShared()`**

```
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

**`acquireSharedInterruptibly()`**

```
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
```

**`releaseShared()`**

```
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

## ReentrantLock

### 使用方式

`ReentrantLock` 是基于 AQS 实现的可重入的独占锁，它的作用与 `synchronized` 相似，但实现在代码层，且功能更多，比如它可以有多个条件对象，还支持尝试性地获取锁。

使用 `ReentrantLock` 对代码块进行同步：

```
lock.lock(); // a ReentrantLock object
try {
    critical section
} finally {
    lock.unlock(); // make sure the lock is unlocked even if an exception is thrown
}
```

> **java.util.concurrent.locks.ReentrantLock**
>
> * `ReentrantLock([boolean fair])`
>
>   构造器，*fair* 开启公平锁，默认关闭，公平锁优先唤醒阻塞时间更长的线程，即使开启公平锁，也不能保证线程调度器一定公平，且公平锁严重降低并发性能。
>
> * `lock()`：获取锁，如果锁不空闲，阻塞当前线程。
>
>   `unlock()`：释放锁。
>
>   `boolean tryLock()`：尝试索取锁，成功返回 `true`，失败返回 `false`，线程不会阻塞。
>
>   `boolean tryLock(long timeout, TimeUnit unit)`：限时获取锁，超时返回 `false`。
>
> * `Condition newCondition()`：构造并返回一个条件对象。

### 源码解析

**内部封装 AQS 实现**

`ReentrantLock` 有 3 个内部类，其中 `Sync` 继承 AQS，`FairSync` 和 `NonfairSync` 继承 `Sync`，实现为公平锁和非公平锁。现在，AQS 的 state 表示是否锁定以及重入次数，初始为 0 表示锁空闲。

```
private final Sync sync;

abstract static class Sync extends AbstractQueuedSynchronizer { ... }

static final class FairSync extends Sync {...} // 公平实现

static final class NonfairSync extends Sync {...} // 非公平实现

public ReentrantLock() {sync = new NonfairSync();} // 默认使用非公平锁

public ReentrantLock(boolean fair) {
	sync = fair ? new FairSync() : new NonfairSync();
}
```

**获取非公平锁 `nonfairTryAcquire()`**

`Sync::lock()` 获取锁，非公平模式，如果锁空闲，直接 CAS 获取，否则就把线程挂起并插入 AQS 队列。所谓的非公平，表现在只要发现 state 为 0，就立即获取，而不管 AQS 队列是否有线程正在等待。

```
final void lock() {
    if (compareAndSetState(0, 1)) // 锁空闲，直接获取锁
        setExclusiveOwnerThread(Thread.currentThread()); // 设置锁被线程持有
    else
        acquire(1); // 获取锁，state+1
}

final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) { // 锁空闲，直接获取锁
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) { // 已经持有锁，只需更新state
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

**获取公平锁 `hasQueuedPredecessors()`**

公平锁的获取与非公平锁基本相同，只是增加 `hasQueuedPredecessors()`，这个方法是公平锁的核心，它检查当前 AQS 队列是否有前驱结点。

```
final void lock() {
    acquire(1); // 获取锁，state+1
}

protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
    	// 锁空闲，且AQS队列没有前驱结点，直接获取锁
        if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) { // 已经持有锁，只需更新state
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}

/*
  判断是否有其它线程正在AQS队列等待
  head==tail，说明AQS队列为空
  head!=tail && head.next==null，说明有线程正在插入AQS队列，只是head还未来得及更新后驱
  head!=tail && head.next.thread != Thread.cunentThread()，说明AQS队列还有其它线程正在等待
*/
public final boolean hasQueuedPredecessors() {
	Node h = head;
	Node t = tail; // Read fields in reverse initialization order
    Node s;
    return h != t && ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

**释放锁 `tryRelease()`**

```
public void unlock() {
    sync.release(1); // 释放锁，state-1
}

protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    // 如果锁不属于当前线程，抛出异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    // 当state减到0，释放锁
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

## CountDownLatch

### 使用方式

`CountDownLatch` 也在内部封装 AQS 实现，以此实现同步计数器。它的作用是使当前线程阻塞，直到并行的其它线程完成任务。`Object::join()` 也有相同的效果，但它只能等待一个任务，且不够灵活。

使用 `CountDownLatch` 使主线程等待两个子线程完成任务：

```
CountDownLatch count = new CountDownLatch(2); // 计数器初始为2
...
t1.start(); // 子线程完成任务后，调用countDown()把计数器-1
t2.start();
...
count.await(); // 挂起当前线程，直到计数器为0
```

> **java.util.concurrent.CountDownLatch**
>
> * `CountDownLatch(int count)`：构造器，*count* 是计数器的初始值。
>* `await[(long timeout, TimeUnit unit])`：使当前线程进入等待状态，直到计数器为 0，或线程中断，或等待超时。
> * `void countDown()`：计数器 -1，当计数器为 0，唤醒所有等待线程。
>* `long getCount()`：获取计数器的值。

### 源码解析

**内部封装 AQS 实现**

`CountDownLatch` 封装 AQS 的继承 `Sync`，这里 state 表示计数器的值。

```
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}

Sync(int count) {setState(count);}
        
protected final void setState(int newState) {state = newState;}
```

**使线程等待 `await()`**

使当前线程阻塞，响应中断，如果 state 为 0，直接返回。

```
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}

public final void acquireSharedInterruptibly(int arg) throws InterruptedException {
    if (Thread.interrupted()) throw new InterruptedException();
    if (tryAcquireShared(arg) < 0) doAcquireSharedInterruptibly(arg);
}

protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}
```

**更新计数器 `countDown()`**

如果 state 等于 0，直接返回，否则把计数器 -1，更新后如果 state 为 0，则把所有因调用 `await()` 而被挂起并放入 AQS 队列的线程唤醒。

```
public void countDown() {
    sync.releaseShared(1);
}

public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

protected boolean tryReleaseShared(int releases) {
    // Decrement count; signal when transition to zero
    for (;;) {
        int c = getState();
        if (c == 0)
            return false;
        int nextc = c-1;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

# 线程本地变量 ThreadLocal

## 基本内容

`ThreadLocal` 类似一个容器，为各个线程提供一个值的空间，它是泛型类，只允许存储引用类型。

主线程创建一个 `ThreadLocal` 实例，其它线程（包括主线程）访问这个变量，设置、获取的值相互隔离。

```
public static ThreadLocal<String> local = new ThreadLocal<>();

Thread t1 = new Thread(new Runnable() { public void run() { //use local }});
Thread t2 = new Thread(new Runnable() { public void run() { //use local }});

t1.start();
t2.start();
```

> **java.lang.ThreadLocal**
>
> * `ThreadLocal()`：构造器。
>
> * `T get()`：获取本地变量，如果还未设置，则用 `initialValue()` 先初始化。
>
> * `set(T value)`：设置本地变量。
>
> * `remove()`：删除本地变量。
>
> * `initialValue()`：返回本地变量的初始值，默认 `null`，可重写该方法。
>
> * `static ThreadLocal withInitial(Supplier supplier)`：工厂方法，它用提供者 Lambda 重写返回实例的 `initialValue()`，这可以省去声明类继承 `ThreadLocal` 的步骤。

## 源码解析

**基本实现原理**

`ThreadLocal` 并不保存各个线程的本地变量，`Thread` 有一个属性 threadLocals，是定制的 `HashMap` 类型，这里才是保存本地变量的地方，对应的 key 是 `ThreadLocal` 实例。threadLocals 默认 `null`，首次被访问时才实例化。

**设置本地变量 `set(T value)`**

```
public void set(T value) {
    Thread t = Thread.currentThread(); // 本地变量的key是对应的线程实例
    ThreadLocalMap map = getMap(t); // return t.threadLocals;
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value); // t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

**获取本地变量 `get`()**

```
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

**删除本地变量 `remove()`**

```
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```

## 内存泄漏

多个线程使用同一个 `ThreadLocal` 实例，且本地变量较大，如果这些线程一直不被销毁，将占用大量空间，造成内存泄漏。虽然 `ThreadLocal` 对 key 是弱引用，但用到 `ThreadLocal` 的线程不被销毁，对应的 key 就一直存在强引用，值自然就无法被回收。

可用 `ThreadLocal::remove()` 方法，手动删除本地变量。

# 定时任务 Timer

`Timer` 是定时器，执行定时任务，它在内部封装一个 `Thread` 实例，可执行多个定时任务。`TimerTask` 是定时任务，它实现 `Runnable` 接口，`run()` 就是任务内容。

```
Timer timer = new Timer(true);
TimerTask timerTask = new TimerTask() {
    @Override
    public void run() {
        System.out.print(Thread.currentThread().getName());
        System.out.println(Thread.currentThread().isDaemon());
    }
};
timer.schedule(timerTask, 100, 1000);
```

> **java.util.Timer**
>
> * `Timer([String name, ]boolean isDaemon)`
>
>   构造器，*isDaemon* 设置定时器线程为守护线程，*name* 是定时器线程的名字。
>
> * `schedule(TimerTask task, long delay)`：延迟 *delay* 毫秒后执行一次任务。
>
> * `schedule(TimerTask task, long delay, long period)`：延迟 *delay* 毫秒后，以 *period* 毫秒为周期，循环执行任务。
