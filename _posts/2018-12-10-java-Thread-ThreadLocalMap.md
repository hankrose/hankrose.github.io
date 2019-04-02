---
layout:     post
title:      "java多线程"
subtitle:   " \"java多线程十(ThreadLocalMap 的解析)\""
date:       2018-12-10 12:00:00
author:     "LingFeng"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - 多线程
---

> “Yeah It's on. ”


## 前言

我们上面分析道ThreadLocal 类，发现其实ThreadLocal这个类其实内部实现就是通过ThreadLocalMap来实现的。我们可以等同的默认为ThreadLocal做的事情其实和ThreadLocalMap差不多。
 
 
<p id = "build"></p>
---

## 正文
 
 
*	 1.ThreadLocalMap 构造函数
*    2.ThreadLocalMap get方法
*    3.ThreadLocalMap set方法

---

1.ThreadLocalMap 构造函数 
    了解一个java的类是怎么回事，我们首先要看这个类的构造方法是怎么来构造这个java对象的。 
	
	
```java 
/**
         * Construct a new map initially containing  (firstKey, firstValue).
         * ThreadLocalMaps are constructed lazily, so we  only create
         * one when we have at least one entry to put in  it.
         */
        ThreadLocalMap(ThreadLocal firstKey, Object  firstValue) {
            //类似于HashMap一样。这个操作基本一致，就是给table属性生成一个数组
            table = new Entry[INITIAL_CAPACITY];
            //初始化首位。
            int i = firstKey.threadLocalHashCode &  (INITIAL_CAPACITY - 1);
                        //开始赋值
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            //设置阈值，用于扩容 。可以参考HashMap
            setThreshold(INITIAL_CAPACITY);
        }
 
```
 
---

从上面这个来看。这个初始化其实和HashMap毫无区别。那么还有一个私有构造我们先看代码

---
 
```java
/**
         * Construct a new map including all Inheritable  ThreadLocals
         * from given parent map. Called only by  createInheritedMap.
         *
         * @param parentMap the map associated with parent  thread.
         */
        private ThreadLocalMap(ThreadLocalMap parentMap) {
            //传进来的ThreadLocalMap parentMap获取它的table
            Entry[] parentTable = parentMap.table;
            int len = parentTable.length;
                        //将传进来的LocalMap的副本覆盖目前的LocalMap
            setThreshold(len);
            table = new Entry[len];
                        //开始循环赋值
            for (int j = 0; j < len; j++) {
                Entry e = parentTable[j];
                if (e != null) {
                    ThreadLocal key = e.get();
                    if (key != null) {
                        Object value =  key.childValue(e.value);
                        Entry c = new Entry(key, value);
                        int h = key.threadLocalHashCode & (len  - 1);
                        while (table[h] != null)
                            h = nextIndex(h, len);
                        table[h] = c;
                        size++;
                    }
                }
            }
        }
 
```

---
   咦？它为什么要自己覆盖自己呢？我们来设想一下场景，比如我们在编辑一个word文档，但是编辑到一半了发现我们编辑错了，但是我们改回去好像不太容易。但是现在word给了我们一个副本快照，那么这个副本快找可以直接覆盖我们现在错的这个，那么我们可以重新从一开始的对的文档开始再次更新。从上面看到了我们的Entry 的get方法返回key是ThreadLocal ？
---
```java
ThreadLocal key = e.get();//what？ why？ 为什么？
````
---
别急我们去看看为什么会这样


```java
/**
         * The entries in this hash map extend  WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e.  entry.get()
         * == null) mean that the key is no longer  referenced, so the
         * entry can be expunged from table.  Such entries  are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends  WeakReference<ThreadLocal> {
            /** The value associated with this ThreadLocal.  */
            Object value;
            Entry(ThreadLocal k, Object v) {
                super(k);
                value = v;
            }
        }
```
---
 咦？ 我们看到了这个静态类 Entry 弱引用了ThreadLocal ，并且用ThreadLocal来作为key。 两个点  1 弱引用   ，2 key。
好，首先我们来了解什么是弱引用。弱引用是GC的时候用到的。java的垃圾回收器会在回收的时候直接回收只有被弱引用的对象。那么我们可以理解。这个entry会在ThreadLocal被回收的时候直接被gc。
第二个用这个ThreadLocal 作为key有什么作用呢。比如我A线程有一个ThreadLocalA，B线程有一个ThreadLocalB，那么用这个做为key的话，我们可以实现一个为每个线程的ThreadLocal都分配一个专属于它的ThreadLocalMap。

---
2.ThreadLocalMap get方法
   首先我们来看一下ThreadLocalMap的get的源码
   
   
```java

/**
         * Get the entry associated with key.  This method
         * itself handles only the fast path: a direct hit  of existing
         * key. It otherwise relays to getEntryAfterMiss.   This is
         * designed to maximize performance for direct  hits, in part
         * by making this method readily inlinable.
         *
         * @param  key the thread local object
         * @return the entry associated with key, or null  if no such
         */
        private Entry getEntry(ThreadLocal key) {
                        //根据ThreadLocal实例对象作为key，经过hash换算。得到数组的下表。
            int i = key.threadLocalHashCode & (table.length  - 1);
            //获取这个下标下的对象
            Entry e = table[i];
            //如果这个对象不为空那么返回
            if (e != null && e.get() == key)
                return e;
            else
                //如果为空那调用getEntryAfterMiss方法。
                return getEntryAfterMiss(key, i, e);
        }

```
---
首先我们为什么要用哪个ThreadLocal这个实例对象来作为一个key。我打个比方，比如有两个家庭是两个Thread。家里的存款和贷款都是不一样的。那么我们用一个ThreadLocal来代表存款，一个ThreadLocal来代表贷款。在里面维护数据。那么我们可以做到线程间数据的隔离，也可以做到同线程内数据的隔离，我们来看一下实验代码.

---

```java
package ThreadTest;
public class ThreadLocalTest {
     public static void main(String[] args) {
          Thread con = new ConnOne();
          con.start();
     }
}
class ConnOne extends Thread{
     ThreadLocal<String> money = new  ThreadLocal<String>();
     ThreadLocal<String> locan = new  ThreadLocal<String>();
     
     @Override
     public void run() {
          money.set("存款1w");
          locan.set("贷款1w");
          
          System.out.println(money.get());
          System.out.println(locan.get());
          super.run();
     }
}

运行结果：
存款1w
贷款1w
```

---

  从上面我们可以看到一个线程中可以有多个ThreadLocal ，所以用ThreadLocal的实例对象来代替值的话。这个可以快速的根据ThreadLocal 来获取entry的值。而不是网上很多的说是通过ThreadLocal的name来获取。
  
---

3.ThreadLocalMap set方法
    还是老规矩，先看源码。咋回事，再解析
```java
/**
         * Set the value associated with key.
         *
         * @param key the thread local object
         * @param value the value to be set
         */
        private void set(ThreadLocal key, Object value) {
            // We don't use a fast path as with get()  because it is at
            // least as common to use set() to create new  entries as
            // it is to replace existing ones, in which  case, a fast
            // path would fail more often than not.
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal k = e.get();
                if (k == key) {
                    e.value = value;
                    return;
                }
                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }
            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
```

从上面可以看到除了用ThreadLocal的实现作为key其他的和hashMap没啥区别。


总结
    ThreadLocalMap是ThreadLocal内部静态实现类，它的key是以ThreadLocal的实现对象为key的。它的作用是可以为一个Thread中的一个ThreadLocal保存内存副本。并且每个线程的数据都是隔离的。

—— LingFeng 后记于 2018.12.10