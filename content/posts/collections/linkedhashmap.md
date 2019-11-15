---
title: "Collections-Linkedhashmap"
date: 2019-08-15T13:19:18+08:00
draft: false
categories: ["CoreJava"]
tags: ["collections"]
---

<!--more-->

> 参考blog:[田小波的blog](https://www.tianxiaobo.com/2018/01/24/LinkedHashMap-%E6%BA%90%E7%A0%81%E8%AF%A6%E7%BB%86%E5%88%86%E6%9E%90%EF%BC%88JDK1-8%EF%BC%89/)

## 介绍
LinkedHashMap 继承自 HashMap，在 HashMap 基础上，通过维护一条`双向链表`，解决了 HashMap 不能随时保持遍历顺序和插入顺序一致的问题。除此之外，LinkedHashMap 对访问顺序也提供了相关支持。在一些场景下，该特性很有用，比如缓存。在实现上，LinkedHashMap 很多方法直接继承自 HashMap，仅为维护双向链表覆写了部分方法。还提供了一些增删改查操作时的回调方法.


## 源码介绍
### Entry
LinkedHashMap的节点 LinkedHashMap.Entry继承了hashMap的Node 增加了before和after两个节点,用于维护链表有序
```java
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

### 变量
```java
    /**
     * 头结点
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * 尾节点 
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;

    /* 设置链表排序规则: true 按照访问顺序排序  false按照插入顺序排序*/
    final boolean accessOrder;
```

### 构造参数
LinkedHashMap基本都是复用的的hashmap的构造方法,只是对`accessOrder`初始化
```java
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }

    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }

    public LinkedHashMap() {
        super();
        accessOrder = false;
    }

    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super();
        accessOrder = false;
        putMapEntries(m, false);
    }

    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }

```

### 基本操作
```java
    /* get操作:和hashmap逻辑一致,只是会判断是否根据访问顺序排序 */
    public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder) 
            afterNodeAccess(e);//访问后的操作,将节点放到最后
        return e.value;
    }

    /**
     * 将节点移到最后的逻辑很简单
     * 将该节点的before,after节点相连,并将该节点挂到last上
     * 如果该节点的before为头结点,则head=after 否则before.after = after 
     * 如果该节点的after为尾节点,则last=before 否则 after.before = before
     * 最后再把该节点挂在尾节点上
     */
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {//e不为尾节点时
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;//获取前后节点
            p.after = null;
            if (b == null)//如果b为头结点时
                head = a;
            else
                b.after = a;//否则与a连接
            if (a != null)//a不为尾节点时
                a.before = b;//与b连接
            else
                last = b;
            if (last == null)//说明链表是空的
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;//挂靠到tail上
            ++modCount;
        }
    }

    /**
     * return true 删除链表中最久的entry
     * 一般重写来实现简单版的LRU缓存
     */
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }

    /* 新建一个node 并插入到尾部 */
    Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }

    // link at the end of list
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)//链表为null时
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }



```
