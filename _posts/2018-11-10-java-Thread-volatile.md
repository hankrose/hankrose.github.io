---
layout:     post
title:      "java多线程"
subtitle:   " \"java多线程五(volatile关键字解析和用处)\""
date:       2018-11-10 12:00:00
author:     "LingFeng"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - 多线程
---

> “Yeah It's on. ”


## 前言

首先我们要知道volatile这个关键字是什么意思。我摘录了百度百科的回答：
![img](/img/in-post/java-Thread/volatile.jpg) 
<p id = "build"></p>
---

## 正文
 
那么可以从字面上的意思我们了解到这个东西代表的含义是轻快的，不稳定的。我们下面就从字面含义上来解析这个关键字在java中的作用

首先我们知道要在多线程中保证操作的原子性和可见性 我们常用的是synchronized 这个关键字。它起到的作用在上一篇文章中我也做了简短的分析。
它的作用我们也可以看到确保同一个时间段内，只有一个线程能对synchronized修饰的代码块或者方法中的java代码进行操作。这样确保了原子性也确保了可见性
但是对系统的开销非常大。所以这个不是轻便的解决方案。我们首先看看多线程和内存之间数据是怎么交互的。
<ba/>
![img](/img/in-post/java-Thread/thread-merry.png) 
<br/>

   首先创建线程后，jvm会为线程创建一个内存副本。比如主存中有一个int a,它的值是10.那么有一个操作是对a进行加1的操作。那么线程1操作的是它内存副本中的a的值。
线程2也是操作内存副本中的a的值。所以他们不同步，但是如果我们用了synchronized这个方法，那么线程1和线程2只有一个线程可以在一个时间点上对a进行修改，
修改完之后，线程1或者线程2将内存副本中a的数据与主存同步后。jvm将主存中的新的值更新到其他线程的内存副本中，就是为什么synchronized可以实现可见性和原子性操作
但是问题来了，如果用synchronized的锁住的数据其实并不是依赖于内存副本中的值呢？那么还是用synchronized的话，是不是对性能影响也是很大？

所以java引入了一个volatile关键字来处理这些只要保证可见性，但是不保证原子性操作的关键字。

那么volatile是怎么保证可见性呢？

    我们先简单的描述一下，在线程1中，对呗volatile修饰的变量在进行了操作之后，volatile直接将变量的值强制更新的主内存中，并且发送给其他线程，volatile修饰的变量的值发生了变化。
然后其他线程会把原来内存副本中的值给丢弃。再去主存中拿最新的值。总体来说volatile做的事情就是这些。
那么我们有疑问了？什么时候适合用volatile来修饰变量呢？

    首先我们来看一下，首先这个不适用与有约束的值。如果一个变量被volatile 修饰的话，在线程有判断这个变量是否大于0，在b线程判断volatile的时候或许这个变量是1，但是a线程修改后这个变量变成了0，但是b的校验判断已经通过。
那么这个就是脏数据了。所以volatile不适合用于写操作。那么volatile适合什么呢？
    读操作大于写操作的。使用volatile性能上要好一点。还有呢?
    还有就是作为一个标识来做处理。比如我们电脑关机前是不是要点击这个windows的菜单键，然后选择关机。那么我们用volatile修饰一个布尔值。这个布尔值只用于判断是否需要关机。
	
	
---


```java 
package ThreadTest;
/**
* volatile关键字使用
* @author lingfengz
*
*/
public class VolatileTest {
     public static void main(String[] args) {
          VolatileThread test  = new VolatileThread();
          
          Thread thread1 = new Thread(test,"线程1");
          Thread thread2 = new Thread(test,"线程2");
          Thread thread3 = new Thread(test,"线程3");
          Thread thread4 = new Thread(test,"线程4");
          Thread thread5 = new Thread(test,"线程5");
          Thread thread6 = new Thread(test,"线程6");
          Thread thread7 = new Thread(test,"线程7");
          
          thread1.start();
          thread2.start();
          thread3.start();
          thread4.start();
          thread5.start();
          thread6.start();
          thread7.start();
          
          test.downComputer();
          
     }
}
class VolatileThread implements Runnable{
     private volatile boolean downComputerstauts = false;
     /**
      * 实现run方法
      */
     @Override
     public void run() {
          // TODO Auto-generated method stub
              if(downComputerstauts){
                   downComputerstauts = false;
                   try {
                        Thread.sleep(20);
                   } catch (InterruptedException e) {
                        // TODO Auto-generated catch  block
                        e.printStackTrace();
                   }
                   System.out.println(Thread.currentThread().getName()+"正在关机!!");
              }
     }
     
     public void downComputer(){
          this.downComputerstauts = true;
     }
     
}
 
```
 
---



其实可以看到,当第一个线程获得这个downComputerstauts的状态发生变化后。进入方法。第一时间再次修改了该变量的值。通知了其他线程这个值的变化，其他线程就获得了false的结果。

—— LingFeng 后记于 2018.11.10