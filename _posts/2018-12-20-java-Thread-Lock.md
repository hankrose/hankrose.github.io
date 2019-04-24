---
layout:     post
title:      "java多线程"
subtitle:   " \"java多线程十(Thread Lock 的解析)\""
date:       2018-12-20 12:00:00
author:     "LingFeng"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - 多线程
---

> “Yeah It's on. ”


## 前言

之前我们讲过了synchronized这个锁。这个锁是锁代码块，或者方法，或者是class的。所以并不能在其他的方法上释放锁。
 
 
<p id = "build"></p>
---

## 正文
 
    那么我们为什么要分开两种锁呢？内部锁和外部锁的区别是什么？我们来探究一下。

    1.内部锁锁住的是一块代码，一个方法，或者一个充当锁的类。那么它释放锁的时候，释放锁的线程会不会再次得到这把锁呢？目前来看是可能，并且有很大概率的。从我们之前的代码也可以看出来，那么我们可以定义一下，这个锁它是不公平的。如果持有锁的线程一直不释放锁，那么其他等待锁的线程只能在等待区等待。这样的设计是不公平的。但是内部所的释放和获取会不会放生锁的泄露呢？然而并不会。所以我们定义一下内部锁的优点

    synchronized 优点: 不会产生锁的泄露。 缺点就是它是一个非公平锁。而且其他线程必须在等待区等待，其实等待的时候其他的锁的状态是unnable的。

那么lock 实现类这个显式锁是个什么东西呢？我们用代码来看一下
---
	
```java 
package ThreadTest;


import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;


public class LockTest
{
    public static void main(String[] args) {
        LockThread test =  new LockThread();
        Thread thread1 = new Thread(test,"thread1");
        Thread thread2 = new Thread(test,"thread2");
        Thread thread3 = new Thread(test,"thread3");
        Thread thread4 = new Thread(test,"thread4");
        Thread thread5 = new Thread(test,"thread5");
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
        thread5.start();
    }
}
class LockThread implements Runnable{
    private int i=0;
    private Lock lock = new ReentrantLock();

    @Override
    public void run() {
        lock.lock();
        try{
            i++;
            System.out.println(Thread.currentThread().getName()+" i :"+i);
        }catch (Exception e){
            e.getStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
 
```
 
---

从上面我们可以看出显式锁的调用方式，首先是在线程内部创建一个实现了lock接口的 ReentrantLock。这也是目前默认的方式。在run里面我们调用了lock方法，这个就是获取锁的方法。在finally里面我们把锁给释放了。为什么要写在finally里面？ 因为这个锁如果我们不释放可能会导致泄漏，所以放在finally里面无论是否执行成功或者异常最后都会把锁给释放了

那么这个显示锁和内部锁(synchronized)有什么区别呢？
1.显示锁复合oop思想，因为内部锁只能锁一个代码块或者一个方法。它不能跨方法释放锁，但是显示锁可以跨方法释放锁
2.显示锁可能会锁泄漏，但是synchronized不会导致锁泄漏
3.内部锁(synchronized)是非公平锁，而显示锁可以是非公平的也可以是公平的
4.lock的trylock方法可以避免锁被某个线程长期持有，导致其他线程的工作都停止等待锁的分配

那么我们首先来看一些trylock方法。

---
 
```java
package ThreadTest;


import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;


public class LockTest
{
    public static void main(String[] args) {
        LockThread test =  new LockThread();
        Thread thread1 = new Thread(test,"thread1");
        Thread thread2 = new Thread(test,"thread2");
        Thread thread3 = new Thread(test,"thread3");
        Thread thread4 = new Thread(test,"thread4");
        Thread thread5 = new Thread(test,"thread5");
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
        thread5.start();
    }
}
class LockThread implements Runnable{
    private int i=0;
    private Lock lock = new ReentrantLock();


    @Override
    public void run() {
        if(lock.tryLock()) {
            try {
                i++;
                System.out.println(Thread.currentThread().getName() + " i :" + i);
            } catch (Exception e) {
                e.getStackTrace();
            } finally {
                lock.unlock();
            }
        }else{
            System.out.println(Thread.currentThread().getName() +"没拿到锁，出去吃个饭再来看看");
        }
    }
}


运行结果：
thread1 i :1
thread2没拿到锁，出去吃个饭再来看看
thread3没拿到锁，出去吃个饭再来看看
thread4 i :2
thread5 i :3
 
```

---
   线程2和线程3没有拿到锁，但是他们走了下面的方法。所以trylock方法的简单作用我们可以看出来。首先获取的锁空闲状态。返回true说明拿到了锁。返回false说明锁正在被其他线程所持有。我们可以做其他事情
---

总结
    显示锁复合oop思想，可以跨方法释放锁，显示锁可能会锁泄漏，显示锁可以是非公平的也可以是公平的，lock的trylock方法可以避免锁被某个线程长期持有，导致其他线程的工作都停止等待锁的分配

—— LingFeng 后记于 2018.12.20