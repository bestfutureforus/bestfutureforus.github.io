---
layout:     post
title:      "Java集合框架源码解读(2)——HashMap"
subtitle:   ""
date:       2017-08-27 21:30:00
author:     "Crow"
#header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Java基础
---

在Java Collections Framework的体系中中，主要有两个重要的接口，一个是List、Set和Queue所属的Collection，还有一个就是Map接口了。在上一篇文章中介绍了List接口，它适用于按数值索引访问元素的情形。本文中将介绍的Map则提供了一个更通用的元素存储方法。

Map 集合类用于存储**元素对（称作“键”和“值”）**也叫**键值对（key/value pair）**，其中每个键映射到一个值。从概念上而言，你可以将 List 看作是具有数值键的 Map。Map接口规定key值是不能重复的，而value值可以重复。

Map接口有三种重要的具体实现类——**HashMap**、**WeakHashMap**和**TreeMap**，其中**HashMap**还有一个重要的子类**LinkedHashMap**，它们都是非线程安全的类，本文将通过分析源码重点介绍HashMap类，关于另外几个类的内容则留到后续文章再讲。

Map接口的架构如下图所示：
<center><img src="http://pic.yupoo.com/crowhawk/02033bd6/a082d0f1.png"></center>

在图中可以看到，Map接口还有一个叫做HashTable的实现类，它是JDK早期的产物，与HashMap实现基本相似，不过是它是线程安全的，由于该容器已经过时，现在基本被弃用，因此在系列文章中就不多加笔墨去介绍了。

# 概述

HashMap是基于哈希表实现的，HashMap的每一个元素是一个key-value对，其内部通过**单链表**和**红黑树**解决冲突问题，容量不足时会自动扩容。

HashMap是非线程安全的，只适用于单线程环境下，多线程环境下可以采用Concurrent并发包下的**ConcurrentHashMap**。

# 哈希冲突

对于每个对象 X 和 Y，如果当且仅当 X.equals(Y) 为 false，使得 X.hashCode()!= Y.hashCode() 为 true，这样的函数叫做**完美 Hash 函数**。当哈希函数对两个不同的数据项产生了相同的hash值时，这就称为**哈希冲突**。

基于对象中变化的字段，我们可以很容易地构造一个完美哈希函数，但是这需要无限的内存大小，这种假设显然是不可能的。而且，即使我们能够为每个 POJO（Plain Ordinary Java Object）或者 String 对象构造一个理论上不会有冲突的哈希函数，但是 hashCode() 函数的返回值是 int 型。根据**鸽笼理论**，当我们的对象超过 232 个时，这些对象会发生哈希冲突。

因此，实现HashMap的一个重要考量，就是**尽可能地避免哈希冲突**。HashMap在JDK 1.8中的做法是，用**链表**和**红黑树**存储**相同hash值的value**。当Hash冲突的个数比较少时，使用链表，否则使用红黑树。

# 底层实现

HashMap实现的接口如下：
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```
HashMap继承自抽象类AbstractMap，实现了Map接口，AbstractMap类实现了Map接口的部分方法，因此Map的最终实现类直接继承AbstractMap，可以减少很多工作量。

先来看HashMap内部两个重要的**静态内部类**。

单向链表的节点**Node**
```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```
Node实现了Map的内部接口Entry，Entry接口定义了**键值对（key-value pair）**的基本操作，Node类提供了这些方法的实现并且还含有一个next引用，作为**单链表**的实现用来指向下一个Node。

红黑树的节点**TreeNode**：
```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }

    /**
     * Returns root of tree containing this node.
     */
    final TreeNode<K,V> root() {
        for (TreeNode<K,V> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
    ……
}
```
**当一个单链表冲突的结点数超过预设值时，将会把这个单链表自动调整为红黑树。**这样做的好处是，最坏的情况下即所有的key都Hash冲突，采用链表的话查找时间为O(n),而采用红黑树为O(logn)。

HashMap的几个重要**字段**如下：

```java
    //存储数据的Node数组，长度是2的幂。
    transient Node<K,V>[] table;
    //键值对缓存，它们的映射关系集合保存在entrySet中。即使Key在外部修改导致hashCode变化，缓存中还可以找到映射关系
    transient Set<Map.Entry<K,V>> entrySet;
    //map中保存的键值对的数量
    transient int size;
    //map结构被改变的次数
    transient int modCount;
    //需要调整大小的极限值（容量*装载因子）
    int threshold;
    //装载因子，在后面会进行详细介绍
    final float loadFactor;
```

HashMap内部使用Node数组实现了一个哈希桶数组table。可以看出，**HashMap**还是**凭借数组实现**的，**数组的元素是单链表或红黑树，对于key的hash值相等的key-value pair，它们将分别作为一个结点（Node或TreeNode）存储在同一个单链表或红黑树中**。我们知道数组的特点：寻址容易，插入和删除困难，而链表的特点是：寻址困难，插入和删除容易，红黑树则对插入时间、删除时间和查找时间提供了最好可能的最坏情况担保。HashpMap将这三者结合在一起。

HashMap的数据结构如下图所示：

<center><img src="https://pic.yupoo.com/crowhawk/82b34b3c/07aa2f3a.png"></center>

此外，这里的**modCount**属性，记录了map结构被改变的次数，它与**"fail-fast"**机制的实现息息相关。fail-fast机制是Java集合的一种错误检测机制，假设存在两个线程（线程1、线程2），线程1通过Iterator在遍历集合A中的元素，在某个时候线程2修改了集合A的结构（是**结构上面的修改**，而不是简单的修改集合元素的内容），那么这个时候程序就会抛出 ConcurrentModificationException 异常，从而产生fail-fast机制。

对于HashMap内容的修改都将使modCount的值增加，在迭代器初始化过程中会将这个值赋给迭代器的**expectedModCount**,在迭代过程中，判断modCount跟expectedModCount是否相等，如果不相等就表示已经有其他线程修改了Map。


HashMap的一些重要的**静态全局变量**如下，它们与HashMap规避哈希碰撞的策略息息相关：

```java
    /**
     * table默认的初始容量，它的值必须是2的整数幂
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * table的最大容量，必须小于2的30次方，如果传入的容量大于这个值，将被替换为该值
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认装载因子，如果在构造函数中不显式指定装载因子，则默认使用该值。
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 结点冲突数达到8时，就会对哈希表进行调整，如果table容量小于64，那么会进行扩容，
     * 如果不小于64，那么会将冲突数达到8的那个单链表调整为红黑树.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * 如果原先就是红黑树，resize以后冲突结点数少于6了，就把红黑色恢复成单链表
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * 如果table的容量少于64，那么即使冲突结点数达到TREEIFY_THRESHOLD后不会把该单链表调整成红黑数，而是将table扩容
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
```

HashMap使用的**hash算法**如下：
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

使用hash值的高位16位与低16进行XORs操作，算法简洁有效。

# 常用API

看完了HashMap的基本数据结构以后，来看一下常用方法的源码，首先自然想到的是`get(key)`和`put(key,value)`。

#### get(key)

`get(key)`方法的作用是的源码如下：

```java
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```
我们将要查找的key值传给get，它调用`hashCode`计算hash从而得到bucket位置，并进一步调用`equals()`方法确定键值对。取模算法中的除法运算效率很低，在HashMap中通过**h & (n-1)**替代取模，得到所在数组位置，效率会高很多（前提是保证数组的容量是2的整数倍）。

#### resize()

在介绍`put`方法之前还要先来看一下`resize()`方法，
```java
   final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```
当HashMap中的元素个数超过 **数组大小 * loadFactor** 时，就会进行数组扩容，loadFactor的默认值为0.75，这是一个折中的取值。也就是说，默认情况下，数组大小为16，那么当HashMap中元素个数超过 **16 * 0.75=12** 的时候，就把数组的大小扩展为 **2 * 16=32** ，即扩大一倍，然后重新计算每个元素在数组中的位置，而这是一个非常消耗性能的操作，所以如果我们已经预知HashMap中元素的个数，那么预设元素的个数能够有效的提高HashMap的性能。

#### put(key,value)

`put(key,value)`方法的作用是向HashMap中添加一对key-value pair。源码如下：

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * Implements Map.put and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
将key-value pair传给`put`方法时，它调用`hashCode`计算hash从而得到bucket位置，进而，**HashMap根据当前bucket的占用情况自动调整容量(超过Load Factor则resize为原来的2倍)**。如果没有发生碰撞就直接放到bucket里，如果发生碰撞，Hashmap先通过链表将产生碰撞冲突的元素组织起来，如果一个bucket中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。
