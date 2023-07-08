---
title: Android Handler 详解
categories: Android
---

## 概述
为了避免多个线程同时更新 UI，导致不可预知的错误；所以现今几乎所有的 GUI 框架都只允许在主线程修改 UI；因此这些框架都选择了消息驱动编程模型；

消息驱动编程模型有以下几个组件：
1. 消息队列：存储待处理的消息
2. 分发器：将不同事件分发到不同的业务逻辑单元
3. 消息通道： 分发器和处理器之间的联系通道
4. 事件处理器：实现业务逻辑

## Handler
Handler 充当了 **分发器** 和 **处理器** 的功能；

```java
public class Handler {
    // 管理事件的循环，它的初始化有一些玄机，在讲到 Looper 的时候会展开
    final Looper mLooper;
    // 存储待处理的事件
    final MessageQueue mQueue;
    // 处理事件的回调
    @UnsupportedAppUsage
    final Callback mCallback;
}
```
上面的几个成员变量会在构造函数中进行初始化；

### 分发器
Handler 作为 **分发器** 实现了以下三个功能：
1. 投递消息
2. 删除消息
3. 查询消息

#### 投递消息
```java
post(Runnable): boolean
postAtTime(Runnable, long): boolean
......
sendMessageAtTime(Message, long)
```

以`post` 开头的系列方法，都是将`Runnable`对象封装成 `Message` 然后调用 `sendMessageAtTime` 放到队列中；

`sendMessageAtTime` 实际调用了 `enqueueMesssage` 方法把消息放入队列；
```java
    private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
            long uptimeMillis) {
        // 把 target 指向自身，在 looper 中进行事件处理
        msg.target = this;
        msg.workSourceUid = ThreadLocalWorkSource.getUid();

        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

#### 删除消息
```java
removeMessages(int)
removeMessages(int, Object)
......
```
以`remove`开头的系列方法，都是移除消息的方法；实际上这些方法都是交给了消息队列去处理；

#### 查询消息
```java
boolean hasMessages(int)
boolean hasMessagesOrCallbacks()
······
```
以 `has` 开头的系列方法，都是查询消息的方法；实际上也是交给了消息队列去处理；

### 处理器
```java
public void dispatchMessage(@NonNull Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```
处理逻辑如下：
1. 如果消息本身包含处理逻辑，则消息自己处理；
2. 如果创建 Handler 时传入了回调，则调用回调处理；
3. 如果上面两种都不满足，则调用 `handleMessage` 方法处理，这个方法是个空方法，没有实现，通常交由子类实现；

## 消息
在谈论 `Looper` 之前，需要理解消息；
```java
public final class Message implements Parcelable {
    // 标记消息，可以通过它查找消息
    public int what;
    // 标记在何时处理该消息，使用 SystemClock.uptimeMillis 作为基线
    public long when;
    // 标记由哪个 Handler 处理该消息
    Handler target;
    // 消息内执行的逻辑
    Runnable callback;
    // 指向下一个消息，可能为空
    Message next;
}
```

### 同步屏障
实际上消息队列中一共存在三种消息，普通消息，同步屏障和异步消息；
普通消息就是开发者平常所接触的消息；
如果一个消息的`target`为空，也就是没有指定 Handler 处理的话，那这个消息被视为同步屏障；
如果一个消息的 `flag` 包含异步标记，则视为异步消息；
```java
public boolean isAsynchronous() {
        return (flags & FLAG_ASYNCHRONOUS) != 0;
}
```
同步屏障是为了在某种情况不处理普通消息的同时，不错过异步消息；
例如在视图无效的情况下，在调用`invalidate`之后发布的消息将通过同步屏障挂起，直到可以绘制下一帧为止。同步屏障确保在恢复之前完全处理无效请求。
异步消息不受同步障碍的影响。它们通常表示中断、输入事件和其他信号，即使其他工作已暂停，这些信号也必须独立处理。

### 消息的复用
在消息队列中必定存在大量消息，但是消息的创建与销毁伴随着大量对象的创建与销毁；因此应该对消息进行复用；

```java
// 操作复用池的锁对象
public static final Object sPoolSync = new Object();
// 由于 Message 本身有 next 对象，所以把 Message 作为复用池的实现；
private static Message sPool;
// 标记当前复用池的消息个数
private static int sPoolSize = 0;
// 最大的复用个数
private static final int MAX_POOL_SIZE = 50;
// 在回收消息之前是否对消息进行检查
private static boolean gCheckRecycle = true;

// 回收消息
void recycleUnchecked() {
        // 当消息在复用池时，标记为正在使用
        // 清除其他状态
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = UID_NONE;
        workSourceUid = UID_NONE;
        when = 0;
        target = null;
        callback = null;
        data = null;

        // 把当前消息作为复用池的头节点
        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
}
// 获取消息
public static Message obtain() {
    // 如果当前复用池不为空，则从复用池中获取消息，否则新建消息对象
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
}
```

## Looper
`Looper` 作为消息通道，是分发器和处理器之间的联系通道；
消息驱动编程模型是基于消息循环的，因此`Looper`需要实现以下几个功能：
1. Looper 能够对消息循环进行管理，例如开启和关闭循环；
2. Looper 的消息循环一定在指定线程执行；

### Looper 的初始化
```java
// 私有的构造方法
private Looper(boolean quitAllowed) {
    // 初始化消息队列
    mQueue = new MessageQueue(quitAllowed);
    // 保留当前线程的引用
    mThread = Thread.currentThread();
}

// 由于构造方法是私有的，因此只能通过 prepare 方法进行初始化
public static void prepare() {
        prepare(true);
}

// ThreadLocal 保存的变量是线程相关的，本线程保存的，其他线程无法访问，具体原理请读者自行学习；
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

private static void prepare(boolean quitAllowed) {
    // 每个线程只能调用此方法一次
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

### 消息循环的管理
```java
    // 开启消息循环
    public static void loop() {
        // 获取当前线程的 Looper
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        if (me.mInLoop) {
            Slog.w(TAG, "Loop again would have the queued messages be executed"
                    + " before this one completed.");
        }

        me.mInLoop = true;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        // Allow overriding a threshold with a system prop. e.g.
        // adb shell 'setprop log.looper.1000.main.slow 1 && stop && start'
        final int thresholdOverride =
                SystemProperties.getInt("log.looper."
                        + Process.myUid() + "."
                        + Thread.currentThread().getName()
                        + ".slow", 0);

        me.mSlowDeliveryDetected = false;

        // 开启消息循环，如果消息队列退出则跳出循环
        for (;;) {
            if (!loopOnce(me, ident, thresholdOverride)) {
                return;
            }
        }
    }

    // 单个消息的处理，此处逻辑较多，因此删除了部分不关心的代码
    private static boolean loopOnce(final Looper me,
            final long ident, final int thresholdOverride) {
        Message msg = me.mQueue.next(); // might block
        if (msg == null) {
            // 无法获取消息，表示消息队列已经退出
            return false;
        }

        try {
            // 把消息分发给 Handler
            msg.target.dispatchMessage(msg);
            if (observer != null) {
                observer.messageDispatched(token, msg);
            }
            dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
        } catch (Exception exception) {
            if (observer != null) {
                observer.dispatchingThrewException(token, msg, exception);
            }
            throw exception;
        } finally {
            ThreadLocalWorkSource.restore(origWorkSource);
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }

        // 消息完成消费，回收消息
        msg.recycleUnchecked();

        return true;
    }

// 消息循环的退出，实际上是交由消息对立处理的
public void quit() {
        mQueue.quit(false);
    }

public void quitSafely() {
    mQueue.quit(true);
}   
```

## 消息队列
承担保存消息的功能，按理说应该实现非常简单，但是它是消息驱动编程在 Android 平台上最复杂的部分；

关于消息队列可以参考以下两篇文章：

https://www.jianshu.com/p/bfe2e380dc47

https://www.kancloud.cn/alex_wsc/android-deep2/413391