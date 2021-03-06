---
layout:     post
title:      "Spring Aop详解"
subtitle:   " \"Spring Aop详解(一)\""
date:       2019-12-12 22:30:00
author:     "LingFeng"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - Spring Aop
---

> “Yeah It's on. ”


## 前言

最近有个同事突然问我spring aop，我虽然理解aop的原理。但是我说的不是很能让他明白。于是有了这篇博客，下面我们开始从最简单的开始了解什么是spring的aop
 
 
<p id = "build"></p>
---

## 正文
 
    1.Spring Aop 是什么？

    2.Aop 能做什么？
	
	3.Aop 和 代理之间的前世今生
	
---
  <h3>1.Spring Aop 是什么？</h3>	
  
  首先我们先看一下这个AOP的单词拆解。面向切面编程，OOP是面向对象编程，看起来是不是很像呢? 按照spring官方文档上说的，Aop的拼写是 Aspect Oriented Programming。和传统的OOP的不同点在哪里？下面我首先举个简单的例子，就拿博主最近做的一个T平台(通用性服务平台) 为例，首先这个平台目前有个需求就是可以通过配置场景和场景对应的团队，进行外部任务数据接入后可以快速的分配
  
  那么我们来理一下这个需求:
  
  1.任务来源多样化
  2.场景和任务来源是对应的
  3.任务分配的对象也是可配置多样化的
  4.分配规则的多样化
  
  按照我们传统的OOP思想的话。那么我们现在有3个来源，是否就是要写3个来源的分配代码呢？那么这样我们的工作重复劳动的性质很大，就成为了所谓的IT民工了，那么博主就想，能不能把任务的获取，或者是任务分配后插入数据这个重复的动作给他抽取出来。那么问题出现了，抽取出来的方法我放在一个类中，那么我多种场景的类都要获取到这个类的对象才能调用到这个类的方法，这和我的初衷是违背的
那么博主想了想是不是可以让spring来帮我们把这个抽取公共方法的类的方法自动给添加到我各种来源的分配逻辑上面呢？显然这个问题不是博主一个人遇到的，早在很多年前就有很多大牛遇到了，所以编写spring的大牛呢就想出了一个方法，就是把这个公共方法给定义一个方式来自动给添加到你要添加的类上的方式。这个方式叫Spring Aop。那么Aop这个东西怎么做到的呢？
 
 1.首先我们jvm能编译的每个独立的method(就是业务类中的独立方法) 都可以被自动添加代码。所以这个方法我们称作连接点。所有能被jvm编译出的独立方法都叫连接点
 
 2.有了连接点之后，我们是不是要区分那些方法可以被spring aop去添加代码呢？那些可以被添加公共方法的我们叫做 切入点。打个比方，我们玩打地鼠，横三竖四，12个地鼠，只有地鼠冒头了我们才需要去用锤子打地鼠。显然，12个地鼠都叫连接点，但是只有冒头的地鼠才可以被称作切入点 所以切入点一定是连接点，但是连接点不一定是切入点(没有冒头的地鼠显然不能用锤子去打，因为你也打不到)
 
 3.有了切入点，那么我们是不是要把切入的代码给找出来呢？显然被那么被切入的代码，就是被添加到切入点方法上的代码。我们把他叫做通知(这个通知具体指的是那些被抽取出的方法)
 
 4.那么通知方法是不是该有个类去存储呢？很好通知方法所在的类，叫通知类
 
 5.通知类有了，切入点有了，问题又出现了，java是由很多的obj，也就是类组成，也许有的类中的方法你想织入，有的类你不想织入。那么再引申去一个概念 目标类，就是可以被织入的类。有3个打地鼠的机器，你总不会用你的锤子去锤别人的地鼠把。
 
 6.有的方法或许需要和通知方法做一些目标类的共享变量的传输，所以spring给我们提供了一个叫引入的概念。这个引入就是为了解决目标类的共享变量传输提供的
 
 7.那么通知方法有了，切入点方法也有了，那么我们怎么把通知方法给添加到切入点方法呢？spring把这个东西叫织入。就是织入进去，顾名思义就是把通知方法中的代码块给添加到切入点方法上，目前织入的方式有4种，后续会详细讲解，博主用的最多的就是方法前织入，方法后织入

 8.那么最后一个问题，我什么时候该把通知类中的通知方法给添加到需要被织入的切入点方法中呢？那么引入了概念？类似于c++的静态编译和动态连编。织入分为静态织入，编译织入，运行时织入。spring目前用的是运行时织入，这个后续也会讲几种织入时间不同的，性能上的不同

---

总结
    我们首先了解AOP是什么，下面我会持续更新，我们代码上怎么实现和运用。会用传统的java代码和最新的springboot方式来实现spring aop的运用

—— LingFeng 后记于 2019.12.12