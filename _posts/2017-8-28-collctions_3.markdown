---
layout:     post
title:      "Java集合框架源码解读(3)——LinkedHashMap"
subtitle:   ""
date:       2017-08-28 19:06:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Java基础
---

# 概述

**LinkedHashMap**是HashMap的子类，它的大部分实现与HashMap相同，两者最大的区别在于，HashMap的对哈希表进行迭代时是无序的，而**LinkedHashMap对哈希表迭代是有序的**，LinkedHashMap默认的规则是，迭代输出的结果保持和插入key-value pair的顺序一致（当然具体迭代规则可以修改）。LinkedHashMap除了像HashMap一样用数组、单链表和红黑树来组织数据外，还额外维护了一个**双向链表**，每次向linkedHashMap插入键值对，除了将其插入到哈希表的对应位置之外，还要将其插入到双向循环链表的尾部。

# 底层实现

先来看一下LinekedHashMap的定义：
```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
```
除了继承自HashMap以外并无太多特殊之处，这里特地标注实现了Map接口应该也只是为了醒目。

大家最关心的应该是LinkedHashMap如何实现有序迭代，下面将逐步通过源码来解答这一问题。

先看一下一个重要的静态内部类**Entry**：
```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```
该类继承自HashMap的Node内部类，前面已经介绍过，Node是一个单链表结构，这里Entry添加了前继引用和后继引用，则是一个**双向链表**的节点。在双向链表中，每个节点可以记录自己前后插入的节点信息，以维持有序性，这也是LinkedHashMap实现有序迭代的关键。

#### 按插入顺序有序和按访问顺序有序

<big>**按插入有序**</big>

按插入有序即先添加的在前面，后添加的在后面，修改操作不影响顺序。以如下代码为例：
```java
Map<String,Integer> seqMap = new LinkedHashMap<>();

seqMap.put("c", 100);
seqMap.put("d", 200);
seqMap.put("a", 500);
seqMap.put("d", 300);

for(Entry<String,Integer> entry : seqMap.entrySet()){
    System.out.println(entry.getKey()+" "+entry.getValue());
}
```
运行结果是：
```
c 100
d 300
a 500
```
可以看到，键是按照"c", "d", "a"的顺序插入的，修改"d"的值不会修改顺序。

<big>**按访问有序**</big>

按访问有序是，序列末尾存放的是最近访问的key-value pair，每次访问一个key-value pair后，就会将其移动到末尾。
```java
Map<String,Integer> accessMap = new LinkedHashMap<>(16, 0.75f, true);

accessMap.put("c", 100);
accessMap.put("d", 200);
accessMap.put("a", 500);
accessMap.get("c");
accessMap.put("d", 300);

for(Entry<String,Integer> entry : accessMap.entrySet()){
    System.out.println(entry.getKey()+" "+entry.getValue());
}
```
运行结果为：
```
a 500
c 100
d 300
```
针对不同的应用场景，LinkedHashMap可以在这两种排序方式中进行抉择。

---

LinkedHashMap定义了三个重要的**字段**：
```java
//双链表的头节点
transient LinkedHashMap.Entry<K,V> head;
//双链表的尾节点
transient LinkedHashMap.Entry<K,V> tail;
/**
 * 这个字段表示哈希表的迭代顺序
 * true表示按访问顺序迭代
 * false表示按插入顺序迭代
 * LinkedHashMap的构造函数均将该值设为false，因此默认为false
 */
final boolean accessOrder;
```
关于它们的具体作用已在注释中标出。

LinkedHashMap有五个构造方法，其中有一个可以指定**accessOrder**的值：
```java
public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

# 重要方法

在HashMap中定义了几个**“钩子”**方法（关于钩子的详细内容，请参考笔者的博客[设计模式(9)——模板方法模式](https://crowhawk.github.io/2017/07/21/designpattern_9_template/)），这里特地列出其中的三个：
+ `afterNodeRemoval(e)`
+ `afterNodeInsertion`
+ `afterNodeInsertion`
 
它们与迭代有序性的实现息息相关。

此外还有两个重要的API`get`和`containsValue`，这里也分析一下它们的源码实现，至于`put`方法，LinkedHashMap并没有覆写该方法，因此其实现与HashMap相同。

#### afterNodeRemoval(e)方法
```java
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```
在HashMap的`removeNode`方法中调用了该钩子方法，对于LinkedHashMap，在执行完对哈希桶中单链表或红黑树节点的删除操作后，还需要调用该方法将双向链表中对应的Entry删除。

#### afterNodeInsertion方法
```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)){
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```
在HashMap的`putVal`方法中调用了该方法，可以看出，在判断条件成立的情况下，该方法会删除双链表中的头节点（当然是在哈希桶和双向链表中同步删除该节点）。判断条件涉及了一个`removeEldestEntry(first)`方法，它的源码如下：

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```
可以看到，它默认是返回false的，即不删除头节点。如果需要定义是否需要删除头节点的规则，只需覆盖该方法并提供相关实现即可。该方法的作用在于，它**提供了当一个新的entry被添加到linkedHashMap中，删除头节点的机会。**这是非常有意义的，可以通过删除头节点来减少内存消耗，避免内存溢出。

#### afterNodeAccess(e)方法
```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```
该方法在HashMap的`putVal`方法、LinkedHashMap的`get`方法中都被调用，它的作用是：**如果accessOrder返回值为true（即按照访问顺序迭代），则将最近访问的节点调整至双向队列的队尾，这也就保证了按照访问顺序迭代时Entry的有序性。**

#### get(key)方法

```java
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}
```
该方法增加了按访问顺序或插入顺序进行排序的选择功能，会根据AccessOrder的值调整双向链表中节点的顺序，获取节点的过程与HashMap中一致。

####  containsValue(value)方法

```java
public boolean containsValue(Object value) {
    for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
        V v = e.value;
        if (v == value || (value != null && value.equals(v)))
            return true;
    }
    return false;
}
```
由于LinkedHashMap维护了一个双向链表，因此它的`containsValue(value)`方法直接遍历双向链表查找对应的Entry即可，而无需去遍历哈希桶。

# LinkedHashMap与HashMap

LinkedHashMap是HashMap的子类，它们最大的区别是，HashMap的迭代是无序的，而**LinkedHashMap是有序的**，并且有**按插入顺序**和**按访问顺序**两种方式。为了实现有序迭代，LinkedHashMap相比HashMap，额外维护了一个双向链表，因此**一般情况下，遍历HashMap比LinkedHashMap效率要高**，在没有按序访问key-value pair的情况下，一般建议使用HashMap（当然也有例外，当**HashMap容量很大，实际数据较少**时，遍历起来可能会比 LinkedHashMap慢，因为LinkedHashMap的遍历速度只和实际数据有关，和容量无关，而HashMap的遍历速度和他的容量有关）。

此外，在HashMap与LinkedHashMap的设计中，多处体现了设计模式的精妙，在设计迭代器时，用到了[迭代器模式](https://crowhawk.github.io/2017/07/21/designpattern_10_iterator/)，在设计它们的继承关系时，又用到了[模板方法模式](https://crowhawk.github.io/2017/07/21/designpattern_9_template/)。
