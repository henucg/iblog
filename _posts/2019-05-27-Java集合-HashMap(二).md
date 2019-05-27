---
layout: post
title:  "Java集合-Hashmap（二）"
date:   2019-05-27 11:35:00 +0800
categories: 博客
tags: Java集合
excerpt: "Java1.8中HashMap的实现方式"
---

## 1、概述
+ Java在1.8之前与1.8之后Hashmap的实现方式有比较大的区别，主要是使用了红黑树，这篇文章中将根据1.8版本解析部分Hashmap的源码

## 2、HashMap简介(1.8)
+ HashMap是存储对象Node是一个键值对（key-value），key与value都可以为null
+ Node对象和原来一样有四个属性：hash,key,value,next

## 3、HashMap数据结构
![]({{site.url}}/assets/20190527_01/0.png){:width="80%",height="80%"}
+ Hashmap是一个散列表，数组每个位置存储的都是一个Node，每次存储数据的时候都要先计算key的hash值，然后根据Hash值计算数组下标
但是JDK1.8之后计算数组下标不再是简单的Hash值与数组长度-1做与运算，而是使用了无符号右移，这样做的目的是为了增加Hash的分散性
+ 与JDK1.7最大的变化是1.8的HashMap使用了数组+链表+红黑树的数据结构
+ 当数据产生Hash冲突之后，依旧和原来一样，采用的是Hash链的方式解决冲突，不一样的是当链表的长度大于8的时候，链表变形为一棵红黑树
当链表的长度小于6的时候，红黑树再次变形为链表

## 4、HashMap部分原码解析
#### 1.初始参数
```java
// 数组默认容量大小，改成位操作
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

// 数组最大值
static final int MAXIMUM_CAPACITY = 1 << 30;

// 加载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 链表长度大于8，转变为红黑树
static final int TREEIFY_THRESHOLD = 8;

// 红黑树元素个数小于6，转变为链表
static final int UNTREEIFY_THRESHOLD = 6;

/*
* 在转变成树之前，还会有一次判断，只有键值对数量大于 64 才会发生转换。这是为了避免在哈希表建立初期，
* 多个键值对恰好被放入了同一个链表中而导致不必要的转化
*/ 
static final int MIN_TREEIFY_CAPACITY = 64;
```

#### 2.实体类
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
+ 实体类与1.7大部分一致

#### 3.hash
```java
static final int hash(Object key) {
    int h;
    // 高位参与运算，hash分布更加均匀，减少冲突
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

#### 4.tableSizeFor
```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
+ 返回最近的不小于输入参数的2的整数次幂,比如10，则返回16

#### 5.put
```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```
+ 调用了putVal方法

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table未初始化或者长度为0，进行扩容，n为桶的个数，即数组的长度
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 确定在桶中的位置，并检查有没有数据，如果没有，新生成一个节点放入桶中
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 如果有数据
    else {
        Node<K,V> e; K k;
        // 比较桶中的第一个位置的hash与key，如果相同，将这个数据赋值给e
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果不等，判断是否为树类型
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 如果不是树，遍历链表
            for (int binCount = 0; ; ++binCount) {
                // 到达链表尾部
                if ((e = p.next) == null) {
                    // 在链表尾部插入新节点，1.7中是在链表的头部插入
                    p.next = newNode(hash, key, value, null);
                    // 当节点数据大于阈值，调用treeifyBin进一步判断是否转变为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 判断key是否存在
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 如果key已经存在
        if (e != null) { // existing mapping for key
            V oldValue = e.value;

            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // 访问后回调函数，不做操作
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 记录操作次数
    ++modCount;
    // 记录数据个数，同时判断是否扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

#### 6.resize
```java
final Node<K,V>[] resize() {
    // 保留当前数组
    Node<K,V>[] oldTab = table;
    // 当前数组大小
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 当前阈值
    int oldThr = threshold;
    // 新数组大小、新阈值
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 如果数组大小已经大于最大值，不能在扩充了，直接返回阈值为Integer的最大值
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 否则，新数组大小扩大为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 阈值也扩大为原来的2倍
            newThr = oldThr << 1; // double threshold
    }
    // 初始容量已存在threshold中
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    // 使用缺省值（使用默认构造函数初始化）    
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 重新计算阈值
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    // 初始化table    
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 只有一个节点，重新计算hash值与数组下标，添加数据
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 如果原本就是红黑树，根据(e.hash & oldCap)分为两个，如果哪个数目不大于UNTREEIFY_THRESHOLD，就转为链表
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    // 将同一桶中的元素根据(e.hash & oldCap)是否为0进行分割成两个不同的链表，完成rehash
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
                    // hash到高部分即原索引+oldCap
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

#### 7.treeifyBin
```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 若数组容量小于MIN_TREEIFY_CAPACITY，不转换而进行resize操作
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            // 为每一个节点构建一个树，一次拼接起来
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                // 构建每个节点的前置节点与后置节点
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            // 重新排序成位红黑树，此方法在此不做介绍，感兴趣可以去了解一下红黑树的数据结构
            hd.treeify(tab);
    }
}
```

#### 8.get
```java
public V get(Object key) {
    Node<K,V> e;
    // 调用getNode方法
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```
+ getNode的源码


```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 比较第一个元素，相同返回
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // 判断是否是红黑树
            if (first instanceof TreeNode)
                // 查找红黑树
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 不是红黑树遍历链表
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

## 5.HashMap在JDK1.8前后区别
+ 添加数据的时候，JDK1.7用的是头插法，而JDK1.8及之后使用的都是尾插法，因为JDK1.7是用单链表进行的纵向延伸，当采用头插法就是能够提高插入的效率，
但是也会容易出现逆序且环形链表死循环问题。但是在JDK1.8之后是因为加入了红黑树使用尾插法，能够避免出现逆序且链表死循环的问题
+ 在JDK1.7的时候是直接用hash值和需要扩容的二进制数进行&（这里就是为什么扩容的时候为啥一定必须是2的多少次幂的原因所在，因为如果只有2的n次幂的情况时最后一位二进制数才一定是1，这样能最大程度减少hash碰撞）（hash值 & length-1） 
+ 而在JDK1.8的时候直接用了JDK1.7的时候计算的规律，也就是扩容前的原始位置+扩容的大小值=JDK1.8的计算方式，而不再是JDK1.7的那种异或的方法。但是这种方式就相当于只需要判断Hash值的新增参与运算的位是0还是1就直接迅速计算出了扩容后的储存方式
+ JDK1.7的时候使用的是数组+ 单链表的数据结构。但是在JDK1.8及之后时，使用的是数组+链表+红黑树的数据结构（当链表的深度达到8的时候，也就是默认阈值，就会自动扩容把链表转成红黑树的数据结构来把时间复杂度从O（N）变成O（logN）提高了效率）

## 参考
+ [https://blog.csdn.net/qazwyc/article/details/76686915](https://blog.csdn.net/qazwyc/article/details/76686915){:target="_blank"}
+ [https://www.jianshu.com/p/ee0de4c99f87](https://www.jianshu.com/p/ee0de4c99f87){:target="_blank"}