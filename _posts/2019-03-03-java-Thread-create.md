---
layout:     post
title:      "Hello 2015"
subtitle:   " \"java多线程一（怎么创建多线程）\""
date:       2019-03-03 12:00:00
author:     "LingFeng"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - 多线程
---

> “Yeah It's on. ”


## 前言

今天面试被人面多线程的问题，只能答出简单的。更深层次的就答不出来了，比如闭锁和棚栏的区别。所以在这里我将从新开始回顾java中的多线程。
 
 
<p id = "build"></p>
---

## 正文
 
* 1.什么是多线程？
* 2.为什么要用多线程 ？
* 3.多线程可以做什么？
* 4.多线程怎么用?
 
1 什么是多线程?
    说到线程，就要说到进程。进程的话，我下面摘录了百度百科的答案
 
    进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础。在早期面向进程设计的计算机结构中，进程是程序的基本执行实体；在当代面向线程设计的计算机结构中，进程是线程的容器。程序是指令、数据及其组织形式的描述，进程是程序的实体。
 
从上面的介绍可以看出进程是线程的容器，所以一个进程可以有多个线程。也就是一个进程有多个执行路径。
 
2 为什么要用多线程
    在我的理解中，多线程这个东西是因为要同时处理或者是因为处理大批量的操作和数据，所采用的方式之一。比如你在聊qq的同时，还想qq语音。那么qq只是一个进程，你既想聊qq又想聊语音。那么我们就要用到多线程。让某些线程负责语音，某些线程负责打字聊天。但是多线程也是有缺点的。
缺点：1.线程的等待，沉睡，挂起，唤醒，以及多线程对某些共享数据的使用会造成一些不可追溯的影响
         2.影响系统的性能 因为系统开辟一个线程要分配内存资源，线程过多会导致内存的不足 
         3.因为系统会在线程中来回切换。所以系统的性能会受到一些影响
 
3.多线程可以做什么
    多线程的场景很多，比如一些客户访问量巨大的一些网站。一些大文件的读取。一些大批量操作的更新。以及一些需要等待时间长的操作，多线程可以将这些场景下的客户体验提升很多。
 
4.多线程怎么用
    在java中，主要有两种方式可以创建多线程，一种是继承Thread类，一种是实现Runnable接口。还有一种不常见的是Callable接口(具有返回值)
    第一种
```java 
package ThreadTest;
/**
* Thread类实现多线程
* @author lingfengz
*
*/
public class ThreadTest {
    public static void main(String[] agrs) {
        //创建10个线程
        for(int i = 0;i<10;i++){
                new Thread1("thread"+i).start();
        }
    }
}
 
 
class Thread1 extends Thread {
    private  int count = 1;
    private String threadName;
 
    public Thread1(String threadName){
        this.threadName = threadName;
    }
    /**
     * 重写run方法
     */
    @Override
    public void run() {
        // TODO Auto-generated method stub
        System.out.println(threadName +": "+count);
        count++;
    }
}
 

 
运行结果是：
thread0: 1
thread2: 1
thread1: 1
thread3: 1
thread4: 1
thread5: 1
thread6: 1
thread9: 1
thread8: 1
thread7: 1
 
```
 
 
 从上面的运行结果可以看出，线程的结束并不是依赖上一个for循环的结束。
 
还有第二种方法是实现runnable的接口
 
 
```java
package ThreadTest;
 
 
/**
* 使用runnable接口来创建线程
* Created by 10360 on 2019/3/12.
*/
public class RunnableTest {
    public static void main(String args[]){
        /**
         * 创建10个runnable接口
         */
        for(int i=0 ; i<10 ;i++){
            Thread thread = new Thread(new Runnable1("runnable"+i));
            thread.start();
        }
    }
 
 
}
 
 
class Runnable1 implements Runnable{
    private String runnableName;
    private int count=1;
 
 
    public Runnable1(String runnableName) {
        this.runnableName = runnableName;
    }
 
 
    /**
     * 实现run方法
     */
    @Override
    public void run() {
        System.out.println(runnableName+": "+count);
    }
}
 
运行结果是：
runnable1: 1
runnable5: 1
runnable8: 1
runnable4: 1
runnable2: 1
runnable7: 1
runnable3: 1
runnable0: 1
runnable9: 1
runnable6: 1
 
```
 
还有内部匿名类和callable实现多线程，暂时这里不解释。我会在后面单独讲callable和内部匿名类实现多线程。

—— LingFeng 后记于 2019.03.03