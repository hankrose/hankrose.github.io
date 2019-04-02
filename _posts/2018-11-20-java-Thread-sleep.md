---
layout:     post
title:      "java多线程"
subtitle:   " \"java多线程八(Thread sleep的用法)\""
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

我们上面讲了wait的用法，下面我们来讲seleep的用法。首先我们还是将上一篇的上体育课的例子拿来做一个示例
<p id = "build"></p>
---

## 正文
 
我们在里面用一下sleep方法

---


```java 
package ThreadTest;
import java.text.SimpleDateFormat;
import java.util.Date;
/**
* Thread sleep运用实例
* @author lingfengz
*
*/
public class SleepTest {
     public static void main(String[] args) {
      new Thread(new Sleep1()).start();
      
      new Thread(new Sleep2()).start();
    }
    
    public static class Sleep1 implements Runnable{
     /**
      * 实现体育老师安排体育课代表进行点名
      */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              //这个步骤我们可以想象成锁住的是一节体育课
              synchronized (SleepTest.class) {
                   System.out.println("体育老师安排体育委员进行点名");
                   try {
                        //体育老师开始看手表，准备上课
                        System.out.println("体育老师开始等待时间:"+new  SimpleDateFormat("yyyy/MM/dd-HH:mm:ss:SSS").format(new  Date()));
                        //体育老师在楼下等待,然后玩一会手机，休息1000L时间
                        Thread.sleep(1000L);
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
    
    public static class Sleep2 implements Runnable{
     
      /**
       * 实现体育课代表进行队列点名
       */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              synchronized (SleepTest.class) {
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
体育老师开始等待时间:2019/03/18-13:44:53:674
体育老师结束等待时间:2019/03/18-13:44:54:676
体育老师去操场开始上课
体育委员进行点名
体育委员点名结束
体育课代表带人去操场
 
```
 
---



从上面可以看出sleep并不会释放锁。那么如果sleep的时间过长会怎么样？

```java 
package ThreadTest;
import java.text.SimpleDateFormat;
import java.util.Date;
/**
* Thread sleep运用实例
* @author lingfengz
*
*/
public class SleepTest {
     public static void main(String[] args) {
      new Thread(new Sleep1()).start();
      
      new Thread(new Sleep2()).start();
    }
    
    public static class Sleep1 implements Runnable{
     /**
      * 实现体育老师安排体育课代表进行点名
      */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              //这个步骤我们可以想象成锁住的是一节体育课
              synchronized (SleepTest.class) {
                   System.out.println("体育老师安排体育委员进行点名");
                   try {
                        //体育老师开始看手表，准备上课
                        System.out.println("体育老师开始等待时间:"+new  SimpleDateFormat("yyyy/MM/dd-HH:mm:ss:SSS").format(new  Date()));
                        //体育老师在楼下等待,然后玩一会手机，休息1000L时间
                        Thread.sleep(10000L);
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
    
    public static class Sleep2 implements Runnable{
     
      /**
       * 实现体育课代表进行队列点名
       */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              synchronized (SleepTest.class) {
                   System.out.println("体育课老师已经到楼下了");
                   System.out.println("体育委员开始点名时间:"+new  SimpleDateFormat("yyyy/MM/dd-HH:mm:ss:SSS").format(new  Date()));
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
体育老师开始等待时间:2019/03/18-13:49:47:031
体育老师结束等待时间:2019/03/18-13:49:57:046
体育老师去操场开始上课
体育课老师已经到楼下了
体育委员开始点名时间:2019/03/18-13:49:57:046
体育委员进行点名
体育委员点名结束
体育课代表带人去操场
 
```
 
---
这么看来sleep睡的时间太长会导致，线程长期持有锁。那么有没有方法可以提前唤醒他呢？方法是有的
我们来看看interrupt怎么来提前唤醒

```java 
package ThreadTest;
import java.text.SimpleDateFormat;
import java.util.Date;
/**
* Thread sleep运用实例
* @author lingfengz
*
*/
public class SleepTest {
     public static void main(String[] args) {
          Date date = new Date();
          Thread thread1 = new Thread(new Sleep1(date));
          thread1.start();
          //学校规定最多准备5分钟就得上课
          while(true){
              //如果现在的时间超过5分钟,老师必须去上课
              if(new Date().getTime()>new  Date((date.getTime()+2*1000*3)).getTime()){
                   thread1.interrupt();
                   break;
              }
          }
          Thread thread2 = new Thread(new Sleep2(date));
          thread2.start();
    }
    
    public static class Sleep1 implements Runnable{
      private Date date;
      private int oneMin = 2*1000;
      
      public Sleep1(Date date){
          this.date = date;
      }
     /**
      * 实现体育老师安排体育课代表进行点名
      */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              //这个步骤我们可以想象成锁住的是一节体育课
              synchronized (SleepTest.class) {
                   System.out.println("体育老师安排体育委员进行点名");
                   try {
                        
                        //体育老师开始看手表，准备上课
                        System.out.println("体育老师开始等待时间:"+new  SimpleDateFormat("yyyy/MM/dd-HH:mm:ss:SSS").format(date));
                        //体育老师在楼下等待,然后玩一会手机，休息10分钟时间
                        Thread.sleep(oneMin*6);
                   } catch (InterruptedException e) {
                        //老师被警告，要去上课了
                        System.out.println("老师被警告了，过了五分钟了。必须要去上课了");
                   }
                   System.out.println("体育老师结束等待时间:"+new  SimpleDateFormat("yyyy/MM/dd-HH:mm:ss:SSS").format(new  Date()));
                   //System.out.println("体育老师得到点名的结果开始上课");
                   System.out.println("体育老师去操场开始上课");
              }
          }
      
    }
    
    public static class Sleep2 implements Runnable{
      private Date date;
      private int oneMin = 60*1000;
      
      public Sleep2(Date date){
          this.date = date;
      }
      /**
       * 实现体育课代表进行队列点名
       */
          @Override
          public void run() {
              // TODO Auto-generated method stub
              synchronized (SleepTest.class) {
                   System.out.println("体育课老师已经到楼下了");
                   System.out.println("体育委员开始点名时间:"+new  SimpleDateFormat("yyyy/MM/dd-HH:mm:ss:SSS").format(new  Date()));
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

运行结果:

体育老师安排体育委员进行点名
体育老师开始等待时间:2019/03/18-14:35:21:637
老师被警告了，过了五分钟了。必须要去上课了
体育老师结束等待时间:2019/03/18-14:35:27:638
体育老师去操场开始上课
体育课老师已经到楼下了
体育委员开始点名时间:2019/03/18-14:35:27:639
体育委员进行点名
体育委员点名结束
体育课代表带人去操场
 
```
---

从这里我们可以看出 sleep时间过长的话，我们可以通过interrupt方法来进行状态的修改。但是必须要catch捕捉到。然后再进行下一步的操作

我们来总结一下sleep的作用：首先并不会释放锁。但是会释放cpu的资源，如果sleep时间过长可以调用interrupt方法进行唤醒。但是要捕捉异常。会进入异常。

—— LingFeng 后记于 2018.11.20