---
layout:     post
title:      "Java集合框架源码解读(4)——WeakHashMap"
subtitle:   ""
date:       2017-08-29 23:30:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Java基础
---

# 概述

**WeakHashMap**也是Map接口的一个实现类，它与HashMap相似，也是一个哈希表，存储key-value pair，而且也是非线程安全的。不过WeakHashMap并没有引入红黑树来尽量规避哈希冲突带来的影响，内部实现只是**数组+单链表**。此外，WeakHashMap与HashMap**最大的不同之处**在于，WeakHashMap的key是**“弱键”（weak keys）**，即**当一个key不再正常使用时，key对应的key-value pair将自动从WeakHashMap中删除，在这种情况下，即使key对应的key-value pair的存在，这个key依然会被GC回收，如此以来，它对应的key-value pair也就被从map中有效地删除了。**

# Java的四种引用

在正式进入WeakHashMap源码之前，我们需要先对**“弱引用”**有一个基本的认识，为此这里先介绍一下JDK 1.2开始推出的四种引用：
+ **强引用（Strong Reference）**
强引用是指在程序代码之中普遍存在的，类似`Objective obj = new Object()`这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象。
+ **软引用（Soft Reference）**
软引用是用来描述一些还有用但并非必需的对象，对于软引用关联着的对象，**在系统将要发生内存溢出异常之前**，将会把这些对象列进回收范围之中进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。在JDK 1.2之后，提供了**SoftReference**类来实现软引用。
+ **弱引用（Weak Reference）**
弱引用也是用来描述非必需对象的，但是它的强度比软引用更弱一点，**被弱引用关联的对象只能生存到下一次垃圾收集发生之前**。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。在JDK 1.2之后，提供了**WeakReference**类来实现弱引用。
+ **虚引用（PhantomReference）**
虚引用也称为幽灵引用或者幻影引用，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的**唯一目的**就是**能在这个对象被收集器回收时收到一个系统通知**。在JDK 1.2之后，提供了**PhantomReference**类来实现虚引用。

我们说WeakHashMap的key是weak-keys，即是说这个Map实现类的key值都是弱引用。

# 底层实现

先来看一下WeakHashMap的定义：
```java
public class WeakHashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V> {
```
与HashMap一样继承自AbstractMap类，并特地标注了Map接口，另外，WeakHashMap并没有实现Cloneable接口和Serializable接口。

再来看一下WeakHashMap的重要静态内部类**Entry**：
```java
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
    V value;
    final int hash;
    Entry<K,V> next;

   /**
     * Creates new entry.
     */
    Entry(Object key, V value,
          ReferenceQueue<Object> queue,
          int hash, Entry<K,V> next) {
        //把key传给父类WeakReference的构造函数，说明Entry的key是弱引用
        //同时key值会进入引用队列queue中，等待被处理
        super(key, queue);
        //显示定义了value，说明了Entry的value是强引用
        this.value = value;
        this.hash  = hash;
        this.next  = next;
    }

    @SuppressWarnings("unchecked")
    public K getKey() {//在获取key时需要unmaskNull，因为对于null的key，是用WeakHashMap的内部成员属性来表示的
        return (K) WeakHashMap.unmaskNull(get());
    }

    public V getValue() {
        return value;
    }

    public V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>)o;
        K k1 = getKey();
        Object k2 = e.getKey();
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
            V v1 = getValue();
            Object v2 = e.getValue();
            if (v1 == v2 || (v1 != null && v1.equals(v2)))
                return true;
        }
        return false;
    }

    public int hashCode() {
        K k = getKey();
        V v = getValue();
        return Objects.hashCode(k) ^ Objects.hashCode(v);
    }

    public String toString() {
        return getKey() + "=" + getValue();
    }
}
```
这里用到了`unmaskNull()`方法，它的作用是用一个空的Object对象来表示null值得key。其源码如下：
```java
private static final Object NULL_KEY = new Object();
/**
 * 当key为null时，使用NULL_KEY表示key
 */
private static Object maskNull(Object key) {
    return (key == null) ? NULL_KEY : key;
}
```
可以看到Entry<K,V>继承自WeakReference类，那么对于Entry类中需要定义为弱引用的字段，直接传入父类的构造函数即可，如代码中看到的key，这也实现了我们前面说的**“弱键”**。而value值则赋予了强引用，但这并不影响，我们在稍后会介绍一个`WeakHashMap.expungeStaleEntries`方法，该方法会把弱键对应的key-value整个赋为null，以帮助GC将其回收。

再来看一下WeakHashMap的重要字段：
```java
  /**
   * 存储键值对的数组，一般是2的幂
   */
    Entry<K,V>[] table;
   /**
    * 键值对的实际个数
    */
    private int size;
   /**
    * 扩容的临界值，通过capacity * load factor可以计算出来。超过这个值HashMap将进行扩容
    * @serial
    */
    private int threshold;
   /**
    * 负载因子
    */ 
    private final float loadFactor;
   /**
    * 引用队列，用于保存会被GC回收的弱键
    */
    private final ReferenceQueue<Object> queue = new ReferenceQueue<>();
   /**
    * 记录HashMap被修改结构的次数。
    * 修改包括改变键值对的个数或者修改内部结构，比如rehash
    * 这个域被用作HashMap的迭代器的fail-fast机制中（参考ConcurrentModificationException）
    */ 
    int modCount;
```
与HashMap相比，WeakHashMap少了entrySet字段，而多了一个引用队列queue。

WeakHashMap定义的静态全局变量如下：
```java
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    private static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The load factor used when none specified in constructor.
     */
    private static final float DEFAULT_LOAD_FACTOR = 0.75f;
```
与HashMap相比，缺少了**TREEIFY_THRESHOLD**、**UNTREEIFY_THRESHOLD**、**MIN_TREEIFY_CAPACITY**三个静态全局变量，它们是用来进行单链表和红黑树转换的，在WeakHashMap中没有实现红黑树，因此不需要这三个变量。

下面再来看一下与弱键实现有关的重要方法`expungeStaleEntries`：
```java
/**
 * 从哈希表中删除被回收的引用
 */
private void expungeStaleEntries() {
    for (Object x; (x = queue.poll()) != null; ) {
        synchronized (queue) {
            @SuppressWarnings("unchecked")
                Entry<K,V> e = (Entry<K,V>) x;//e为要清理的Entry
            int i = indexFor(e.hash, table.length);
            Entry<K,V> prev = table[i];
            Entry<K,V> p = prev;
            //遍历碰撞链表
            while (p != null) {
                Entry<K,V> next = p.next;
                if (p == e) {
                    if (prev == e)
                        table[i] = next;
                    else
                        prev.next = next;
                    // Must not null out e.next;
                    // stale entries may be in use by a HashIterator
                    e.value = null; // 把value赋值为null，帮助GC回收强引用的value
                    size--;
                    break;
                }
                prev = p;
                p = next;
            }
        }
    }
}
```
从WeakHashMap.Entry的源码中我们看到，weak keys的值会被保存到引用队列中，该方法就说将引用队列中保存的弱键对应的Entry从单链表中删除，即**删除哈希表中被GC回收了的键值对**。在WeakHashMap定义的增、删、改、查方法中，都要调用该方法。

# 应用场景

#### 缓存

缓存是**内存泄漏**的一个常见来源，一旦你把对象引用放到缓存中，它就很容易被遗忘掉，从而使得它不再有用之后很长一段时间内仍然留在缓存中。

对于这个问题，有几种可能的解决方案。如果你正好要实现这样的缓存：只要在缓存之外存在对某个项的键的引用，该项就有意义，那么就可以用WeakHashMap代表缓存。当缓存中的项过期之后，它们就会自动被删除（要注意的是，只有当**所要的缓存项的生命周期是由该键的外部引用而不是由值决定**时，WeakHashMap才有用处）。

Tomcat就用WeakHashMap实现了它的并发缓存[**ConcurrentCache**](https://github.com/apache/tomcat/blob/trunk/java/org/apache/tomcat/util/collections/ConcurrentCache.java)，源码如下：
```java
package org.apache.tomcat.util.collections;

import java.util.Map;
import java.util.WeakHashMap;
import java.util.concurrent.ConcurrentHashMap;

public final class ConcurrentCache<K,V> {

    private final int size;

    private final Map<K,V> eden;

    private final Map<K,V> longterm;

    public ConcurrentCache(int size) {
        this.size = size;
        this.eden = new ConcurrentHashMap<>(size);
        this.longterm = new WeakHashMap<>(size);
    }

    public V get(K k) {
        V v = this.eden.get(k);
        if (v == null) {
            synchronized (longterm) {
                v = this.longterm.get(k);
            }
            if (v != null) {
                this.eden.put(k, v);
            }
        }
        return v;
    }

    public void put(K k, V v) {
        if (this.eden.size() >= size) {
            synchronized (longterm) {
                this.longterm.putAll(this.eden);
            }
            this.eden.clear();
        }
        this.eden.put(k, v);
    }
}
```

#### 监听器和其他回调

内存泄漏的另一个常见来源是监听器和其他回调。如果你在实现的是客户端注册回调却没有显式地取消注册的API，除非你采取某些动作，否则它们就会积聚。确保回调立即被当作垃圾回收的最佳方法是只保存它们的**弱引用（weak reference）**，可以只将它们保存成WeakHashMap中的键。

