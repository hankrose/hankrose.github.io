---
layout:     post
title:      "java多线程"
subtitle:   " \"java多线程七(Thread wait的用法)\""
date:       2018-11-20 12:00:00
author:     "LingFeng"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - 多线程
---

> “Yeah It's on. ”


## 前言

 Thread类中把线程从running状态转化为非runnable状态有一个方法就是wait方法。wait方法是线程的等待状态。我们来看看wait方法简单运用
<p id = "build"></p>
---

## 正文
 
下面是一个wait方法的代码

---


```java 
package ThreadTest;
/**
* thread wait方法详解
* @author lingfengz
*
*/
public class WaitTest {
     public static void main(String[] args) {
          Waitthread wait = new Waitthread();
          Thread test1 = new Thread(wait,"线程1");
          Thread test2 = new Thread(wait,"线程2");
          Thread test3 = new Thread(wait,"线程3");
          Thread test4 = new Thread(wait,"线程4");
          Thread test5 = new Thread(wait,"线程5");
          
          test1.start();
          test2.start();
          test3.start();
          test4.start();
          test5.start();
     }
}
class Waitthread implements Runnable{
     private Object lock = new Object();
     /**
      * 实现run方法
      */
     @Override
     public void run() {
          // TODO Auto-generated method stub
          try{
              synchronized(lock){
                    System.out.println(Thread.currentThread().getName()+"开始了");
                   lock.wait();
              }
          }catch(InterruptedException e){
              e.getStackTrace();
          }
                    System.out.println(Thread.currentThread().getName()+"结束了");
     }
}
 
```
 
---



从代码运行的结果可以看出，wait是会释放锁的,但是线程下一步操作却被挂起了。那么我们来看看通过notify();会不会唤醒线程

```java 
package ThreadTest;
/**
* thread wait方法详解
* @author lingfengz
*
*/
public class WaitTest {
     public static void main(String[] args) {
       new Thread(new Wait1()).start();
       
       new Thread(new Wait2()).start();
     }
     
     public static class Wait1 implements Runnable{
      /**
       * 实现体育老师安排体育课代表进行点名
       */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              //这个步骤我们可以想象成锁住的是一节体育课
              synchronized (WaitTest.class) {
                   System.out.println("体育老师安排体育委员进行点名");
                   try {
                        //体育老师在楼下等待,并且交出体育课的锁。等待体育课代表告诉他点名结束,他开始接管这节体育课
                        WaitTest.class.wait();
                   } catch (InterruptedException e) {
                        // TODO Auto-generated catch  block
                        e.printStackTrace();
                   }
                   System.out.println("体育老师得到点名的结果开始上课");
              }
          }
       
     }
     
     public static class Wait2 implements Runnable{
      
       /**
       * 实现体育课代表进行队列点名
       */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              synchronized (WaitTest.class) {
                   System.out.println("体育委员进行点名");
                   
                   try {
                        //点名需要花费的时间
                        Thread.sleep(1000L);
                   } catch (InterruptedException e) {
                        // TODO Auto-generated catch  block
                        e.printStackTrace();
                   }
                   System.out.println("体育委员点名结束");
                   //通知体育老师点名结束了
                       WaitTest.class.notify();
                   
                   System.out.println("体育课代表带人去操场");
              }
          }
       
       
     }
}

运行结果:
体育老师安排体育委员进行点名
体育委员进行点名
体育委员点名结束
体育课代表带人去操场
体育老师得到点名的结果开始上课
 
```
 
---
我们从上面的代码可以看到test1这个对象锁.wait后。Waitthread线程中notify();进行了唤醒。线程被唤醒,如果我们把notify();给注销掉会有什么样的结果呢.

```java 
package ThreadTest;
/**
* thread wait方法详解
* @author lingfengz
*
*/
public class WaitTest {
     public static void main(String[] args) {
       new Thread(new Wait1()).start();
       
       new Thread(new Wait2()).start();
     }
     
     public static class Wait1 implements Runnable{
      /**
       * 实现体育老师安排体育课代表进行点名
       */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              //这个步骤我们可以想象成锁住的是一节体育课
              synchronized (WaitTest.class) {
                   System.out.println("体育老师安排体育委员进行点名");
                   try {
                        //体育老师在楼下等待,并且交出体育课的锁。等待体育课代表告诉他点名结束,他开始接管这节体育课
                        WaitTest.class.wait();
                   } catch (InterruptedException e) {
                        // TODO Auto-generated catch  block
                        e.printStackTrace();
                   }
                   System.out.println("体育老师得到点名的结果开始上课");
              }
          }
       
     }
     
     public static class Wait2 implements Runnable{
      
       /**
       * 实现体育课代表进行队列点名
       */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              synchronized (WaitTest.class) {
                   System.out.println("体育委员进行点名");
                   
                   try {
                        //点名需要花费的时间
                        Thread.sleep(1000L);
                   } catch (InterruptedException e) {
                        // TODO Auto-generated catch  block
                        e.printStackTrace();
                   }
                   System.out.println("体育委员点名结束");
                   //通知体育老师点名结束了
                   //WaitTest.class.notify();
                   
                   System.out.println("体育课代表带人去操场");
              }
          }
       
       
     }
}


运行结果

体育老师安排体育委员进行点名
体育委员进行点名
体育委员点名结束
体育课代表带人去操场
 
```
---

从上面可以看到，体育委员没有通知体育老师点名结束(notify();),那么体育老师还在等待体育委员的通知。但是体育委员已经带人去操场了。这节体育课，如果体育委员不通知体育老师点名结束的，体育老师就不会来上课了。
从上面我们可以看到wait();方法是无限等待。那么我们生活中体育老师中，体育委员忘了通知体育老师，体育老师就不来上课了？答案肯定是体育老师还是会过来，但是肯定有一段等待的时间
那么我们试试给wait的其他方法。给wait一个时间的参数

```java

package ThreadTest;
import java.text.SimpleDateFormat;
import java.util.Date;
/**
* thread wait方法详解
* @author lingfengz
*
*/
public class WaitTest {
     public static void main(String[] args) {
       new Thread(new Wait1()).start();
       
       new Thread(new Wait2()).start();
     }
     
     public static class Wait1 implements Runnable{
      /**
       * 实现体育老师安排体育课代表进行点名
       */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              //这个步骤我们可以想象成锁住的是一节体育课
              synchronized (WaitTest.class) {
                   System.out.println("体育老师安排体育委员进行点名");
                   try {
                        //体育老师开始看手表，准备上课
                        System.out.println("体育老师开始等待时间:"+new  SimpleDateFormat("yyyy/MM/dd-HH:mm:ss:SSS").format(new  Date()));
                        //体育老师在楼下等待,并且交出体育课的锁。等待体育课代表告诉他点名结束,他开始接管这节体育课。如果1000L之后体育课代表还不通知他，他将直接去操场上课
                        WaitTest.class.wait(1000L);
                   } catch (InterruptedException e) {
                        // TODO Auto-generated catch  block
                        e.printStackTrace();
                   }
                   System.out.println("体育老师结束等待时间:"+new  SimpleDateFormat("yyyy/MM/dd-HH:mm:ss:SSS").format(new  Date()));
                   //System.out.println("体育老师得到点名的结果开始上课");
                   System.out.println("体育老师去操场开始上课");
              }
          }
       
     }
     
     public static class Wait2 implements Runnable{
      
       /**
       * 实现体育课代表进行队列点名
       */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              synchronized (WaitTest.class) {
                   System.out.println("体育委员进行点名");
                   
                   try {
                        //点名需要花费的时间
                        Thread.sleep(1000L);
                   } catch (InterruptedException e) {
                        // TODO Auto-generated catch  block
                        e.printStackTrace();
                   }
                   System.out.println("体育委员点名结束");
                   //通知体育老师点名结束了
                   //WaitTest.class.notify();
                   
                   System.out.println("体育课代表带人去操场");
              }
          }
     }
}


运行结果：

体育老师安排体育委员进行点名
体育老师开始等待时间:2019/03/18-13:01:08:995
体育委员进行点名
体育委员点名结束
体育课代表带人去操场
体育老师结束等待时间:2019/03/18-13:01:09:999
体育老师去操场开始上课
```

---

从上面的执行结果来看wait,如果有时间参数的话，在时间参数之内被没被唤醒的话，在时间到达之后他可以自己唤醒。
所以我们得出结论：wait的几种情况
首先wait是会放弃锁，进入等待区。第二wait可以被其他线程所唤醒。
第三如果wait有时间参数，就算没有其他线程唤醒，他也会自动唤醒。


—— LingFeng 后记于 2018.11.20