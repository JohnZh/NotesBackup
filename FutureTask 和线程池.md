# Runable\Callable\Future

```java
public interface Runnable {
  public void run();
}

public interface Callable<V> {
  V call() throws Exception;
}

public interface Future<V> {
  boolean cancel(boolean mayInterruptIfRunning);
  boolean isCancelled();
  boolean isDone();
  V get() throws InterruptedException, ExecutionException;

  V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException;
}
```



# FutureTask

```java
Runnable, Future<V> (run(), boolean cancel(), isCanceled(), isDone(), V get())
  RunnableFuture<V> (run())
  	FutureTask<V>
```



# FutureTask 使用

```java
FutureTask futureTask = new FutureTask<>(new Callable<String>() {
  @Override
  public String call() throws Exception {
    System.out.println("MyCode.call");
    Thread.sleep(2000);
    return "Done";
  }
});
new Thread(futureTask).start();
System.out.println(futureTask.get());

// output:
// MyCode.call
// Done
```



# FutureTask 数据结构

定义了 volatile 内存可见成员变量

- task 的状态 state
- 等待线程的队列 waiters（队头指针）
- 当前运行这个 task 的线程 runner

并且可以使用 UNsafe 直接改变 task 对象成员变量内存中的值

```java
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;

/** The underlying callable; nulled out after running */
private Callable<V> callable;
/** The result to return or exception to throw from get() */
private Object outcome; // non-volatile, protected by state reads/writes
/** The thread running the callable; CASed during run() */
private volatile Thread runner;
/** Treiber stack of waiting threads */
private volatile WaitNode waiters;

private static final long stateOffset;
private static final long runnerOffset;
private static final long waitersOffset;
static {
  try {
    UNSAFE = sun.misc.Unsafe.getUnsafe();
    Class<?> k = FutureTask.class;
    stateOffset = UNSAFE.objectFieldOffset
      (k.getDeclaredField("state"));
    runnerOffset = UNSAFE.objectFieldOffset
      (k.getDeclaredField("runner"));
    waitersOffset = UNSAFE.objectFieldOffset
      (k.getDeclaredField("waiters"));
  } catch (Exception e) {
    throw new Error(e);
  }
}
```

## FutureTask 构造器

```java
public FutureTask(Callable<V> callable) {
  if (callable == null)
    throw new NullPointerException();
  this.callable = callable;
  this.state = NEW;       // ensure visibility of callable
}
```



# Thread.start -> FutureTask.run

1. state == NEW, cas(runner, null, Thread.currentThread)
2. 获取来自构造器的 callable，调用 call 执行
3. set(result)
   1. cas(state, NEW, COMPLETING)，
   2. cas 成功：outcome = result
   3. unsafe.put(state, NORMAL)
   4. finishCompletion()：自旋 cas(waiters, q, null)，成功后，唤醒（unpark） q（原先 waiters 指向的队列）内所有的线程。

```java
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable; // 来自于构造器
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

## FutureTask.set

```java
protected void set(V v) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = v;
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
        finishCompletion();
    }
}
```

### FutureTask.finishCompletion

```java
private void finishCompletion() {
    // assert state > COMPLETING;
    for (WaitNode q; (q = waiters) != null;) {
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
            for (;;) {
                Thread t = q.thread;
                if (t != null) {
                    q.thread = null;
                    LockSupport.unpark(t);
                }
                WaitNode next = q.next;
                if (next == null)
                    break;
                q.next = null; // unlink to help gc
                q = next;
            }
            break;
        }
    }

    done();

    callable = null;        // to reduce footprint
}
```



# FutureTask.get

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}
```

## FutureTask.awaitDone

- timed: false，nanos: 0

for 循环 3 次：

1. q == null，创建新节点 q，WaitNode.thread = Thread.currentThread
2. !queued，插入队头
3. LockSupport.park

任务执行完成，线程被唤醒，for 循环继续

1. s = state，if (s > COMPLETING)，return s
2. 回到 get 方法，执行 return report(s)
3. report 中，检查 s == NORMAL，return 结果（outcome）

```java
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
        else if (q == null)
            q = new WaitNode();
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        else
            LockSupport.park(this);
    }
}
```

## FutureTask.report

task 执行完毕，会唤醒阻塞的线程，执行 `return report(s)`

```java
private V report(int s) throws ExecutionException {
    Object x = outcome;
    if (s == NORMAL)
        return (V)x;
    if (s >= CANCELLED)
        throw new CancellationException();
    throw new ExecutionException((Throwable)x);
}
```



# FutureTask 小结

- 通过 `FutureTask.run`可以发现，多个线程运行 FutureTask 只有一个线程可以执行成功
- 通过 `get -> awaitDone`可以发现，多个线程调用 `get`会加入到 waiters 队列里面，并且 park（阻塞），直到运行结束，获得结果，waiters 里面的线程才会依次 unpark（唤醒），然后继续执行，获取到结果

> 以上结论已验证

补充：ThreadPoolExecutor.submit 方法可以有返回值的任务提交方法，都是将 Callable 和 Runnable 包装成 FutureTask 来完成的



# ThreadPoolExecutor 类

```java
Executor
  ExecutorService
  	AbstractExecutorService（abstract class）
  		ThreadPoolExecutor
```



## ThreadPoolExecutor 内部类

- Worker，继承 AQS，实现 Runnable，这意味着，能加锁，能运行。同时也是对线程的封装

- AbortPolicy，DiscardPolicy，DiscardOldestPolicy，CallerRunsPolicy，全部实现 RejectedExecutionHandler，实现线程全忙，队列满之后的拒绝策略  

### Worker 线程的包装类

```java
Worker(Runnable firstTask) {
    setState(-1); // inhibit interrupts until runWorker
    this.firstTask = firstTask;
    this.thread = getThreadFactory().newThread(this); // 传入是 worker 对象，不是任务 runnable
}

// 所有成员变量
/** Thread this worker is running in.  Null if factory fails. */
final Thread thread; 
/** Initial task to run.  Possibly null. */
Runnable firstTask;
/** Per-thread task counter */
volatile long completedTasks;
```



# ThreadPoolExecutor 构造及参数

```java
public ThreadPoolExecutor(int corePoolSize, // 核心线程数
                          int maximumPoolSize, // 最大线程数
                          long keepAliveTime, // 保活时间
                          TimeUnit unit, // 保活时间单位
                          BlockingQueue<Runnable> workQueue, // 任务阻塞队列
                          ThreadFactory threadFactory, // 线程工厂
                          RejectedExecutionHandler handler) { // 拒绝处理
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

# ThreadPoolExecutor.execute 

## 变量 ctl 及其相关变量

- COUNT_BITS：29
- CAPACITY：1 左移 29 位减一，(2^29) - 1，二进制就是 00011111 11111111 11111111 11111111
- workerCountOf(ctl)，worker 的数量，就是 ctl & CAPACITY，即低 29 位的数据
- runStateOf(ctl)，状态值，ctl & ~CAPACITY，高 3 位的数据，其他29 位都为 0
- ctlOf(rs, wc)，ctl 数据合成，rs 是 Running State，wc 是 Worker Count

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

再看 execute 方法

```java
public void execute(Runnable command) {
  if (command == null)
    throw new NullPointerException();
  
  int c = ctl.get();
  if (workerCountOf(c) < corePoolSize) { // worker 数小于核心线程数
    if (addWorker(command, true)) // 添加新 worker 执行 cmd，第二个参数代表是否核心（true 核心）
      return;
    c = ctl.get();
  }
  
  // 执行到这，说明核心线程数满
  if (isRunning(c) && workQueue.offer(command)) { // 线程池运行中，添加 cmd 进阻塞队列
    int recheck = ctl.get(); // 添加成功后如果线程池不运行了，删除 cmd，把 cmd 给拒绝策略处理
    if (! isRunning(recheck) && remove(command))
      reject(command);
    // 如果 worker 数为 0，添加一个线程
    // 这解释了核心数为 0，添加任务到有空间的阻塞队列，依然会触发创建线程执行任务
    else if (workerCountOf(recheck) == 0) 
      addWorker(null, false);
  }
  // 线程池不在运行态，或者添加阻塞队列失败会到这
  else if (!addWorker(command, false))  // 添加新 worker，非核心线程
    reject(command); // 添加线程失败（达到最大数），执行拒绝策略
}

// reject
final void reject(Runnable command) {
  handler.rejectedExecution(command, this);
}
```



## ThreadPoolExecutor.addWorker

- firstTask：当前加入的任务
- core：是否是添加核心线程

```java
private boolean addWorker(Runnable firstTask, boolean core) {
  retry:
  for (;;) {
    int c = ctl.get();
    int rs = runStateOf(c);

    // Check if queue empty only if necessary.
    if (rs >= SHUTDOWN && // shutdown，stop，tidying，terminated
        ! (rs == SHUTDOWN && // stop||tidying||terminated||firstTask!=null||workQueue empty
           firstTask == null &&
           ! workQueue.isEmpty()))
      return false;

    for (;;) {
      int wc = workerCountOf(c);
      // worker 数大于等于最大容量 (2^29)-1
      // 或者添加核心线程时大于核心线程数，非核心时代表最大线程数，都会添加失败
      if (wc >= CAPACITY ||
          wc >= (core ? corePoolSize : maximumPoolSize))
        return false;
      if (compareAndIncrementWorkerCount(c)) // ctl.cas(c, c + 1) 成功，跳出 retry 块
        break retry;
      // CAS 失败（ctl 值被其他线程改写）重读，如果是状态改写了，跳到 retry 重新循环，判断 rs
      // 状态没有被改写，下一次循环继续 ctl.cas(c, c + 1)
      c = ctl.get();  // Re-read ctl
      if (runStateOf(c) != rs)
        continue retry;
      // else CAS failed due to workerCount change; retry inner loop
    }
  }

  // 执行到这，代表 ctl.cas(c, c + 1) 成功，worker 数加 1，实际添加 worker
  boolean workerStarted = false;
  boolean workerAdded = false;
  Worker w = null;
  try {
    w = new Worker(firstTask); // 新 worker，firstTask 也可以是 null
    final Thread t = w.thread;
    if (t != null) {
      final ReentrantLock mainLock = this.mainLock;
			// 这个加锁是为了 workers 在多线程调用 execture->addworker 时 workers 的添加删除数据的线程安全
      mainLock.lock(); 
      try {
        // Recheck while holding lock.
        // Back out on ThreadFactory failure or if
        // shut down before lock acquired.
        int rs = runStateOf(ctl.get());

        if (rs < SHUTDOWN ||
            (rs == SHUTDOWN && firstTask == null)) {
          if (t.isAlive()) // precheck that t is startable
            throw new IllegalThreadStateException();
          workers.add(w);
          int s = workers.size();
          if (s > largestPoolSize) // 更新 largetPoolSize
            largestPoolSize = s;
          workerAdded = true;
        }
      } finally {
        mainLock.unlock();
      }
      if (workerAdded) {
        t.start(); // 启动线程
        workerStarted = true;
      }
    }
  } finally { // 处理线程工厂 newThread 失败的情况
    if (! workerStarted)
      // workers Set 里面 remove 刚刚加入的 worker
      // 自旋 cas 更新 ctl(wc)
      addWorkerFailed(w); 
  }
  return workerStarted;
}
```

## Worker.run

```java
// class Worker.run
public void run() {
  runWorker(this);
}

final void runWorker(Worker w) {
  Thread wt = Thread.currentThread();
  Runnable task = w.firstTask;
  w.firstTask = null;
  w.unlock(); // allow interrupts
  boolean completedAbruptly = true;
  try {
    while (task != null || (task = getTask()) != null) { // firstTask 非 null || 任务队列非空
      w.lock();
      // If pool is stopping, ensure thread is interrupted;
      // if not, ensure thread is not interrupted.  This
      // requires a recheck in second case to deal with
      // shutdownNow race while clearing interrupt
      if ((runStateAtLeast(ctl.get(), STOP) ||
           (Thread.interrupted() &&
            runStateAtLeast(ctl.get(), STOP))) &&
          !wt.isInterrupted())
        wt.interrupt();
      try {
        beforeExecute(wt, task);
        Throwable thrown = null;
        try {
          task.run();
        } catch (RuntimeException x) {
          thrown = x; throw x;
        } catch (Error x) {
          thrown = x; throw x;
        } catch (Throwable x) {
          thrown = x; throw new Error(x);
        } finally {
          afterExecute(task, thrown);
        }
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
```

## ThreadPoolExecutor.getTask

```java
private Runnable getTask() {
  boolean timedOut = false; // Did the last poll() time out?

  for (;;) {
    int c = ctl.get();
    int rs = runStateOf(c);

    // Check if queue empty only if necessary.
    // shutdown/stop/tidying/terminated && (stop/tidying/terminated || workQueue.isEmpty())
    // 即如果状态是 shutdown，那么 workQueue 必须空，条件才会成立
    // 这意味着，shutdown 状态，队列不为空，任务依然会被线程执行完
    if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
      decrementWorkerCount();
      return null;
    }

    int wc = workerCountOf(c);

    // 是否允许线程超时的条件 = 允许核心线程数超时标记 || 总活线程数大于核心线程数
    // Are workers subject to culling?
    boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

    // 需要 cas(wc, c, c-1) 的条件：
    // 总 worker 数大于最大线程数 || 允许线程超时且已超时
    // 以及在上面两个条件满足其一后，还要满足（worker 数大于 1 || 任务队列空）
    if ((wc > maximumPoolSize || (timed && timedOut))
        && (wc > 1 || workQueue.isEmpty())) {
      if (compareAndDecrementWorkerCount(c))
        return null;
      continue;
    }

    try {
      // 允许线程超时，poll 阻塞这个线程 keepAliveTime 的时间
      // 不允许线程超时，take 一直阻塞，典型的生产者与消费者模式
      Runnable r = timed ?
        workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
      workQueue.take();
      if (r != null)
        return r;
      // 执行到这，说明从队列获取的 r 为 null，来自 poll，线程阻塞（活）超时，继续下一次循环
      // 下次循环会满足上面 compareAndDecrementWorkerCount 的条件，并返回 null
      timedOut = true; 
    } catch (InterruptedException retry) {
      timedOut = false;
    }
  }
}
```





# 问题 -> 源码 -> 答案

## 任务提交到线程执行的主流程是怎么样的？

1. 执行或提交 Runnable
2. 根据条件选择新建 worker 执行 Runnable，或者加入任务队列
3. 新建 worker，加锁之后加入到 worker Set，Runnable 会被赋值到 worker.firstTask
4. 启动 worker.thread.start，执行 worker.run -> worker.runWorker
5. 获取 firstTask，或者 getTask 获取任务队列里面的任务（firstTask 不存在的情况下），执行 firstTask
6. 第 5 步是一个循环，线程在 执行 getTask时会由于阻塞队里的 take 或者 poll 方法而阻塞
7. poll 是一个超时阻塞，经过了线程保活时间后，激活线程 getTask 返回 null，退出循环
8. 继续执行，加锁，从 worker Set 里面删除 worker



## 线程是如何保活的，多余线程何时释放？

保活靠的阻塞队列 poll 方法的阻塞超时机制，阻塞 keepAliveTime，就说明线程活了 keepAliveTime。那除了核心线程，多余的线程又是如何释放的呢？参考源码：getTask，关键代码如下：

解析：

假设 allowCoreThreadTimeOut为 false，然后 1 核心 1 非核心线程 t1 和 t2完成 firstTask 后同时进入 getTask：

1. wc = 2，则 timed = true，timeOut 一开始为 false
2. 两个线程都阻塞在 poll，时间为 keepAliveTime
3. 超时，两线程都获取到 null 的 r，并且 timeOut = true
4. 下一次循环 wc = 2，两个线程 timed  = true，workQueue 空，进入 if 条件
5. cas(ctl, c, c - 1)，由于cas 的原子性，一个线程cas 成功，一个失败
6. cas 成功的线程返回 null，进行worker 移除操作
7. cas 失败的线程 continue，下一次循环，wc = 1，timed = false，不满足 if 条件，进入阻塞队列的 take 阻塞

```java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
...
        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```



## shutdown 和 shutdownNow 有什么区别？

### shutdown

```java
public void shutdown() {
  final ReentrantLock mainLock = this.mainLock;
  mainLock.lock();
  try {
    checkShutdownAccess();
    advanceRunState(SHUTDOWN); // 自旋修改状态：running -> shutdown
    interruptIdleWorkers();
    onShutdown(); // hook for ScheduledThreadPoolExecutor
  } finally {
    mainLock.unlock();
  }
  tryTerminate();
}

private void interruptIdleWorkers() {
  interruptIdleWorkers(false);
}

private void interruptIdleWorkers(boolean onlyOne) {
  final ReentrantLock mainLock = this.mainLock;
  mainLock.lock();
  try {
    for (Worker w : workers) {
      Thread t = w.thread;
      // 遍历所有的 worker，尝试获取锁。
      // 如果失败，说明在 worker.run -> runWorker 方法里还 lock 着，还在执行任务，不进行线程的中断设置
      // 这意味着任务会继续完成，并且这些未标记的线程还会继续从任务队列获取任务执行
      // trylock 成功，不在执行任务的线程会被中断标记
      // 这又得分为两个情况：
      // 情况 1：任务队列空，线程在阻塞时，或即将被阻塞时被设置；
      // 情况 2：任务队列非空，线程在执行完再去获取任务时被设置
      if (!t.isInterrupted() && w.tryLock()) { 
        try {
          t.interrupt();
        } catch (SecurityException ignore) {
        } finally {
          w.unlock();
        }
      }
      if (onlyOne)
        break;
    }
  } finally {
    mainLock.unlock();
  }
}
```

情况 1：线程阻塞时候被标记中断，异常捕获，然后 timeOut = false，重新循环，然后逻辑就和即将被阻塞相同了。会满足 `if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) ` 条件，wc 减一，返回 null，删除这个 worker。

其他还在执行任务的线程，执行完毕后进入 getTask 时候也会进入这个条件，然后 wc 减一，返回 null，删除这个 worker，直到线程池的 workers 全部删除。在 `processWorkerExit` 里面还会继续调用 `tryTerminate` 根据 workerCount（wc） 的数量是否为 0 把状态改为 TIDYING 和 TERMINATED。

`tryTerminate` 在 `shutdown`里也会调用，但是如果 wc 不是 0，就不会去把状态改为 TIDYING 和 TERMINATED。

情况 2：任务队列非空的情况，`if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())`这个条件不满足，`if ((wc > maximumPoolSize || (timed && timedOut)) && (wc > 1 || workQueue.isEmpty()))`这个条件一般也无法满足，因此会从任务队列获取任务，但是会抛出异常，标记 `timedOut = false` ，但是由于这个 try-catch 在 for 循环里面，于是会继续下一次循环。

任务队列的阻塞机制是通过 Condition.await 实现的，在抛出异常后，线程的中断标记会被恢复“非中断”，于是这个线程在下一次循环又可以获取任务进行执行，直到任务队列为空。满足 `if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty()))` 这个条件为止，于是逻辑就和情况 1 一样了。



getTask 参考代码：

```java
private Runnable getTask() {
  boolean timedOut = false; // Did the last poll() time out?

  for (;;) {
    int c = ctl.get();
    int rs = runStateOf(c);

    // Check if queue empty only if necessary.
    if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
      decrementWorkerCount();
      return null;
    }

    int wc = workerCountOf(c);

    // Are workers subject to culling?
    boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

    if ((wc > maximumPoolSize || (timed && timedOut))
        && (wc > 1 || workQueue.isEmpty())) {
      if (compareAndDecrementWorkerCount(c))
        return null;
      continue;
    }

    try {
      Runnable r = timed ?
        workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
      workQueue.take();
      if (r != null)
        return r;
      timedOut = true;
    } catch (InterruptedException retry) {
      timedOut = false;
    }
  }
}
```



### shutdownNow

```java
public List<Runnable> shutdownNow() {
  List<Runnable> tasks;
  final ReentrantLock mainLock = this.mainLock;
  mainLock.lock();
  try {
    checkShutdownAccess();
    advanceRunState(STOP); // 自旋修改状态：running -> stop
    interruptWorkers(); // 中断标记所有 workers 包括在执行任务的 worker
    tasks = drainQueue(); // 把任务队列里面未执行的都放到新的 List 里面，返回给调用线程
  } finally {
    mainLock.unlock();
  }
  tryTerminate();
  return tasks;
}

private void interruptWorkers() {
  final ReentrantLock mainLock = this.mainLock;
  mainLock.lock();
  try {
    for (Worker w : workers)
      w.interruptIfStarted();
  } finally {
    mainLock.unlock();
  }
}

// Worker
void interruptIfStarted() {
  Thread t;
  // getState() >= 0 这包括了在执行任务和不在执行任务的
  // 所以就是把所有的线程都标记了中断
  if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
    try {
      t.interrupt();
    } catch (SecurityException ignore) {
    }
  }
}
```

状态改为了 STOP，在 getTask 方法里面就满足了 `if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty()))` 这意味着只要是执行完任务的线程，就会被移除，不用管任务队列是否空。不过在执行任务的线程，虽然线程标记了中断，但是任务如果不出意外，都是执行完后再移出 worker Set 的。

当然如果某个任务本身就是不死的，那中断也没有用



**总结：**

**shutdown 会标记状态为 SHUTDOWN，但是会要求当前所有的 worker 先完成所有的任务，包括任务队列里面的任务之后再移除，然后再把状态改为 TIDYING 以及 TERMINATED**

**shutdownNow 会标记线程池状态为 STOP，会直接标记所有在的 worker 为中断，且不需要 worker 去执行完任务队列里面的任务，而是直接返回未执行完的任务给调用线程。但是 worker 正在执行的任务还是得执行完，然后再被移除。之后状态再会被改为  TIDYING 以及 TERMINATED**



