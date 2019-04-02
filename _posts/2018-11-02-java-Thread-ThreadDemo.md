---
layout:     post
title:      "java多线程"
subtitle:   " \"java多线程三(典型的多线程案例)\""
date:       2018-11-02 12:00:00
author:     "LingFeng"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - 多线程
---

> “Yeah It's on. ”


## 前言

上面两篇讲了怎么创建线程和两个种常见的创建线程的方式。那么我们来试一试怎么日常生活中的多线程的场景
 
 
<p id = "build"></p>
---

## 正文
 
1.经典的卖票应用

---


```java 
package ThreadTest;
/**
* 两个窗口同时卖票
* @author lingfengz
*
*/
public class SalesThread {
     public static void main(String[] args) {
          Thread thread1 = new Thread(new salesTest("卖票窗口1"));
          Thread thread2 = new Thread(new salesTest("卖票窗口2"));
          thread1.start();
          thread2.start();
     }
}
class salesTest implements Runnable{
     private String salesName;
     //票据总量为100
     private int salescount = 100;
     
     public salesTest(String salesName){
          this.salesName = salesName;
     }
     
     /**
      * 无参构造方法，我建议每个要实例化的model都写一个无参构造方法
      */
     public salesTest(){
          
     }
     /**
      * 实现run方法
      */
     @Override
     public void run() {
          // TODO Auto-generated method stub
          try{
               //当还有余票的时候,我们就开始销售
               while(salescount>0){
                    sales();
               }
          }catch(Exception e){
               e.getStackTrace();
          }
     }
     
     /**
      * 记录销售的方法
      */
     public void sales(){
          System.out.println(salesName+"---卖出第"+(100-salescount+1)+"票");
          salescount--;
     }
}
 
```
 
---


表面上我们看这段代码好像并没有什么问题,那么我们下面获得的结果却是？
 
---
 
```java
卖票窗口1---卖出第1票
卖票窗口2---卖出第1票
卖票窗口2---卖出第2票
卖票窗口2---卖出第3票
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

怎么会窗口1和2同时卖出第一张票呢？这是为什么呢？

原因就是每次多线程启动的时候其实修改的是jvm为线程分配的内存副本中的数据，而不是真实的内存中的数据。我们可以把这个操作想象成我们平时用的git或者svn。我们从这些版本控制器中获取代码，但是我们修改的时候修改的并不是版本控制器中的代码，而是我们的本地副本。如果我们想要更新版本控制器中的代码，那么我们就要进行提交操作。
所以我们上面会出现这样的结果。因为没有保证数据的原子性。原子性是什么呢？原子性就是一件事情不能被干扰。比如 ++这个操作，其实分三步。首先创建一个数据副本，然后对数据副本做加操作，最后再把操作后的值给原来的数据。如果我们创建了数据副本后，该数据被其他的线程修改了，那么我们数据副本中的值其实是过期的。这个就是破坏了数据的原子性。

那么上面这个我们又可以牵扯到数据副本和内存的映射。因为我们多线程操作数据其实都是操作的内存副本中的数据。所以如果线程修改了某个共享的变量，那么其他线程可以从主存中获取到实际的值。那么这就是数据的可见性。

我们这一章就讲这些，下一章我们来看看有什么方式来解决这个问题


—— LingFeng 后记于 2018.11.02