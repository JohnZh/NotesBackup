# 从 App 启动到 UI 的显示

分析会从下面几个点去进行

1. App 启动与 Activity 生命周期方法的执行
2. 与UI 相关的类与 Activity 生命周期方法的关系



# App 启动入口

不谈 Launcher 或者其他 App 启动一个 App 的过程，仅仅从 Zygote 进程 fork 之后反射调用 ActivityThread.main 开始分析。



## AMS 通过 socket 向 Zygote 进程发送 fork 请求

```java
AMS.startProcessLocked
	Process.start
  	ZygoteProcess.start
  		ZygoteProcess.startViaZygote
```

`ZygoteProcess.startViaZygote` 会构造 fork 进程时候需要的参数列表 `args:ArrayList<String>`，然后调用 `openZygoteSocketIfNeeded(abi)` 进行创建一个 LocalSocket 进行 socket 连接，并配备当前设备支持的 abi 返回 `ZygoteState` 这个类是socket 连接、写出、关闭的包装类。

最后，再调用 `zygoteSendArgsAndGetResult(zygoteState, args)` 通过 socket 写出参数到目标 ServerSocket 并读取结果。

## Zygote 进程通过 socket 接收处理 fork 请求

`ZygoteInit.main` 会注册一个 LocalServerSocket，并在 `ZygoteInit.runSelectLoop` 里执行 while(true) 循环，在循环里调用 `LocalServerSocket.accept()` 阻塞监听来自其他进程的 socket 连接。

accept 到连接后将得到的 LocalSocket 包装成 `ZygoteConnection`，再下一次循环的时候执行 `ZygoteConnection.runOnce` 读取处理请求，这个方法会抛出 `ZygoteInit.MethodAndArgsCaller` 异常，`ZygoteInit.runSelectLoop` 继续向上抛出，最后在 `ZygoteInit.main`  里面处理：

```java
main() {
  ...
	} catch (MethodAndArgsCaller caller) {
  	caller.run(); 
	}
	...
}
```

实际上是调用了 MethodAndArgsCaller.mMethod，而这个方法就是新 App 的 `ActivityThread.main`

### runOnce 方法 fork 进程后的处理

forkAndSpecialize 会返回 pid 2 次，一次是 0，代表子进程接收到的结果；一次非 0 代表父进程接收到的结果，小于 0 代表失败。

父进程的处理：通过 socket 将得到的 pid，即子进程 id 和 usingWrapper 发送给 socket 客户端，即 AMS。AMS 继续执行

子进程的处理：`handleChildProc` -> `RuntimeInit.zygoteInit` -> `RuntimeInit.applicationInit` -> `RuntimeInit.invokeStaticMain` 反射获得 ActivityThread.main 方法，然后构造并抛出 MethodAndArgsCaller 异常

```java
runOnce() {
	pid = Zygote.forkAndSpecialize(parsedArgs...);
  // forkAndSpecialize 会返回 2 次，一次是 0，代表子进程，一次非 0 代表父进程
  try {
    if (pid == 0) {
      // in child
      IoUtils.closeQuietly(serverPipeFd);
      serverPipeFd = null;
      handleChildProc(parsedArgs, descriptors, childPipeFd, newStderr);

      // should never get here, the child is expected to either
      // throw ZygoteInit.MethodAndArgsCaller or exec().
      return true;
    } else {
      // in parent...pid of < 0 means failure
      IoUtils.closeQuietly(childPipeFd);
      childPipeFd = null;
      return handleParentProc(pid, descriptors, serverPipeFd, parsedArgs);
    }
  } finally {
    IoUtils.closeQuietly(childPipeFd);
    IoUtils.closeQuietly(serverPipeFd);
  }
}
```



## 执行 ActivityThread.main

main 方法做了两件比较重要的事情：

1. 准备主线程的 Looper，并且 loop，总线消息队列准备完毕
2. 创建 ActivityThread 对象，执行 ActivityThread.attach(false)

### ActivityThread.attach(false)

attach 做了下面几件比较重要的事情：

1. 获得 AMS 的代理 `ActivityManagerProxy`，调用 `attachApplication` 方法把这个 ActivityThread 的 唯一的ApplicationThread 对象通过 binder 机制绑定到 AMS，后续和 Activity 生命周期相关的来自 AMS 的消息都将通过 ApplicationThread 来接收，然后再交给 ActivityThread 处理
2. ViewRootImpl 添加 ConfigCallback，为了触发屏幕旋转时候 Activity 的生命周期回调：`onConfigurationChanged`

## AMS 与 ActivityThread 通信进行 App 生命周期回调

### 从 AMS 启动 Application

 `ActivityManagerProxy.attachApplication` -> `AMS.attachApplicationLocked`  这是一次进程间通信，AMS 进行相关初始化后又会调用 `thread.bindApplication` -> `ActivityThread.ApplicationThread.bindApplication` 触发 `ActivityThread.handleBindApplication` ，这个方法的作用：

1. 创建 Instrumentation 对象，这个类在维系系统和 App 之间的行为上起到很大作用。比如创建启动 Activity，Service，Application 等

2. 使用 `Instrumentation.newApplication` 创建的，调用 `Application.onCreate` 

   > 注意：onCreate 并非最开始调用的方法，在 newApplication 调用的时候，最开始会调用 Application.attach 进而调用到 Application.attachBaseContext

### 从 AMS 启动 Activity

`AMS.attachApplicationLocked` 在 `thread.bindApplication` 之后还会调用 

```java
ActivityStackSupervisor.attachApplicationLocked
	ActivityStackSupervisor.realStartActivityLocked
  	app.thread.scheduleLaunchActivity
```

跨进程通信到 ActivityThread

```java
ActivityThread.ApplicationThread.scheduleLaunchActivity
	ActivityThread.handleLaunchActivity
  	ActivityThread.performLaunchActivity
  	ActivityThread.handleResumeActivity
	  	ActivityThread.performResumeActivity
```

#### 创建初始化 Activity 执行其生命周期

ActivityThread.performLaunchActivity 重要操作：

1. `Instrumentation.newActivity`
2. `Activity.attach`
3. `Instrumentation.callActivityOnCreate`
4. `activity.performStart`
5. `Instrumentation.callActivityOnPostCreate`

ActivityThread.handleResumeActivity 重要操作：

1. `ActivityThread.performResumeActivity` 即 `Activity.performResume`



# 与 UI 相关类的介绍

- Window：唯一子类 PhoneWindow

- WindowManager：接口，实现 ViewManger 接口，也是用于和系统服务 Window Manager 通信的接口

- WindowManagerImpl：WindowManager 接口的具体实现类

- WindowManagerGlobal：WindowManagerImpl 内部的具体实现类，WindowManagerImpl 只是它的包装

- View：UI 相关展示的最小单元，所有的控件的父类

- ViewGroup：继承 View，实现 ViewParent 和 ViewManager，能包含子 View，所有布局控件的父类
- DecorView：本质是一个 FrameLayout，PhoneWindow 的成员变量
- ViewRootImpl：直接负责 DecorView 及其子 View 的测量，布局，绘制

# UI 相关类与 Activity 生命周期

### Activity.attach

创建 Activity 后，在执行 `Activity.attach` 的时候：

1. 会创建 PhoneWindow
2. 会 `getSystemService` 获取系统 WindowManager，并会调用 `PhoneWindow.setWindowManager` 设置给 PhoneWindow

### Activity.onCreate

在执行 `Activity.onCreate`，以及执行 `Activity.setContentView` 的时候：

1. 实际上是执行 `PhoneWindow.setContentView` 

2. 在 `PhoneWindow.setContentView` 里会创建 DecorView，同时把 PhoneWindow 里面另外两个成员变量赋值

   1. mContentRoot，DecorView 的唯一子 View，会根据 Window.mLocalFeatures 的值来选择系统布局

      > Window.mLocalFeatures 通过 Window.requestFeature 设置

   2. mContentParent，mContentRoot 布局中 id 为 `com.android.internal.R.id.content` 的子 View，一般也是一个 FrameLayout

3. 在 `PhoneWindow.setContentView` 的最后会把 Activity 的布局通过 `LayoutInflater.inflate(layoutResID, mContentParent)` 添加到 mContentParent 下面

### Activity.onResume & onPostResume

在执行 `handleResumeActivity` 时，当执行完 `performResumeActivity` 之后，实际上 `Activity.onResume & onPostResume` 都已经执行完了。接着继续执行：

1. 获取 Activity 里面的 PhoneWindow 的 DecorView，`decor`，把可见性设置为 INVISIBLE
2. 获取 Activity.mWindowManager，和 PhoneWindow 里面的是同一个。获取 PhoneWindow 的属性 `l`
3. 调用 `WindowManager.addView(decor, l)`，实际是调用 `WindowManagerImpl.addView`，或者说是调用 `WindowManagerGlobal.addView`，这会创建一个 ViewRootImpl 对象，由 ViewRootImpl 来执行 DecorView 及其所有子 View 的测量，布局，绘制
4. 最后，调用 `Activity.makeVisible` 把 DecorView 的可见性设置为 VISIBLE

**综上，UI 就给用户显示出来了**

补充：`handleResumeActivity` 的最后还会告诉 AMS 这个 Activity 已经处于 Resumed 状态，需要更新它保存在 AMS 里面的状态



# DecorView 的测量布局绘制的起点

```java
WindowManager.addView(decor, l)
	WindowManagerImpl.addView
		WindowManagerGlobal.addView
			root = new ViewRootImpl(view.getContext(), display)
			root.setView(view, wparams, panelParentView)
			  mView = view;	
				ViewRootImpl.requestLayout
				view.assignParent // DecorView 绑定 ViewRootImpl
```

## ViewRootImpl.requestLayout

```java
public void requestLayout() {
  if (!mHandlingLayoutInLayoutRequest) {
    checkThread();
    mLayoutRequested = true;
    scheduleTraversals();
  }
}

void scheduleTraversals() {
  mChoreographer.postCallback(
    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
}

final TraversalRunnable mTraversalRunnable = new TraversalRunnable();
final class TraversalRunnable implements Runnable {
  @Override
  public void run() {
    doTraversal();
  }
}

void doTraversal() {
  ...
  performTraversals();
  ...
}
```

**调用流程：requestLayout -> scheduleTraversals -> doTraversal -> performTraversals**

## ViewRootImpl.performTraversals

```java
// api 22
private void performTraversals() {
  	measureHierarchy() { [lines: 1379, 1435, 2111]
        ...
        performMeasure() [lines: 1148, 1159, 1173]
        ...
    }
    ...
    performMeasure() [lines: 1781, 1807]
    ...
    performLayout() [line: 1843]
    ...
    performDraw() [line: 1982]
    ...
}
```



## 测量的起点

导致测量和重新测量的原因非常多，一旦发生了测量，那么就必须重新布局和绘制

**测量的诱因先不展开，先看测量的起点**

```java
// desiredWindowWidth 和 desiredWindowHeight 只是一个指代
childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);
childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);

// getRootMeasureSpec() 为了 DecorView 构造最初的“尺寸规格”，
// 即根据 Window 大小和参数构造出来的“尺寸规格”
// DecorView 会根据“尺寸规格” 和自己的布局设置确定自己的测量尺寸
private static int getRootMeasureSpec(int windowSize, int rootDimension) {
    int measureSpec;
    switch (rootDimension) {

    case ViewGroup.LayoutParams.MATCH_PARENT:
        // Window can't resize. Force root view to be windowSize.
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
        break;
    case ViewGroup.LayoutParams.WRAP_CONTENT:
        // Window can resize. Set max size for root view.
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
        break;
    default:
        // Window wants to be an exact size. Force root view to be that size.
        measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
        break;
    }
    return measureSpec;
}
```

### getRootMeasureSpec()

- 一般情况下 lp.width 和 lp.height，Window 的参数宽高都是 MATCH_PARENT，见 Window.mWindowAttributes 构造器
- MATCH_PARENT 的情况会构造一个(windowSize, EXACTLY) 的 measureSpec
- WRAP_CONTENT 的情况会构造一个(windowSize, AT_MOST) 的 measureSpec
- Window 大小设置成了固定值是构造 (固定值, EXACTLY) 的 measureSpec

### performMeasure() 

```
private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
    if (mView == null) {
        return;
    }
    Trace.traceBegin(Trace.TRACE_TAG_VIEW, "measure");
    try {
        mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_VIEW);
    }
}
```

核心逻辑就是 `mView.measure(childWidthMeasureSpec, childHeightMeasureSpec)` ，其中 mView 是 DecorView

#### 