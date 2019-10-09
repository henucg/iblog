---
layout: post
title:  "Java集合-Hashmap（一）"
date:   2019-05-20 11:35:00 +0800
categories: 博客
tags: Java集合
excerpt: "Java1.7中HashMap的实现方式"
---

## 1、概述
+ Java在1.8之前与1.8之后Hashmap的实现方式有比较大的区别，这篇文章中将根据1.7版本解析Hashmap的实现

## 2、HashMap简介
+ HashMap是存储对象Entry是一个键值对（key-value），key与value都可以为null
+ key为null的Entry直接放在数组的第一个位置table[0]
+ Entry对象有四个属性：hash,key,value,next
+ HashMap继承了AbstractMap，实现了Map、Cloneable、Serializable接口
+ HashMap默认容量为16，加载因子默认为0.75，初始化时可以自己指定初始容量，但是容量大小会自动设置为2的指数次方
+ HashMap每次扩容都扩展到原来的2倍
+ HashMap是非线程安全的

## 3、HashMap数据结构
![]({{site.url}}/assets/20190520_01/0.png){:width="100%",height="100%"}
+ Hashmap是一个散列表，数组每个位置存储的都是一个Entry，每次存储数据的时候都要先计算key的hash值，然后用hash值对（数组长度-1）
做与运算，其效果就是对数组长度模运算取余操作，余数就是数据在数组的下标位置，因为不同的数据key值计算出来的hash值可能相同，所以会产
生不同的数据定位到数组的相同位置，我们称之为：Hash冲突。HashMap解决hash冲突的方法是“拉链法”，即当数据的hash值相同的时候，比较
key值是否相同，如果相同就替换对应的value，如果不同，就将新添加的数据放在数组，并将它的next指向原来的数据，形成一个单向链表

## 4、HashMap部分原码解析

#### 1.初始参数
```java
// 数组默认容量大小
static final int DEFAULT_INITIAL_CAPACITY = 16;

// 数组最大值
static final int MAXIMUM_CAPACITY = 1 << 30;

// 默认加载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 存储数据的对象数组
transient Entry[] table;

// 数据数量
transient int size;

// 扩容的上限，数组里面的数据达到这个上限就会扩容 (数组大小 * 加载因子).
int threshold;

// 加载因子
final float loadFactor;

// 操作次数
transient int modCount;
```

#### 2.构造函数 
```java
// 自定义容量大小和加载因子的构造方法
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    // 如果自定义最大容量大于数组规定的最大值，默认就是这个最大值：2^30
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);

    // Find a power of 2 >= initialCapacity
    int capacity = 1;
    while (capacity < initialCapacity)
    	// 如果小于自定义容量大小，数组大小一直向左位移，所以HashMap的大小一定是2的次方
        capacity <<= 1;

    this.loadFactor = loadFactor;
    // 计算扩容上限
    threshold = (int)(capacity * loadFactor);
    // 初始化数组
    table = new Entry[capacity];
    // 不做操作
    init();
}

// 自定义加载因子的构造方法
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 默认构造方法
public HashMap() {
	// 默认数组大小16，加载因子0.75
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
    table = new Entry[DEFAULT_INITIAL_CAPACITY];
    init();
}

// 包含子HashMap的构造方法
public HashMap(Map<? extends K, ? extends V> m) {
    this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                  DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
    putAllForCreate(m);
}
```

#### 3.数据节点Entry的数据结构
```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    final int hash;

    // h：hash值，k：key值，v：value值，n：下一个Entry
    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }

    public final K getKey() {
        return key;
    }

    public final V getValue() {
        return value;
    }

    // 新数据set成功之后，返回原来的value
    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    // 重写equals方法与hashCode方法
    public final boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry e = (Map.Entry)o;
        Object k1 = getKey();
        Object k2 = e.getKey();
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
            Object v1 = getValue();
            Object v2 = e.getValue();
            if (v1 == v2 || (v1 != null && v1.equals(v2)))
                return true;
        }
        return false;
    }

    public final int hashCode() {
        return (key==null   ? 0 : key.hashCode()) ^
               (value==null ? 0 : value.hashCode());
    }

    public final String toString() {
        return getKey() + "=" + getValue();
    }

    /**
     * This method is invoked whenever the value in an entry is
     * overwritten by an invocation of put(k,v) for a key k that's already
     * in the HashMap.
     */
    void recordAccess(HashMap<K,V> m) {
    }

    /**
     * This method is invoked whenever the entry is
     * removed from the table.
     */
    void recordRemoval(HashMap<K,V> m) {
    }
}
```
+ 我们可以看出 Entry 实际上就是一个单向链表，Entry 实现了Map.Entry 接口，即实现getKey(), getValue(), setValue(V value),
 equals(Object o), hashCode()这些函数，这些都是基本的读取/修改key、value值的函数。

#### 4.hash
```java
static int hash(int h) {
    // 位运算计算hash值
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

#### 4.indexFor
```java
static int indexFor(int h, int length) {
	// hash值和数组长度减1做与运算，结果等同于hash值对数组长度模运算取余
    return h & (length-1);
}
```

#### 5.size
```java
public int size() {
	// 每次新增size+1，每次删除size-1
    return size;
}
```

#### 6.isEmpty
```java
public boolean isEmpty() {
	// 根据size值判断HashMap是否为空
    return size == 0;
}
```

#### 7.get方法
```java
public V get(Object key) {
	// 如果key值为空，调用getForNullKey()方法
    if (key == null)
        return getForNullKey();
    int hash = hash(key.hashCode());
    // 如果key值不为空，先计算数组下标，然后对该下标的链表遍历
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
    return null;
}
```
+ 如果key值为空，调用getForNullKey()方法，这里看一下getForNullKey方法  

```java
private V getForNullKey() {
	// 取数组下标为0的链表，遍历
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null)
            return e.value;
    }
    return null;
}
```

#### 8.containsKey
```java
public boolean containsKey(Object key) {
    return getEntry(key) != null;
}
```
+ containsKey方法返回getEntry方法是否等于null 

```java
final Entry<K,V> getEntry(Object key) {
	// 计算hash值
    int hash = (key == null) ? 0 : hash(key.hashCode());
    // 计算数组下标，遍历对应的链表
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
    	// 即比较hash值又要比较key值，考虑了key=null的情况
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

#### 9.put方法
```java
public V put(K key, V value) {
	// 判断key是否为null
    if (key == null)
        return putForNullKey(value);
    // 计算hash值
    int hash = hash(key.hashCode());
    // 计算数组下标
    int i = indexFor(hash, table.length);
    // 便利链表
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        // 链表中存在这个key，替换原来的值，并且将原来的值返回
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    // 操作数+1
    modCount++;
    // 添加新的Entry
    addEntry(hash, key, value, i);
    return null;
}
```
+ key为null时的调用了putForNullKey方法

```java
private V putForNullKey(V value) {
	// 遍历数组下标等于0的链表，key=null如果已存在，替换原来的值并返回
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    // 添加新的Entry
    addEntry(0, null, value, 0);
    return null;
}
```
+ 只要是新增数组，都调用了addEntry方法

```java
// bucketIndex：数组下标
void addEntry(int hash, K key, V value, int bucketIndex) {
    // 将新增加的数组放在链表的第一个位置，并且next指向原来的链表
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    // size+1，同时判断是否达到扩容上限，达到的话，数组扩容为原来的2倍，调用resize方法
    if (size++ >= threshold)
        resize(2 * table.length);
}
```
+ resize方法实现了数组的扩容

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable);
    table = newTable;
    threshold = (int)(newCapacity * loadFactor);
}
```
+ resize方法先检查新的数组长度有没有超过规定的数组最大长度，然后新建一个长度为原来2倍的数组，实现了对数组的扩容，然后调用transfer方法
对原数组的数据重新计算数组下标，防止到对应新的链表当中，此操作比较耗时，所以在已知HashMap的数据量的时候，建议一开始就初始化HashMap的大
小，减少后续扩容次数

```java
void transfer(Entry[] newTable) {
    Entry[] src = table;
    int newCapacity = newTable.length;
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null;
            do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            } while (e != null);
        }
    }
}
```
#### 10.createEntry
```java
void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```
+ createEntry方法与addEntry方法相似，但是没有扩容机制，使用在已知不会扩容的场景下

#### 11.remove
```java
public V remove(Object key) {
    Entry<K,V> e = removeEntryForKey(key);
    return (e == null ? null : e.value);
}
```
+ remove调用removeEntryForKey方法执行删除操作

```java
final Entry<K,V> removeEntryForKey(Object key) {
    // 计算hash
    int hash = (key == null) ? 0 : hash(key.hashCode());
    // 计算数组下标
    int i = indexFor(hash, table.length);
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;

    // 遍历链表
    while (e != null) {
        Entry<K,V> next = e.next;
        Object k;
        // 存在对应的key值，删除当前Entry，并将前一个Entry的next指向下一个Entry
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;
            if (prev == e)
                table[i] = next;
            else
                prev.next = next;
            e.recordRemoval(this);
            return e;
        }
        prev = e;
        e = next;
    }

    return e;
}
```

#### 12.clear
```java
public void clear() {
    modCount++;
    Entry[] tab = table;
    // 数组每个值置为null
    for (int i = 0; i < tab.length; i++)
        tab[i] = null;
    // 	长度置为0
    size = 0;
}
```

#### 13.containsValue
```java
public boolean containsValue(Object value) {
    // 检查value是否为null
    if (value == null)
        return containsNullValue();

    // 没有key只能双层循环遍历，检查是否有相同的value，且value!=null
    Entry[] tab = table;
    for (int i = 0; i < tab.length ; i++)
        for (Entry e = tab[i] ; e != null ; e = e.next)
            if (value.equals(e.value))
                return true;
    return false;
}
```
+ value为null，调用containsNullValue方法处理

```java
private boolean containsNullValue() {
    Entry[] tab = table;
    // 双层循环遍历，检查是否有相同的value，且value==null
    for (int i = 0; i < tab.length ; i++)
        for (Entry e = tab[i] ; e != null ; e = e.next)
            if (e.value == null)
                return true;
    return false;
}
```

## 5、HashMap的遍历方式
#### 1.遍历HashMap的key-value对
```java
Iterator iterator = map.entrySet().iterator();
while(iterator.hasNext()) {
    Entry entry = (Entry)iterator.next();
    // 获取key
    String key = (String)entry.getKey();
    // 获取value
    String value = (String)entry.getValue();
}
```

#### 2.遍历HashMap的key
```java
Iterator iterator = map.keySet().iterator();
while (iterator.hasNext()) {
    // 获取key
    String key = (String)iterator.next();
    // 根据key，获取value
    String value = (String)map.get(key);
}
```

#### 3.遍历HashMap的value
```java
Collection c = map.values();
Iterator iterator = c.iterator();
while (iterator.hasNext()) {
    String value = (String)iterator.next();
}
```

## 6、对象equals与hashCode重写
+ 如果一个类需要放到hash散列表中使用的时候，如果业务需要重写了equals方法，那么必须重写hashCode方法
+ 原因是一个对象即使每个属性都相同的对象因为内存地址不同，所以计算的hashCode也不同，HashMap中的比较key是这样的，先求出key的hashcode，
比较其值是否相等，若相等再比较equals()，若相等则认为他们是相等的，若equals()不相等则认为他们不相等。如果只是重写了equals方法而没有重写
hashCode方法，则会导致业务上需要相同的对象在散列表中存取不一致，反之亦然，所以必定要两个方法一起重写

## 7、参考
+ https://www.cnblogs.com/skywang12345/p/3310835.html