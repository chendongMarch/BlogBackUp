---
layout: post
title: Handler源码分析
category: Android
tags:
  - Android
  - SourceCode
keywords:
  - Android
  - Handler
abbrlink: 541527599
date: 2015-11-17 00:00:00
---


1. Handler意思是处理者，它是android特有的用来消息处理的一个类。使用它可以解决很多android中常见的问题.


2. Handler体现的是一种消息的发送与处理异步进行的机制，消息发送时即刻返回，将消息加入队列中，另一端looper负责循环取出消息进行操作。


3. 需要注意的是Handler可以发送Message也可以发送Runnable对象。但是消息处理必定是在Handler所在的线程中，也就是说UI线程中的Handler发送的消息也是在UI线程中执行，所以同样不能执行耗时操作。

<!--more-->

## 使用场景：

1. 发送延时消息，执行延时任务，使用线程睡眠的方式过于粗糙。。通常是用Handler发送延时消息或者使用Timer来完成延时操作。


2. 在子线程“操作”UI，众所周知子线程是不能操作UI的，想要在子线程的任务执行完之后更改UI,就需要在子线程向主线程发送消息让主线程来修改UI。


3. 结合异步任务实现任务回调，这个实际上是等同于2的，因为异步任务内部也是子线程，使用Handler+异步任务可以实现请求的回调，但是通常我们更加偏向于接口回调的方式，而不是传递Handler。


4. 在子线程中使用Handler,对任务进行串行化处理，可以更加高效的管理任务的执行。


## Handler原理图解：
![1](http://7xtjec.com1.z0.glb.clouddn.com/handler.png)


## Looper类

### 介绍
- Looper类负责在消息队列的另一端取出消息进行处理，Handler采用消息的先进先出原则。

### 成员变量

- Looper中的成员变量ThreadLocal，提供线程内部的局部变量，在本线程内随时随地可取，隔离其他线程。查了很多资料，[这里有很好的的解释](http://qifuguang.me/2015/09/02/%5BJava%E5%B9%B6%E5%8F%91%E5%8C%85%E5%AD%A6%E4%B9%A0%E4%B8%83%5D%E8%A7%A3%E5%AF%86ThreadLocal/)，大家可以去膜拜一下。


- Looper类中，首先ThreadLocal变量是private static的，每个线程只能有一个Looper对象，在我看来，使用ThreadLocal是为了对每个线程的Looper进行管理。使用ThreadLocal存储Looper就可以很方便的隔离其他线程随时存取本线程的Looper对象，结合后面的代码，发现创建Looper时会使用当前线程为键，新创建的Looper为值存放到ThreadLocal中，prepare()又会进行判断是否已经创建了该线程的Looper,这么说可能有些抽象，建议看完后面的代码，再回来看这段也许会更加清晰一点。

```java
//ThreadLocal.get() will return null unless you've called prepare().
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
private static Looper sMainLooper;  
// guarded by Looper.class（被这个类保护）
final MessageQueue mQueue;
final Thread mThread;
```

### 构造方法
- 构造方法,私有化的构造方法，并不允许外部调用，做的操作是创建一个MessageQueue和将mThread对象指向了本线程。这里有个重要的地方是，MessageQueue是Looper创建并首先持有的。


```java
private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
}
```

- 在Looper之中有一个`private static Looper sMainLooper; `变量，这个变量代表主线程（UI）的Looper，这也就是我们不需要在UI线程显式调用Looper.prepare()方法的原因，看下面的代码及注释：


```java
/**
//初始化当前线程Looper,将它标记为应用的主要Looper。应用程序的主Looer是Android环境创建的 ,所以你应该不需要自己调用这个函数
* Initialize the current thread as a looper, marking it as an
* application's main looper. The main looper for your application
* is created by the Android environment, so you should never need
* to call this function yourself.  See also: {@link #prepare()}
*/
public static void prepareMainLooper() {
//进行一次准备并且是不允许打断的
prepare(false);
synchronized (Looper.class) {
 if (sMainLooper != null) {
	 throw new IllegalStateException("The main Looper has already been prepared.");
 }
//获取这个looper
sMainLooper = myLooper();
}
}
//myLooper返回的是本线程的Looper
public static Looper myLooper() {
return sThreadLocal.get();
}
```


### 成员方法
- prepare方法,创建Looper放入sThreadLocal，同时要求每个线程只能有一个Looper对象，多创建会报异常。


```java
public static void prepare() {
prepare(true);
}
private static void prepare(boolean quitAllowed) {
if (sThreadLocal.get() != null) {
throw new RuntimeException("Only one Looper may be created per thread");
}
sThreadLocal.set(new Looper(quitAllowed));
}
```

- loop方法，使用该方法循环取出MessageQueue的消息。死循环取出消息，进行处理，处理完之后，同时消息会被回收掉，使用这个机制可以进行消息对象的复用，这里loop()结合MessageQueue的next()方法，形成了一个轮询的过程，详细的内容会在MessageQueue类的分析中来看，到时候会解释死循环轮询消息的机制。这里有其他对象的部分方法，暂且不去考虑，做下标记，看到Message的源码自然就会明白。

###留下的问题：

`Message msg = queue.next();`
`msg.target.dispatchMessage(msg);`
`msg.recycleUnchecked();`稍后我们会去解决。

```java
public static void loop() {
final Looper me = myLooper();
if (me == null) {
    throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
}
final MessageQueue queue = me.mQueue;
// Make sure the identity of this thread is that of the local process,
// and keep track of what that identity token actually is.
Binder.clearCallingIdentity();
final long ident = Binder.clearCallingIdentity();
//死循环取出消息，进行处理，当取不到消息时，return掉，等待下次调用loop(),同时消息会被回收掉，使用这个机制可以进行消息对象的复用，这里涉及了Message对象的部分方法，暂且不去考虑，做下标记，看到Message的源码自然就会明白
for (;;) {
    Message msg = queue.next(); // might block
    if (msg == null) {
// No message indicates that the message queue is quitting.
	return;
}
msg.target.dispatchMessage(msg);
// Make sure that during the course of dispatching the
// identity of the thread wasn't corrupted.
final long newIdent = Binder.clearCallingIdentity();
msg.recycleUnchecked();
    }
}
```



## Message类
### 成员变量

1. Message类十分类似于Bean类，毕竟它是用来承载消息的。


2. `what,arg1,arg2,obj`都是预设好用来盛放简单消息内容的变量，原文的注释中使用了`lower-cost`表示使用这些变量可以降低开销，就是不用自己创建维护这些变量，而且消息是可以被复用的，确实降低了开销。


3. target变量，是一个Handler,后面的代码中会具体介绍他的作用，这个变量也算这个类的核心了。


4. Runnable callback,代表一个任务，前面说过Handler可以发送简单消息也可以发送任务。


5. Message next，代表下一个Message的引用，由此形成了一个链表的结构。


6. 还有很多常量，变量不做一一介绍。下面有注释，不太详细，大家可以看完后面的再回来看也许会更加清晰。感兴趣的可以仔细去看源码注释。

```java
public int what;//消息标示
public int arg1; 
public int arg2;
public Object obj;
public Messenger replyTo;
public int sendingUid = -1;
//标示该消息正在使用之中
static final int FLAG_IN_USE = 1 << 0;
static final int FLAG_ASYNCHRONOUS = 1 << 1;
tatic final int FLAGS_TO_CLEAR_ON_COPY_FROM = FLAG_IN_USE;
int flags;
long when;
Bundle data;
Handler target;//Handler重点哦
Runnable callback;//一个任务可以用来发送，类似于Bean的一个属性
Message next;//下一个Message，用于构成链表
private static final Object sPoolSync = new Object();//同步对象
private static Message sPool;//链表根节点，他维护了一个空的Message队列用来复用
private static int sPoolSize = 0;//链表的长度
private static final int MAX_POOL_SIZE = 50;//链表最大数量
private static boolean gCheckRecycle = true;
```
### 成员方法obtain()
1. Message类中实现了obtain()的8个重载方法，提供了各种各样的参数，为的只是我们在外部用起来好用，所以大家使用Message时不妨多去用用其他方法，我到现在为止一般都是偏用参数为空的方法，其他的重载基本没看过，另外Handler也提供了大量的obtain方法，是的Message的重用和管理更加的方便了，所以千万不要去new Message。其他的方法也会回调空参方法然后进行一下外围的初始化，所以我们就来看看空参方法。
2. 代码不多，解释一下，首先这个变量sPoolSync，看名字就知道他是用来同步操作的，目的是当一个操作在获取Message是进行同步操作，避免其他的操作再来创建Message，否则会怎么样？Message本身形成了一个链表的结构，不进行同步就会，出现多个头，或者一个Message后面接入多个Message，那么后接入的就会覆盖掉，是这样吗？还是出错误。
3. 接下来是一个判断，我们看到sPool这个变量，它是一个Message对象，经过我的研究它是这个链表的头指针，<font color="red">同时sPool维护的是一个曾经创建过的空的可复用的Message队列，了解这一点至关重要，这是Message可复用的关键</font>。看看他是怎么操作的
4. 如果这个根sPool为空，则返回一个新的Message，Message的构造方法我看了，是个空的，所有的属性都在外部或者发送的那一刻设置。
5. 如果不是空，那么表示可复用Message队列可用，则取出头部的Message`Message m = sPool`,同时指针向后移动一位`sPool = m.next;`此时m是等于sPool的，以此表示sPool后移一位，然后将取出的Message.next置为null，因为这个消息是要拿来发送的，他此时可是指向的可复用Message队列的头，所以将它的next置为null，不然会怎么样？我们看Looper源码时，循环何时终止呢，就是在next==null时终止，如果不置为null,for循环是不会停止的，会把空的Message队列遍历一遍。最后可复用的链表长度减一`sPoolSize--;`
6. 其他的obtain方法我们不再去研究，大同小异吧。
```java
public static Message obtain() {
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
### Message的回收方法

- 看一下代码，了解了上面说的机制，这个代码不难理解。解释一下，如何回收一个Message,要有一个概念就是Message调用该方法回收的是自己，首先将自己的next指向sPool` next = sPool;`也就是说。此时自己已经链接到了可复用的Message队列头部（每次都叫他。可复用的Message队列。真麻烦），然后` sPool = this;`sPool指针又指向了 可复用的Message队列头部，队列长度++完成了消息的回收。在Looper中提到的`msg.recycleUnchecked();`这个方法就是在这里实现的。


```java
 void recycleUnchecked() {
.....//这里进行了好多代码，做了一个操作，将成员变量清空。
        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
```
###补充

- Message实现了Parcelable接口表明它是可以传输的。

```java
public final class Message implements Parcelable {}
```


## MessageQueue类

### 介绍
- 内部通过链接Message形成了一个消息队列，有两个比较核心的方法。`boolean enqueueMessage(Message msg, long when){}`和`Message next()`方法，Looper类中遗留的第二个问题`Message msg = queue.next();`会在这里解释。


### 入队方法

1. 前面巴拉巴拉一通判断，内部的target不能是null,Message对象不能被占用，线程不能退出。。。。

2. 这里的mMessages变量起到了与Message中sPool相同的作用，它是队列的指针，指向消息队列的头，只是这里维护的队列是要被处理的消息队列。但是由于sendMsgDelay方法的存在，入队时不能单纯的链接消息，还需要判断时间戳。


3. `if (p == null || when == 0 || when < p.when) `这个判断if(是第一个消息	||要被处理的时间是0	||要被处理的时间小于当前队列头消息的时间也就是已经到达处理这个消息的时间)，此时将会将消息链接在队列头部`msg.next = p;`，同时指针指向它`mMessages = msg;`,并且此消息是不要被唤醒的`needWake = mBlocked;`,这个操作与Message中的操作十分相似，不多做介绍。


4. else语句块中表明此消息是一个延时消息，此时进行的操作是采用了一个for循环，完成的功能是找到这个消息被处理的时间when，大于这个时间的第一个消息，将它插入到该位置。如果没有找到会一直循环，最后找到preMsg指向的前一个消息和p = mMessage指向的后一消息，进行链接操作，`msg.next = p; prev.next = msg;`就是普通的链接操作。


```java
boolean enqueueMessage(Message msg, long when) {
if (msg.target == null) {
throw new IllegalArgumentException("Message must have a target.");
}
if (msg.isInUse()) {
throw new IllegalStateException(msg + " This message is already in use.");
}
synchronized (this) {
if (mQuitting) {
IllegalStateException e = new IllegalStateException( msg.target + " sending message to a Handler on a dead thread");
 Log.w("MessageQueue", e.getMessage(), e);
msg.recycle();
return false;
}
//核心方法从这里开始
msg.markInUse();
msg.when = when;
Message p = mMessages;
boolean needWake;
if (p == null || when == 0 || when < p.when) {
// New head, wake up the event queue if blocked.
msg.next = p;
mMessages = msg;
needWake = mBlocked;
} else {
// Inserted within the middle of the queue.  Usually we don't have to wake
// up the event queue unless there is a barrier at the head of the queue
// and the message is the earliest asynchronous message in the queue.
needWake = mBlocked && p.target == null && msg.isAsynchronous();
Message prev;
for (;;) {
    prev = p;
    p = p.next;
    if (p == null || when < p.when) {
         break;
    }
	if (needWake && p.isAsynchronous()) {
         needWake = false;
    }
}
msg.next = p; // invariant: p == prev.next
prev.next = msg;
}
// We can assume mPtr != 0 because mQuitting is false.
if (needWake) {
      nativeWake(mPtr);
}
}
return true;
}
```

### 出队方法

1. 之前在Looper.loop()方法中看到有这么一段代码,结合MessageQueue的出队方法，可以发现这是一个循环轮询消息队列的操作。在死循环中调用了`Message msg = 	queue.next(); // might block`方法，当返回msg==null，循环体结束，什么时候会结束，看next()方法中返回null的只有一种情况就是线程结束时，返回null，线程结束，他的Looper自然应该结束。


2. 	死循环实际是发生在MessageQueue中的，我在注释中写了说明。关于MessageQueue的很多本地方法的介绍，大家可以参考[这里](http://blog.csdn.net/android_panda/article/details/8161501)


```java
Message msg = 	queue.next(); // might block
if (msg == null) {
// No message indicates that the message queue is quitting.
return;
}
```

```java
Message next() {
//
// Return here if the message loop has already quit and been disposed.
// This can happen if the application tries to restart a looper after quit
// which is not supported.
final long ptr = mPtr;
if (ptr == 0) {
   return null;
}
int pendingIdleHandlerCount = -1; // -1 only during first iteration
//超时时间
int nextPollTimeoutMillis = 0;
//开始循环取出消息
for (;;) {
    if (nextPollTimeoutMillis != 0) {
        Binder.flushPendingCommands();
}
nativePollOnce(ptr, nextPollTimeoutMillis);
//接下来进行锁定，开始取出消息
synchronized (this) {
	
     // Try to retrieve the next message.  Return if found.
     final long now = SystemClock.uptimeMillis();
     Message prevMsg = null;
     Message msg = mMessages;
     //取到消息是空，然后进行一个循环查找下一个可用消息
     if (msg != null && msg.target == null) {
     // Stalled by a barrier.  Find the next asynchronous message in the queue.
     do {
      prevMsg = msg;
      msg = msg.next;
     } while (msg != null && !msg.isAsynchronous());
 }
if (msg != null) {
   if (now < msg.when) {
// Next message is not ready.  Set a timeout to wake up when it is ready.
//当前时间没有达到执行这个消息的时间。将会进行超时等待
    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
   } else {
   //取出头部的消息返回
   // Got a message.
   mBlocked = false;
   if (prevMsg != null) {
      prevMsg.next = msg.next;
   } else {
      mMessages = msg.next;
   }
   msg.next = null;
   return msg;
}
} else {
  // No more messages.
  nextPollTimeoutMillis = -1;
}
// Process the quit message now that all pending messages have been handled.
//线程不存在时返回null，loop()也会随之结束
if (mQuitting) {
       dispose();
       return null;
}
// If first time idle, then get the number of idlers to run.
// Idle handles only run if the queue is empty or if the first message
// in the queue (possibly a barrier) is due to be handled in the future.
if (pendingIdleHandlerCount < 0&& (mMessages == null || now < mMessages.when)) {
      pendingIdleHandlerCount = mIdleHandlers.size();
}
//死循环发生在这里，具体还没有很明白，需要再去研究
if (pendingIdleHandlerCount <= 0) {
// No idle handlers to run.  Loop and wait some more.
 mBlocked = true;
continue;
}
if (mPendingIdleHandlers == null) {
mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
}
mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
}
// Run the idle handlers.
// We only ever reach this code block during the first iteration.
for (int i = 0; i < pendingIdleHandlerCount; i++) {
    final IdleHandler idler = mPendingIdleHandlers[i];
    mPendingIdleHandlers[i] = null; // release the reference to the handler
boolean keep = false;
try {
    keep = idler.queueIdle();
} catch (Throwable t) {
Log.wtf("MessageQueue", "IdleHandler threw exception", t);
}
if (!keep) {
    synchronized (this) {
      mIdleHandlers.remove(idler);
  }
}
}
// Reset the idle handler count to 0 so we do not run them again.
pendingIdleHandlerCount = 0;
// While calling an idle handler, a new message could have been delivered
// so go back and look again for a pending message without waiting.
nextPollTimeoutMillis = 0;
 }
}
```

## Handler类
### 成员变量
```java
//从这里看得出Handler很好的连接了Looper和MessageQueue
final MessageQueue mQueue;
final Looper mLooper;
//这是一个接口，用来处理消息，下面是具体实现，也是我们通常要实现的方法。
final Callback mCallback;
public interface Callback {
        public boolean handleMessage(Message msg);
}
```

### 构造方法
- 重载了多个构造方法，老规矩，我们只看没有最底层的构造方法和我们最常用的构造方法


```java
//常用空参构造方法
public Handler() {
        this(null, false);
}
//根构造方法
public Handler(Callback callback, boolean async) {
//获得当前线程的Looper，大家还记得Looper.myLooper()方法吧，return sThreadLocal.get();
mLooper = Looper.myLooper();
if (mLooper == null) {
//必须先调用Looper.prepare()
throw new RuntimeException("Can't create handler inside thread that has not called Looper.prepare()");
}
//拿到Looper的MessageQueue,这个MessageQueue是Looper创建的。
mQueue = mLooper.mQueue;
//这个是回调，用来处理消息
mCallback = callback;
mAsynchronous = async;
}
```

## 成员方法

1. obtain()方法，这个方法不做解释，返回的是Message的obtain()方法，详细请看Message类的分析。


2. 发送消息，我们浏览一下所有发送消息的方法。方法很多，大致的思想是，填充消息，发送消息，很多方法都是重载互相调用的，关注最后一个方法，它调用了MessageQueue的入队方法，同时将msg.target设置为this,将这个消息插入到了队列中，也就是被发送的Message持有发送它的Handler的引用。


```java
//以下是各种发送消息的方法，可以浏览一下。
public final boolean post(Runnable r)
{
	return  sendMessageDelayed(getPostMessage(r), 0);
}
public final boolean postAtTime(Runnable r, long uptimeMillis)
{
    return sendMessageAtTime(getPostMessage(r), uptimeMillis);
}
public final boolean postAtTime(Runnable r, Object token, long uptimeMillis)
{
    return sendMessageAtTime(getPostMessage(r, token), uptimeMillis);
}
public final boolean postDelayed(Runnable r, long delayMillis)
{
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}
public final boolean postAtFrontOfQueue(Runnable r)
{
    return sendMessageAtFrontOfQueue(getPostMessage(r));
}
public final boolean sendMessage(Message msg)
{
    return sendMessageDelayed(msg, 0);
}
public final boolean sendEmptyMessage(int what)
{
    return sendEmptyMessageDelayed(what, 0);
}
public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
    Message msg = Message.obtain();
    msg.what = what;
    return sendMessageDelayed(msg, delayMillis);
}
public final boolean sendEmptyMessageAtTime(int what, long uptimeMillis) {
    Message msg = Message.obtain();
    msg.what = what;
    return sendMessageAtTime(msg, uptimeMillis);
}
public final boolean sendMessageDelayed(Message msg, long delayMillis)
{
    if (delayMillis < 0) {delayMillis = 0;}
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
     MessageQueue queue = mQueue;
     if (queue == null) {
        RuntimeException e = new RuntimeException(this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
	}
    return enqueueMessage(queue, msg, uptimeMillis);
}
public final boolean sendMessageAtFrontOfQueue(Message msg) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
     }
     return enqueueMessage(queue, msg, 0);
}
//这两个方法是发送Runnable任务时会调用的方法
private static Message getPostMessage(Runnable r) {
     Message m = Message.obtain();
     m.callback = r;
     return m;
}
private static Message getPostMessage(Runnable r, Object token) {
     Message m = Message.obtain();
     m.obj = token;
     m.callback = r;
     return m;
}
//消息入队，同时Message将会持有发送它的Handler的引用
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
     msg.target = this;
     if (mAsynchronous) {msg.setAsynchronous(true);}
     return queue.enqueueMessage(msg, uptimeMillis);
}
```
### 事件分发与处理

1. 还记得Looper中遗留的问题吗？`msg.target.dispatchMessage(msg);`结合发送消息的方法，可以得出，msg.target就是发送它的那个Handler,处理消息时调用dispatchMessage方法，也就是下面的方法，所以，Message携带它的发送者，谁发送的消息誰来处理它。
2. 分析一下下面的逻辑。`msg.callback`是消息中包含的任务，如果是一个任务的消息，那么不需要外部处理，直接调用该Runnable任务的run方法，所以还是在当前线程执行，并没有开启新的线程，不要看到Runnable就想到线程，这也是我以前的一个误区
3. 不是一个Runnable任务，`mCallback `是一个接口，在介绍成员变量是提到过，他可以通过构造方法在外部实现，当然不是必须实现的，如果实现了这个接口，那么调用这个接口的`mCallback.handleMessage(msg)`方法处理消息。
4. 如果没有实现这个接口，则调用`handleMessage(msg);`方法，这个方法是空的，需要你在子类中重载，如果你没有重载他不会执行任何操作。这算提供了处理消息的两种方式。

```java
public void dispatchMessage(Message msg) {
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
private static void handleCallback(Message message) {
        message.callback.run();
}
public void handleMessage(Message msg) {
}
```

##总结
- 线程中Handler消息机制的使用,Looper类中给出了很好的示例代码。注意的是在非UI线程需要我们显式的调用` Looper.prepare(); Looper.loop();`方法来完成消息的轮询。


```java
class LooperThread extends Thread {
      public Handler mHandler;  
      public void run() {
           Looper.prepare();
           mHandler = new Handler() {
                public void handleMessage(Message msg) {
                    // process incoming messages here
                }
            };
            Looper.loop();
      }
}
```
