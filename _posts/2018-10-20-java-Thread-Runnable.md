---
layout:     post
title:      "java多线程"
subtitle:   " \"java多线程二(Thread和Runnable创建多线程的区别)\""
date:       2018-10-20 12:00:00
author:     "LingFeng"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - 多线程
---

> “Yeah It's on. ”


## 前言

上一章我们讲到创建线程常用的两种方式，一种是继承Thread类，一种是实现Runnable接口。那么我们这一样主要看继承Thread和实现Runnable的区别
 
 
<p id = "build"></p>
---

## 正文
 
首先我们可以看到Thread也是实现了Runnable的接口

---


```java 
public
class Thread implements Runnable {
    /* Make sure registerNatives is the first thing <clinit> does. */
    private static native void registerNatives();
    static {
        registerNatives();
    }
 
 
    private char        name[];
    private int         priority;
    private Thread      threadQ;
    private long        eetop;
 
 
    /* Whether or not to single_step this thread. */
    private boolean     single_step;
 
 
    /* Whether or not the thread is a daemon thread. */
    private boolean     daemon = false;
 
 
    /* JVM state */
    private boolean     stillborn = false;
 
 
    /* What will be run. */
    private Runnable target;
 
 
    /* The group of this thread */
    private ThreadGroup group;
 
 
    /* The context ClassLoader for this thread */
    private ClassLoader contextClassLoader;
 
 
    /* The inherited AccessControlContext of this thread */
    private AccessControlContext inheritedAccessControlContext;
 
 
    /* For autonumbering anonymous threads. */
    private static int threadInitNumber;
    private static synchronized int nextThreadNum() {
        return threadInitNumber++;
    }
 
 
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
 
 
    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
 
 
    /*
     * The requested stack size for this thread, or 0 if the creator did
     * not specify a stack size.  It is up to the VM to do whatever it
     * likes with this number; some VMs will ignore it.
     */
    private long stackSize;
 
 
    /*
     * JVM-private state that persists after native thread termination.
     */
    private long nativeParkEventPointer;
 
 
    /*
     * Thread ID
     */
    private long tid;
 
 
    /* For generating thread ID */
    private static long threadSeqNumber;
 
 
    /* Java thread status for tools,
     * initialized to indicate thread 'not yet started'
     */
 
 
    private volatile int threadStatus = 0;
 
 
 
 
    private static synchronized long nextThreadID() {
        return ++threadSeqNumber;
    }
 
 
    /**
     * The argument supplied to the current call to
     * java.util.concurrent.locks.LockSupport.park.
     * Set by (private) java.util.concurrent.locks.LockSupport.setBlocker
     * Accessed using java.util.concurrent.locks.LockSupport.getBlocker
     */
    volatile Object parkBlocker;
 
 
    /* The object in which this thread is blocked in an interruptible I/O
     * operation, if any.  The blocker's interrupt method should be invoked
     * after setting this thread's interrupt status.
     */
    private volatile Interruptible blocker;
    private final Object blockerLock = new Object();
 
 
    /* Set the blocker field; invoked via sun.misc.SharedSecrets from java.nio code
     */
    void blockedOn(Interruptible b) {
        synchronized (blockerLock) {
            blocker = b;
        }
    }
 
 
    /**
     * The minimum priority that a thread can have.
     */
    public final static int MIN_PRIORITY = 1;
 
 
   /**
     * The default priority that is assigned to a thread.
     */
    public final static int NORM_PRIORITY = 5;
 
 
    /**
     * The maximum priority that a thread can have.
     */
    public final static int MAX_PRIORITY = 10;
 
 
    /**
     * Returns a reference to the currently executing thread object.
     *
     * @return  the currently executing thread.
     */
    public static native Thread currentThread();
 
 
    /**
     * A hint to the scheduler that the current thread is willing to yield
     * its current use of a processor. The scheduler is free to ignore this
     * hint.
     *
     * <p> Yield is a heuristic attempt to improve relative progression
     * between threads that would otherwise over-utilise a CPU. Its use
     * should be combined with detailed profiling and benchmarking to
     * ensure that it actually has the desired effect.
     *
     * <p> It is rarely appropriate to use this method. It may be useful
     * for debugging or testing purposes, where it may help to reproduce
     * bugs due to race conditions. It may also be useful when designing
     * concurrency control constructs such as the ones in the
     * {@link java.util.concurrent.locks} package.
     */
    public static native void yield();
 
 
    /**
     * Causes the currently executing thread to sleep (temporarily cease
     * execution) for the specified number of milliseconds, subject to
     * the precision and accuracy of system timers and schedulers. The thread
     * does not lose ownership of any monitors.
     *
     * @param  millis
     *         the length of time to sleep in milliseconds
     *
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public static native void sleep(long millis) throws InterruptedException;
 
```
 
---


所以其实我们继承了Thread，然后重构了Thread的run方法。其实就是实现了Runnable的run方法。
 
那么为什么Thread可以讲Runnable的实例对象作为参数呢？
我们可以到Thread类中看到Thread提供了一个Runnable为参数的构造方法

---
 
```java
/**
* Allocates a new {@code Thread} object. This constructor has the same
* effect as {@linkplain #Thread(ThreadGroup,Runnable,String) Thread}
* {@code (null, target, gname)}, where {@code gname} is a newly generated
* name. Automatically generated names are of the form
* {@code "Thread-"+}<i>n</i>, where <i>n</i> is an integer.
*
* @param  target
*         the object whose {@code run} method is invoked when this thread
*         is started. If {@code null}, this classes {@code run} method does
*         nothing.
*/
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}
 
```

---

其实这个构造方法是调用的init方法。那么我们继续看init方法

---
```java

/**
* Initializes a Thread.
*
* @param g the Thread group
* @param target the object whose run() method gets called
* @param name the name of the new Thread
* @param stackSize the desired stack size for the new thread, or
*        zero to indicate that this parameter is to be ignored.
*/
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize) {
    if (name == null) {
        throw new NullPointerException("name cannot be null");
    }
 
 
    Thread parent = currentThread();
    SecurityManager security = System.getSecurityManager();
    if (g == null) {
        /* Determine if it's an applet or not */
 
 
        /* If there is a security manager, ask the security manager
           what to do. */
        if (security != null) {
            g = security.getThreadGroup();
        }
 
 
        /* If the security doesn't have a strong opinion of the matter
           use the parent thread group. */
        if (g == null) {
            g = parent.getThreadGroup();
        }
    }
 
 
    /* checkAccess regardless of whether or not threadgroup is
       explicitly passed in. */
    g.checkAccess();
 
 
    /*
     * Do we have the required permissions?
     */
    if (security != null) {
        if (isCCLOverridden(getClass())) {
            security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
        }
    }
 
 
    g.addUnstarted();
 
 
    this.group = g;
    this.daemon = parent.isDaemon();
    this.priority = parent.getPriority();
    this.name = name.toCharArray();
    if (security == null || isCCLOverridden(parent.getClass()))
        this.contextClassLoader = parent.getContextClassLoader();
    else
        this.contextClassLoader = parent.contextClassLoader;
    this.inheritedAccessControlContext = AccessController.getContext();
    this.target = target;
    setPriority(priority);
    if (parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    /* Stash the specified stack size in case the VM cares */
    this.stackSize = stackSize;
 
 
    /* Set thread ID */
    tid = nextThreadID();
}

```
---

首先是判断name属性是否为空。我们从上面构造方法来看 "Thread-" + nextThreadNum() 这个其实就是name属性的值

---

```java

/* For autonumbering anonymous threads. */
private static int threadInitNumber;
private static synchronized int nextThreadNum() {
    return threadInitNumber++;
}

```
---

现在我们可以看出这个值其实是一个在同步的值。
 
下面的 Thread parent = currentThread(); 我们看一下这个currentThread();是什么

---

```java

/**
* Returns a reference to the currently executing thread object.
*
* @return  the currently executing thread.
*/
public static native Thread currentThread();

```

---

其实是一个静态的object。这个native其实就是jni,不懂的可以参考 https://blog.csdn.net/jiakw_1981/article/details/3073613 该博客，该博客讲的很详细。这里不做解释。
 
再下面的代码其实就是一些判断。那么我们从这个init方法中可以知道什么呢？首先java的多线程其实是有2个调用栈(Call stack),一个用于跟踪java代码。一个用于跟踪java代码
对本地代码的调用关系。本地代码通常是c语言。
 
看到现在我们发现其实Runnable接口的实现类交给Thread运行和直接Thread继承重写run方法。其实差别不是很大。那么我们为什么习惯用Runnable不用Thread呢？

---

*1.克服单继承的缺点。
*2.可以实现数据的共享。

---

```java

 package org.thread.demo;  
 class MyThread extends Thread{  
 private String name;  
 public MyThread(String name) {  
 super();  
 this.name = name;  
 }  
 public void run(){  
 for(int i=0;i<10;i++){  
 System.out.println("线程开始："+this.name+",i="+i);  
 }  
 }  
 }  
 package org.thread.demo;  
 public class ThreadDemo01 {  
 public static void main(String[] args) {  
 MyThread mt1=new MyThread("线程a");  
 MyThread mt2=new MyThread("线程b");  
 mt1.run();  
 mt2.run();  
 }  
 }

```

```java

 package org.thread.demo;  
 public class ThreadDemo01 {  
 public static void main(String[] args) {  
 MyThread mt1=new MyThread("线程a");  
 MyThread mt2=new MyThread("线程b");  
 mt1.start();  
 mt2.start();  
 }  
 };

```

```java

 package org.runnable.demo;  
 class MyThread implements Runnable{  
 private String name;  
 public MyThread(String name) {  
 this.name = name;  
 }
 public void run(){  
 for(int i=0;i<100;i++){  
 System.out.println("线程开始："+this.name+",i="+i);  
 }  
 }  
 };

```

```java

 package org.runnable.demo;  
 import org.runnable.demo.MyThread;  
 public class ThreadDemo01 {  
 public static void main(String[] args) {  
 MyThread mt1=new MyThread("线程a");  
 MyThread mt2=new MyThread("线程b");  
 new Thread(mt1).start();  
 new Thread(mt2).start();  
 }  
 } 
 }  
 }

```
 
其实资源只有10个，3个Thread。各自有10个。所以是30个。但是通过Runnable的实现类却是共享这10个资源。
 
所以Thread和Runnable的区别就这么多

—— LingFeng 后记于 2019.03.03