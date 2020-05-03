---
layout:     post
title:      "java基础之继承"
subtitle:   " \"java(一)基础之继承\""
date:       2020-04-11 22:30:00
author:     "LingFeng"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - Spring Aop
---

> “Yeah It's on. ”


## 前言

最近有个想法就是把工作中使用过的知识进行一次总体的记录和知识点的梳理,于是便有了 [java_interview](https://github.com/hankrose/java-interview) 这个项目。这个项目我会带着大家学习下面这个思维导图上的知识

<img src="img/java-interview.png">
 
<p id = "build"></p>
---

## 正文
 
    1.java 继承怎么在调用的时候分清父类和子类方法相同名称的方法的
	
	2.为什么接口实现多继承而类却只能单根继承

    3.this和super关键字的含义和使用
	
---
  <h3><font color="Process Cyan">1.怎么在调用的时候分清父类和子类方法相同名称的方法的?</font></h3>	
  
  首先java为了杜绝c++的多继承带来的弊端,防止后期子类的继承关系过于复杂,规定了只能使用单根继承。然后我们一个子类继承一个父类。就像下面这段代码

  ```
    package org.lingfeng.web;

    /**
    * Created by lingfeng on 2020/4/11.
    */
    public class ExtendsFather {

        void Test(){
            System.out.println("ExtendsFather sout");
        }
    }

  ```

  子类实现如下
  
  ```
  package org.lingfeng.web;

    /**
    * Created by lingfeng on 2020/4/11.
    */
    public class ExtendsChild extends ExtendsFather{

        @Override
        void Test() {
            System.out.println("ExtendsChild sout");
        }
    }
  ```

  下面我们来写一个测试类来看一下

  ```
  package org.lingfeng.web;

    /**
    * Created by lingfeng on 2020/4/11.
    */
    public class TestMain {

        public static void main(String[] args) {
            ExtendsFather test = new ExtendsChild();
            test.Test();
        }

    }
  ```  
  -----
  我们将父类的指针指向了子类的实现类.那么java执行这个方法的时候到底会去使用父类的方法呢?还是子类的方法呢?

  <font color="Process Blue">答案显而易见 使用了子类的方法</font>  

  ```
   ExtendsChild sout
  ```
   为什么jvm编译的时候可以选择到子类的方法而不是父类的方法,这一点我们在后面的 [多态]() 会详细的说明。
  
   那么学会继承,有什么好处呢?
   1. 可以对父类的方法进行扩展
   2. 可以减少很多重复的代码量
   3. <font color="red">解耦</font> 这点很重要 后面会说到 [桥接模式]() 到底怎么将庞大的继承体系给解耦的  

   <h3><font color="Process Cyan">2.为什么接口实现多继承而类却只能单根继承?</font></h3>

   这个问题我们来看一下,前面的那段代码，我们再新增一个父类。

   ```
   package org.lingfeng.web;

    /**
    * Created by lingfeng on 2020/4/11.
    */
    public class ExtendsUncle {

        void Test(){
            System.out.println("ExtendsFather sout");
        }
    
    }
   ```
----
   我们再将 子类的方法直接改为调用父类实现呢  

```
     package org.lingfeng.web;

    /**
    * Created by lingfeng on 2020/4/11.
    */
    public class ExtendsChild extends ExtendsFather extends ExtendsUncle{

      
    }
```
----
会发现这样写，编译器会报错。为什么呢? 我们可以想一下，很好理解，我们把这个事情解释成一件生活中经常会遇到的一些事情。
1. 小明(子类) 有两双鞋(两个父类有相同名称的方法)
2. 小明和母亲(jvm) 说妈妈给我拿一双鞋(选择执行父类的方法)
3. 母亲看着鞋架上的两双鞋,流露出选择困难的表情
4. 于是乎母亲和小明说以后你要穿的鞋子只能放一双在鞋架上，要不然我怎么知道你要穿那双？

其实就是这么简单的事情，生活中我们大家也经常会遇到。看到这里有的小伙伴可能会问了,那你前面说c++是多继承的,它是怎么实现的?  

其实c++的方法简单粗暴,在子类调用多个父类同名的方法的时候，需要指定是调用的具体哪个父类的同名方法。但是这样对代码的侵入性太大了。如果哪天这个父类被删除了？只留了一个其他父类呢?那么你需要改的代码也太多了

所以java直接给你简单粗暴的，只允许你单根继承，从根本上解决这个问题。

下面说接口
  因为接口是只有定义，具体方法需要子类去实现的。所以接口继承的父类接口有相同名称的方法定义的时候，其实并不产生冲突。我们还是拿前面的小明和鞋子进行举例子
  1. 小明的妈妈让小明去买鞋(实现接口)
  2. 具体买什么鞋子，妈妈并不知道(只是定义了接口)
  3. 小明在adidas和nike门口转悠了半天最后选择买adidas(子类的实现)

  到这里大家应该对继承有点深刻的了解了吧.下面我们继续

 <h3><font color="Process Cyan">3.this和super关键字的含义和使用</font></h3>


总结
    我们首先了解AOP是什么，下面我会持续更新，我们代码上怎么实现和运用。会用传统的java代码和最新的springboot方式来实现spring aop的运用

—— LingFeng 后记于 2019.12.12