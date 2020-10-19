# IntentService

1. startService/bindService -> onCreate，会启动一个 HanderThread

   1. HandlerThread，继承自 Thread，里面开启了一个 Looper

   ```java
   @Override
   public void run() {
     mTid = Process.myTid();
     Looper.prepare(); // 为这个线程新建一个 Looper，存在 ThreadLocal 里面，即线程的 ThreadLocalMap 里面
     synchronized (this) {
       mLooper = Looper.myLooper(); // 获取之前创建的 Looper
       notifyAll();
     }
     Process.setThreadPriority(mPriority);
     onLooperPrepared(); // hook 方法
     Looper.loop(); // 开启消息队列的循环
     mTid = -1;
   }
   ```

2. 获取 HanderThread 的 Looper，并构造一个 mServiceHandler:Handler

3. 生命周期 onStartCommand -> onStart，构造一个 msg(startId, intent)，通过 mServiceHandler 发送

4. 在 Looper.loop 的循环中，这个消息会被获取到，并调用 msg.target.dispathMessage(msg)，target 为 Handler（mServiceHandler），然后到 Handler.handleMessage

5. 这会调用 onHandleIntent(msg.intent)，接着是 stopSelf(msg.startId)，一个是 hook 方法，表示需要在这个线程中执行的任务，执行完毕后关闭这个 Service

   ```java
   private final class ServiceHandler extends Handler {
     public ServiceHandler(Looper looper) {
       super(looper);
     }
   
     @Override
     public void handleMessage(Message msg) {
       onHandleIntent((Intent)msg.obj);
       stopSelf(msg.arg1);
     }
   }
   ```

   