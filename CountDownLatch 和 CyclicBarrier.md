# CountDownLatch

减数同步辅助工具，初始化参数为一个整数 N，await() 可以阻塞当前线程，countdown() 执行操作为 N 减一，await() 不会影响计数

主要两个作用：

- 可以利用 countdown() 后值为 0，来启动所有所有 await() 的线程
- 也可以在线程执行完后 countdown()，直到为 0 后，触发相同 latch 对象使用 await() 阻塞的线程继续执行

注意：CountDownLatch 对象无法 reset，即值为 0 后无法再使用

Sample：

```
public static void execute() {
    CountDownLatch startSignal = new CountDownLatch(1);
    CountDownLatch doneSignal = new CountDownLatch(10);
    for (int i = 0; i < 10; i++) {
        new Thread(new Worker(i, startSignal, doneSignal)).start();
    }

    System.out.println("Main thread say: start!");
    startSignal.countDown();
    System.out.println("Main thread is still here");
    try {
        doneSignal.await(); // this will block main thread until work threads done
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Main thread is done!");
}

static class Worker implements Runnable {

    CountDownLatch startSignal;
    CountDownLatch doneSignal;
    private int id;

    public Worker(int id, CountDownLatch startSignal, CountDownLatch doneSignal) {
        this.startSignal = startSignal;
        this.doneSignal = doneSignal;
        this.id = id;
    }

    @Override
    public void run() {
        try {
            startSignal.await();
            System.out.println("Thread " + id + " is working....");
            Thread.sleep(5000);
            System.out.println("Thread " + id + " is done");
            doneSignal.countDown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

// output:
Main thread say: start!
Main thread is still here
Thread 0 is working....
Thread 2 is working....
Thread 1 is working....
Thread 4 is working....
Thread 7 is working....
Thread 5 is working....
Thread 3 is working....
Thread 9 is working....
Thread 8 is working....
Thread 6 is working....
Thread 4 is done
Thread 0 is done
Thread 2 is done
Thread 1 is done
Thread 5 is done
Thread 7 is done
Thread 8 is done
Thread 9 is done
Thread 3 is done
Thread 6 is done
Main thread is done!
```

## 原理简要
初始化 CountDownLatch(N)，Sync 对象(继承 AQS) 的 state 初始化成 N。

调用 await() 的线程创建队列头节点（waitStatus 为 SIGNAL），线程节点（nextWaiter 指向 SHADE 节点）加入等待队列，然后 park 这个线程。

其他线程调用 coundown() 一次就会 N-1，直到 state 等于 0，调用 await() 的线程 unpark 继续执行，并把 Sync 之前创建的节点（nextWaiter 指向 SHARED 节点）设置为队列 HEAD

# CyclicBarrier

与 CountDownLatch 的递减计数，CyclicBarrier 是累加计数。使用的是 await() 阻塞线程数作为累加信号，当阻塞的线程到达 parties 的时候，线程全部跳出阻塞继续运行

构造器有两个：

- CyclicBarrier(int parties)
- CyclicBarrier(int parties, Runnable barrierAction)

parties 代表有多少线程等候在障碍处，也就意味着多少个线程内使用了 CyclicBarrier#await

barrierAction 代表当 parties 个线程都抵达了障碍处后会里面执行的行为。注意：这个 Action 是最后一个调用 await() 的线程去执行的

Sample:

```
static class Worker implements Runnable {

    private CyclicBarrier cyclicBarrier;

    public Worker(CyclicBarrier cyclicBarrier) {
        this.cyclicBarrier = cyclicBarrier;
    }

    @Override
    public void run() {
        try {
            System.out.println(Thread.currentThread().getName() + " is working...");
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + " await()");
            cyclicBarrier.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + " finish");
        }
    }
}

public static void execute() {
    CyclicBarrier cyclicBarrier = new CyclicBarrier(11, new Runnable() {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " last await()");
        }
    });

    System.out.println(Thread.currentThread().getName() + " is working...");

    for (int i = 0; i < 10; i++) {
        new Thread(new Worker(cyclicBarrier)).start();
    }

    try {
        System.out.println(Thread.currentThread().getName() + " await()");
        cyclicBarrier.await();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (BrokenBarrierException e) {
        e.printStackTrace();
    }

    System.out.println(Thread.currentThread().getName() + " finish");
}

// output:
main is working...
Thread-0 is working...
Thread-1 is working...
Thread-2 is working...
Thread-3 is working...
Thread-4 is working...
Thread-5 is working...
Thread-6 is working...
Thread-7 is working...
Thread-8 is working...
main await()
Thread-9 is working...
Thread-0 await()
Thread-6 await()
Thread-5 await()
Thread-4 await()
Thread-3 await()
Thread-2 await()
Thread-1 await()
Thread-7 await()
Thread-9 await()
Thread-8 await()
Thread-8 last await()
Thread-8 finish
main finish
Thread-0 finish
Thread-6 finish
Thread-5 finish
Thread-4 finish
Thread-3 finish
Thread-2 finish
Thread-1 finish
Thread-9 finish
Thread-7 finish
```

## 原理简要
CyclicBarrier 内部使用 ReentrantLock 和 ReentrantLock.Condition 实现

Condition 的使用类似于 Object.wait/notify/notifyAll，底层 ConditionObject(实现 Condition) 实现了一个等待队列。

线程调用 lock 之后，就可以使用 condition.await() 进行阻塞，其他线程使用相同 lock 和 condition 的可以使用 notify/notifyAll 进行唤醒

CyclicBarrier 的初始化时会初始化一个 count，每次 await() 会进行 lock.lock，然后count--，并且调用相应的 codition.await()，最后进行 lock.unlock。

等到 count 减到 0，减到 0 的线程还会 run barrierCommand，然后调用 lock.signalAll() 唤醒所有阻塞的线程