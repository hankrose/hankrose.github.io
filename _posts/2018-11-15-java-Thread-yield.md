---
layout:     post
title:      "java多线程"
subtitle:   " \"java多线程六(Thread类中yield方法)\""
date:       2018-11-15 12:00:00
author:     "LingFeng"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - 多线程
---

> “Yeah It's on. ”


## 前言

这边我们来说一下java Thread类中的方法。首先我们来说yield方法。我们再说yield之前我们先看一下java 多线程的生命周期。
<p id = "build"></p>
---

## 正文
 
一般分为四种状态 new  runnable runing  dead 四种状态。下面我们来看一下图
<ba/>
![img](/img/in-post/java-Thread/threadlife.png) 
<br/>
下面我们来试一段代码
---


```java 
package ThreadTest;
public class YieldTest {
     public static void main(String[] args) {
          
          Thread test1 = new Yield("test1");
          Thread test2 = new Yield("test2");
          Thread test3 = new Yield("test3");
          
          test1.start();
          test2.start();
          test3.start();
     }
}
//继承Thread
class Yield extends Thread{
     
     public Yield(){
          
     }
     
     public Yield(String name){
          super(name);
     }
     
     @Override
     public void run() {
          // TODO Auto-generated method stub
          for(int i=0;i<100;i++){
              System.out.println(Thread.currentThread().getName()+":  "+i);
              if(i%20==0){
                   System.out.println(Thread.currentThread().getName()+": "+  i +"  yield一下");
                   Thread.yield();
              }
          }
     }
     
}
 
```
 
---



可以看到执行了yield之后。其实线程是放出一点资源的。或许看的不是很明显。因为我们现在的电脑cpu大多是多核心多线程的。在test1交出cpu的资源时。其他线程已经在执行过程中。所以这个test1在交出cpu资源之后瞬间又进入running的状态


那么我们是否可以设想一下这玩意的用处的场景。首先这个操作会不会释放锁呢？我们来看一下

```java 
package ThreadTest;
public class YieldTest {
     public static void main(String[] args) {
          
          Thread test1 = new Yield("test1");
          Thread test2 = new Yield("test2");
          Thread test3 = new Yield("test3");
          
          test1.start();
          test2.start();
          test3.start();
     }
}
//继承Thread
class Yield extends Thread{
     private static Object lock = new Object();
     public Yield(){
          
     }
     
     public Yield(String name){
          super(name);
     }
     
     @Override
     public void run() {
          // TODO Auto-generated method stub
          synchronized (lock){
              for(int i=0;i<10;i++){
                   System.out.println(Thread.currentThread().getName()+":  "+i);
                        if(i%2==0){
                             System.out.println(Thread.currentThread().getName()+": "+  i +"  yield一下");
                             Thread.yield();
                        }
                   }
              }
          }
}

运算结果：

test1: 0
test1: 0  yield一下
test1: 1
test1: 2
test1: 2  yield一下
test1: 3
test1: 4
test1: 4  yield一下
test1: 5
test1: 6
test1: 6  yield一下
test1: 7
test1: 8
test1: 8  yield一下
test1: 9
test3: 0
test3: 0  yield一下
test3: 1
test3: 2
test3: 2  yield一下
test3: 3
test3: 4
test3: 4  yield一下
test3: 5
test3: 6
test3: 6  yield一下
test3: 7
test3: 8
test3: 8  yield一下
test3: 9
test2: 0
test2: 0  yield一下
test2: 1
test2: 2
test2: 2  yield一下
test2: 3
test2: 4
test2: 4  yield一下
test2: 5
test2: 6
test2: 6  yield一下
test2: 7
test2: 8
test2: 8  yield一下
test2: 9
 
```
 
---
发现了其实并不会释放锁。yield会放弃cpu资源重新抢夺，但是不会释放锁。那么既然它不能释放锁。说明这个不适合在用锁的环境中使用。
那么我们什么时候使用这个yield呢？线程其实是分线程的优先级的，虽然线程中优先级高的不全是先执行结束。但是优先级高的线程总是先执行结束。那么我们可以设想一下，如果现在有两个线程一个是优先级高的线程，进行的是重要的操作比如是付工资，
但是还有一个优先级比较低的线程是进行人员上周考勤的统计。优先级较低的线程其实可能占用的cpu资源的时间长，所以我们希望把资源更多的交给付工资这个线程。所以我们可以把yield来做。


—— LingFeng 后记于 2018.11.15