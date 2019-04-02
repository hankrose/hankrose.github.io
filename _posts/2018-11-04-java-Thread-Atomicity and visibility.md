---
layout:     post
title:      "java多线程"
subtitle:   " \"java多线程四(怎么确保多线程操作数据的原子性和可见性)\""
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

上面我们讲到了多线程的不能确保原子性和可见性。那么我们下面试试用一下用锁来试试，首先介绍到锁，我们就要知道锁是什么，锁干了些什么。目前java多线程用的锁，其中最多的就是synchronized
 
 
<p id = "build"></p>
---

## 正文
 
我们还是用代码来看一个经典的案例,就是卖票案例

---


```java 
package ThreadTest;


/**
* 用锁来确保多线程操作的原子性
* @author lingfengz
*/
public class SaleThread {
    public static void main(String[] args){
        SalesTest test = new SalesTest();
        Thread thread1 = new Thread(test,"售票窗口1");
        Thread thread2 = new Thread(test,"售票窗口2");
        Thread thread3 = new Thread(test,"售票窗口3");
        Thread thread4 = new Thread(test,"售票窗口4");
        Thread thread5 = new Thread(test,"售票窗口5");
        Thread thread6 = new Thread(test,"售票窗口6");
        Thread thread7 = new Thread(test,"售票窗口7");
        Thread thread8 = new Thread(test,"售票窗口8");
        Thread thread9 = new Thread(test,"售票窗口9");
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
        thread5.start();
        thread6.start();
        thread7.start();
        thread8.start();
        thread9.start();
    }
}
class SalesTest implements Runnable{
    private static int salesCount = 1000;//100张票


    Object lock = new Object();
    /**
     * 实现run方法
     */
    @Override
    public void run() {
        while(true){
            synchronized (lock){
                if(salesCount>0){
                    try{
                        //先沉睡,让线程体现不同步
                        Thread.sleep(20L);
                    }catch (InterruptedException e){
                        e.getStackTrace();
                    }
                        System.out.println(Thread.currentThread().getName()+"卖出第"+(salesCount--)+"号票");
                }else{
                    break;
                }
            }
        }
    }
}

运行结果呢是
售票窗口1卖出第1000号票
............
售票窗口1卖出第895号票
售票窗口1卖出第894号票
售票窗口7卖出第893号票
售票窗口7卖出第892号票
............
售票窗口7卖出第1号票
 
```
 
---


这个比较常用的一种，将涉及到共享变量的操作放进被synchronized修饰的代码块中。synchronized拿的是对象锁。这个对象只有这个代码块都执行结束了，才会被放开，被jvm调度池重新分配给其他线程。

还有一种用法就是用synchronized修饰一个方法，这种方式和上面的方式有何不同呢？

---
 
```java
package ThreadTest;


/**
* 用锁来确保多线程操作的原子性
* @author lingfengz
*/
public class TestThread {
    public static void main(String[] args){
       final Threadtest x = new Threadtest();
        Thread t1 = new Thread(new Runnable(){
            @Override
            public void run() {
                x.test1();
            }
        });


        Thread t2 = new Thread(new Runnable(){
            @Override
            public void run() {
                x.test2();
            }
        });
        t1.start();
        t2.start();
    }
}
class Threadtest{
    public void test1(){
        synchronized (this){
            for(int i=0;i<5;i++){
                System.out.println("test1:  "+i);
            }
        }
    }


    public synchronized void test2(){
        for(int i=0;i<5;i++){
            System.out.println("test2:  "+i);
        }
    }
}


运行结果：
test1:  0
test1:  1
test1:  2
test1:  3
test1:  4
test2:  0
test2:  1
test2:  2
test2:  3
test2:  4
 
```

---

我们可以看到用synchronized修饰的方法在第一个方法执行之后开始执行。那么我们是否可以认为synchronized修饰代码块和synchronized修饰方法其实获得的都是this锁呢？

我们再来看
---
```java

package ThreadTest;


/**
* 用锁来确保多线程操作的原子性
* @author lingfengz
*/
public class TestThread {
    public static void main(String[] args){
       final Threadtest x = new Threadtest();
        Thread t1 = new Thread(new Runnable(){
            @Override
            public void run() {
                x.test1();
            }
        });


        Thread t2 = new Thread(new Runnable(){
            @Override
            public void run() {
                x.test2();
            }
        });
        t1.start();
        t2.start();
    }
}
class Threadtest{
    Object lock = new Object();
    public void test1(){
        synchronized (lock){
            for(int i=0;i<5;i++){
                System.out.println("test1:  "+i);
            }
        }
    }


    public synchronized void test2(){
        for(int i=0;i<5;i++){
            System.out.println("test2:  "+i);
        }
    }
}


运行结果：
test2:  0
test1:  0
test1:  1
test1:  2
test1:  3
test1:  4
test2:  1
test2:  2
test2:  3
test2:  4

```
---

我们给test1获取lock这个对象锁后，两个线程方法又不同步。说明synchronized修饰方法的时候拿到的其实是this锁。


那么我们再来看synchronized修饰静态方法

---

```java

package ThreadTest;


/**
* 用锁来确保多线程操作的原子性
* @author lingfengz
*/
public class TestThread {
    public static void main(String[] args){
       final Threadtest x = new Threadtest();
        Thread t1 = new Thread(new Runnable(){
            @Override
            public void run() {
                x.test1();
            }
        });


        Thread t2 = new Thread(new Runnable(){
            @Override
            public void run() {
                x.test2();
            }
        });
        t1.start();
        t2.start();
    }
}
class Threadtest{


    public void test1(){
        synchronized (this){
            for(int i=0;i<5;i++){
                System.out.println("test1:  "+i);
            }
        }
    }


    public static synchronized void test2(){
        for(int i=0;i<5;i++){
            System.out.println("test2:  "+i);
        }
    }
}

运行结果：
test1:  0
test1:  1
test1:  2
test1:  3
test1:  4
test2:  0
test2:  1
test2:  2
test2:  3
test2:  4

```
---

从上面可以看出来修饰静态方法的得到锁并不是this锁，也就是不是这个当前对象的锁。那么是什么呢？
其实它是class锁。因为一个java对象可以有多个实例。但是只有一个class。所以这边这个静态方法的锁是class锁，我们来验证一下

---

```java
package ThreadTest;


/**
* 用锁来确保多线程操作的原子性
* @author lingfengz
*/
public class TestThread {
    public static void main(String[] args){
       final Threadtest x = new Threadtest();
        Thread t1 = new Thread(new Runnable(){
            @Override
            public void run() {
                x.test1();
            }
        });


        Thread t2 = new Thread(new Runnable(){
            @Override
            public void run() {
                x.test2();
            }
        });
        t1.start();
        t2.start();
    }
}
class Threadtest{


    public void test1(){
        synchronized (Threadtest.class){
            for(int i=0;i<5;i++){
                System.out.println("test1:  "+i);
            }
        }
    }


    public static synchronized void test2(){
        for(int i=0;i<5;i++){
            System.out.println("test2:  "+i);
        }
    }
}
运行结果：

test1:  0
test1:  1
test1:  2
test1:  3
test1:  4
test2:  0
test2:  1
test2:  2
test2:  3
test2:  4

```

---

结论是正确。


所以我们可以得出 synchronized修饰非静态方法的时候其实获得的是对象实例锁。synchronized修饰代码块的时候，既可以修饰当前对象本身，也可以修饰其他对象。也可以修饰当前class。而synchronized修饰静态方法则是修饰当前class

—— LingFeng 后记于 2019.11.04