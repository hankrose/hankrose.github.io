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

&nbsp;&nbsp;最近有个想法就是把工作中使用过的知识进行一次总体的记录和知识点的梳理,于是便有了 [java_interview](https://github.com/hankrose/java-interview) 这个项目。这个项目我会带着大家学习下面这个思维导图上的知识

<img src="../img/java-interview.png" >
 
<p id = "build"></p>
---

## 正文
 
1. <h4>java 继承怎么在调用的时候分清父类和子类方法相同名称的方法的</h4>
	
2. <h4>为什么接口实现多继承而类却只能单根继承</h4>

3. <h4>this和super关键字的含义和使用</h4>
	
---
  <h3><font color="Process Cyan">1.怎么在调用的时候分清父类和子类方法相同名称的方法的?</font></h3>	
  
  &nbsp;&nbsp;首先java为了杜绝c++的多继承带来的弊端,防止后期子类的继承关系过于复杂,规定了只能使用单根继承。然后我们一个子类继承一个父类。就像下面这段代码

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

  &nbsp;&nbsp;下面我们来写一个测试类来看一下

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
  &nbsp;&nbsp;我们将父类的指针指向了子类的实现类.那么java执行这个方法的时候到底会去使用父类的方法呢?还是子类的方法呢?

  <font color="Process Blue">答案显而易见 使用了子类的方法</font>  

  ```
   ExtendsChild sout
  ```
   &nbsp;&nbsp;为什么jvm编译的时候可以选择到子类的方法而不是父类的方法,这一点我们在后面的 [多态]() 会详细的说明。
  
   &nbsp;&nbsp;那么学会继承,有什么好处呢?
   1. 可以对父类的方法进行扩展
   2. 可以减少很多重复的代码量
   3. <font color="red">解耦</font> 这点很重要 后面会说到 [桥接模式]() 到底怎么将庞大的继承体系给解耦的  

   <h3><font color="Process Cyan">2.为什么接口实现多继承而类却只能单根继承?</font></h3>

   &nbsp;&nbsp;这个问题我们来看一下,前面的那段代码，我们再新增一个父类。

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
   &nbsp;&nbsp;我们再将 子类的方法直接改为调用父类实现呢  

```
     package org.lingfeng.web;

    /**
    * Created by lingfeng on 2020/4/11.
    */
    public class ExtendsChild extends ExtendsFather extends ExtendsUncle{

      
    }
```
----
&nbsp;&nbsp;会发现这样写，编译器会报错。为什么呢? 我们可以想一下，很好理解，我们把这个事情解释成一件生活中经常会遇到的一些事情。
1. 小明(子类) 有两双鞋(两个父类有相同名称的方法)
2. 小明和母亲(jvm) 说妈妈给我拿一双鞋(选择执行父类的方法)
3. 母亲看着鞋架上的两双鞋,流露出选择困难的表情
4. 于是乎母亲和小明说以后你要穿的鞋子只能放一双在鞋架上，要不然我怎么知道你要穿那双？

&nbsp;&nbsp;其实就是这么简单的事情，生活中我们大家也经常会遇到。看到这里有的小伙伴可能会问了,那你前面说c++是多继承的,它是怎么实现的?  

&nbsp;&nbsp;其实c++的方法简单粗暴,在子类调用多个父类同名的方法的时候，需要指定是调用的具体哪个父类的同名方法。但是这样对代码的侵入性太大了。如果哪天这个父类被删除了？只留了一个其他父类呢?那么你需要改的代码也太多了

&nbsp;&nbsp;所以java直接给你简单粗暴的，只允许你单根继承，从根本上解决这个问题。

&nbsp;&nbsp;下面说接口
  因为接口是只有定义，具体方法需要子类去实现的。所以接口继承的父类接口有相同名称的方法定义的时候，其实并不产生冲突。我们还是拿前面的小明和鞋子进行举例子
  1. 小明的妈妈让小明去买鞋(实现接口)
  2. 具体买什么鞋子，妈妈并不知道(只是定义了接口)
  3. 小明在adidas和nike门口转悠了半天最后选择买adidas(子类的实现)

  到这里大家应该对继承有点深刻的了解了吧.下面我们继续

 <h3><font color="Process Cyan">3.this和super关键字的含义和使用</font></h3>

&nbsp;&nbsp;如果父类有一个成员变量(我们暂时默认他是public修饰的) ，子类也有一个成员变量。当我们不指明调用具体是某个成员变量的话，具体会怎么样?

&nbsp;&nbsp;很好我们开始引入第一个关键字,这不是变量也不是什么类型，只是也一个特殊的语法糖 super .为什么要引入super呢。java是一个oop语言，如果我们需要调用某一个父类的方法的时候。必须要生成一个父类的对象。

&nbsp;&nbsp;那么疑问来了，我子类继承一个父类，我也没有显式的去获取父类的对象。为什么我不重写父类的方法的时候，我可以用子类直接调用到父类的方法呢? 总结一下疑问的点。  

1. 为什么子类不显示的生成父类对象,可以调用父类的方法
2. 如果子类和父类有相同名称的成员变量，在不显示的调用方法的时候具体会调用哪一个？

&nbsp;&nbsp; 我们先看第一个问题  <font color="greed">为什么子类不显示的生成父类对象,可以调用父类的方法</font>
&nbsp;&nbsp; 我们首先给子类定义一个无参构造，父类不显式的定义
```
package org.lingfeng.web;

    /**
    * Created by lingfeng on 2020/4/11.
    */
    public class ExtendsChild extends ExtendsFather{

        public ExtendsChild() {
             System.out.println("I am child");
        }
        
    }
```
&nbsp;&nbsp;我们看一下这个main的执行的结果
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

结果是:
I am child
ExtendsFather sout
```
&nbsp;&nbsp;那么我们在父类中也加入构造方法呢?

```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class ExtendsFather {

  public ExtendsFather() {
    System.out.println("I am father");
  }

  void Test(){
    System.out.println("ExtendsFather sout");
  }
}
```
&nbsp;&nbsp;
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

结果是:
I am father
I am child
ExtendsFather sout
```
&nbsp;&nbsp;这里可以发现子类执行构造方法的时候，会默认生成一个父类的实例，通过父类的无参构造来生成。

&nbsp;&nbsp; 那我们在子类，加上super这个语法糖呢?

```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class ExtendsChild extends ExtendsFather{

  public ExtendsChild() {
    super();
    System.out.println("I am child");
  }

}
```

&nbsp;&nbsp;那么我们再来执行一次

```
结果:
I am father
I am child
ExtendsFather sout
```
&nbsp;&nbsp;看到这里不难发现，其实这个就是jvm帮我们默认生成无参构造一样，在我们子类执行构造方法的时候，默认生成了super();这个语法糖。这个语法糖的作用就是帮我们默认生成了一个父类实例

&nbsp;&nbsp;作为一个码农，我本着求真的精神。向下面继续的了解。如果说如果父类子类都有两个属性叫 age 和 name 呢;

很好我们现在首先来试一下
```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class ExtendsFather {

  public String name = "father li";

  public Integer age = 35;

  public ExtendsFather() {
    System.out.println("my name is " + name + ",my age is " + age);
  }

  void Test(){
    System.out.println("ExtendsFather sout");
  }
}
```

&nbsp;&nbsp;子类也加上这段方法
```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class ExtendsChild extends ExtendsFather{

  public String name = "child li";

  public Integer age = 10;


  public ExtendsChild() {
    super();
    System.out.println("my name is " + name + ",my age is " + age);
  }

}
```
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

结果:
my name is father li,my age is 35
my name is child li,my age is 10
ExtendsFather sout
```
&nbsp;&nbsp;通过super这个语法糖使用的还是父类的成员变量。那么我考虑了一下如果是获取父类的成员变量的值呢(如果父类的成员变量是private，需要使用父类提供的get方法获取。这一块我在  [封装]() 的时候会解释的)

&nbsp;&nbsp;直接上方法

```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class ExtendsChild extends ExtendsFather{

  public String name = "child li";

  public Integer age = 10;


  public ExtendsChild() {
    super();
    System.out.println("my name is " + name + 
        ",my age is " + age + " my father is " + super.name );
  }
}
```

```
结果:
my name is father li,my age is 35
my name is child li,my age is 10 my father is father li
ExtendsFather sout
```
&nbsp;&nbsp;下面我们来讲一下this这个关键字的使用

&nbsp;&nbsp;我们首先在修改一下子类的构造方法

```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class ExtendsChild extends ExtendsFather{

  public String name = "child li";

  public Integer age = 10;


  public ExtendsChild() {
    super();
    String name ="";
    int age = 0;
    System.out.println("my name is " + name + ",my age is " + age + " my father is " + super.name );
  }

}
```

```
结果:
my name is father li,my age is 35
my name is ,my age is 0 my father is father li
ExtendsFather sout
```
&nbsp;&nbsp;从这里可以看出来其实我们局部变量比成员变量的优先级要高。但是我们有的时候就是想获取成员变量怎么办? 这个时候就可以引入,this这个关键字

```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class ExtendsChild extends ExtendsFather{

  public String name = "child li";

  public Integer age = 10;


  public ExtendsChild() {
    super();
    String name ="";
    int age = 0;
    System.out.println("my name is " + this.name + ",my age is " + this.age + " my father is " + super.name );
  }

}
```

```
结果:
my name is father li,my age is 35
my name is child li,my age is 10 my father is father li
ExtendsFather sout
```

&nbsp;&nbsp;为什么用了this就可以获取到成员变量的值呢? 我们把this 给输出出来看一下

```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class ExtendsChild extends ExtendsFather{

  public String name = "child li";

  public Integer age = 10;


  public ExtendsChild() {
    super();
    String name ="";
    int age = 0;
    System.out.println("my name is " + this.name + ",my age is " + this.age + " my father is " + super.name );
    System.out.println(this);
  }

}
```

```
my name is father li,my age is 35
my name is child li,my age is 10 my father is father li
org.lingfeng.web.ExtendsChild@433c675d
ExtendsFather sout
```

&nbsp;&nbsp;从这里可以看出来其实this关键字是一个指向当前对象的一个引用。这也很好的说明了为什么通过this就可以获取到当前对象的成员变量的值.(变量的作用域)

&nbsp;&nbsp;super和this的混合使用，假设一个情景，你母亲让你父亲刷碗，你父亲把水给准备好了，临时有事走了，让你接着刷碗。你到了厨房应该是直接刷碗。不用准备水。那么java对于继承的关系也是一样。那么我们继续修改父类代码，增加一个刷碗的方法

```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class ExtendsChild extends ExtendsFather{

  public String name = "child li";

  public Integer age = 10;


  public ExtendsChild() {
    super();
    String name ="";
    int age = 0;
    System.out.println("my name is " + this.name + ",my age is " + this.age + " my father is " + super.name );
    System.out.println(this);
  }

  @Override
  void washDishes() {
    super.washDishes();
    System.out.println("我接着刷碗");
  }
}
```

```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class ExtendsFather {

  public String name = "father li";

  public Integer age = 35;

  public ExtendsFather() {
    System.out.println("my name is " + name + ",my age is " + age);
  }

  void Test(){
    System.out.println("ExtendsFather sout");
  }

  void washDishes(){
    System.out.println("母亲安排父亲刷碗");
    System.out.println("父亲放好了刷碗水");
    System.out.println("父亲接到一个紧急电话，走了");
  }
}

```

&nbsp;&nbsp;

```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class TestMain {

  public static void main(String[] args) {
    ExtendsFather test = new ExtendsChild();
    test.washDishes();
  }

}

结果:
my name is father li,my age is 35
my name is child li,my age is 10 my father is father li
org.lingfeng.web.ExtendsChild@433c675d
母亲安排父亲刷碗
父亲放好了刷碗水
父亲接到一个紧急电话，走了
我接着刷碗
```

&nbsp;&nbsp;从上面看来，子类继承父类，重写了父类方法后，可以通过super关键字，调用父类的方法，并且增加自己的逻辑。

&nbsp;&nbsp;然而我自己又有一个疑问了,this能不能在构造方法中调用其他构造呢。super继续写在这个this前面会有问题吗

```
package org.lingfeng.web;

/**
 * Created by lingfeng on 2020/4/11.
 */
public class ExtendsChild extends ExtendsFather{

  public String name = "child li";

  public Integer age = 10;


  public ExtendsChild() {
    super();
    this(this.name,this.age);
  }

  public ExtendsChild(String name, Integer age) {
    this.name = name;
    this.age = age;
  }

  @Override
  void washDishes() {
    super.washDishes();
    System.out.println("我接着刷碗");
  }
}

```
&nbsp;&nbsp;这个问题大家可以先考虑一下，暂时别急着去网上百度答案。等到继承封装多态都讲完了。我会对这三个里面的一些思考点进行一次总结
  

## 总结
1. 继承在父类指针指向子类实现类的时候，调用同名方法会使用子类的方法。如果子类没有重写这个父类方法的话。会调用父类的方法
2. java类单根继承是为了解决多个父类同名方法在调用时的选择困难。而接口只是定义，多个定义，其实并不影响调用的选择
3. super只是一个关键字。super()是代表调用父类的无参构造。this代表的是当前对象的指针引用。
4. super()不用显式的去写明。java会在子类的构造方法中隐式的生产这段代码。
5. java继承可以在子类重写父类方法的时候，调用父类的代码逻辑.是通过super.方法名来实现的。super指向子类构造方法中的super()生成的父类指针。

—— LingFeng 后记于 2020.04.11