# Collections-Hashmap


<!--more-->

> 美团的blog:https://tech.meituan.com/java_hashmap.html
> 参考blog: [田小波的博客](https://www.tianxiaobo.com/2018/01/18/HashMap-%E6%BA%90%E7%A0%81%E8%AF%A6%E7%BB%86%E5%88%86%E6%9E%90-JDK1-8/)
> [红黑树介绍](https://www.tianxiaobo.com/2018/01/11/%E7%BA%A2%E9%BB%91%E6%A0%91%E8%AF%A6%E7%BB%86%E5%88%86%E6%9E%90/)

## HashMap,HashTable,ConcurrentHashMap的区别?
**HashMap**是非线程安全的,**HashTable**和**ConcurrentHashMap**是线程安全的.HashTable不允许null key和null Value,HashMap允许.
ConcurrentHashMap推出之后官方推荐不要在使用HashTable作为线程安全的使用类,而是使用这个.关于**ConcurrentHashMap**后面再学习.
> [参考blog](https://juejin.im/post/5add97a46fb9a07aa212f4c0)

## HashMap在不同版本之间实现的区别?
> [区别](http://linfenliang.com/hashmap/2017/08/04/HashMapInJDK-6-7-8-Differ/)


## 官方文档介绍:
1. 基于`Map`接口实现的哈希表.提供了所有map可选的操作,允许key为null,value为null.`HashMap`与`HashTable`基本一致,除了`HashMap` *线程不安全并且允许为空*.
不保证有序,尤其不保证顺序一直不变(因为扩容时会rehash,基本上就顺序就重排了)
2. 假设hash分布均匀的情况下,基本的操作(`get/put`)性能很不错.迭代所需要的时间与`buckets`数量与每个`bukets`下的键值对的数量之和成正比.所以官方建议如果要求hashmap的迭代性能的话,初始的`capacity`不能太高,`loadFactor`不要太高.
3. HashMap有两个重要的参数:`initial capacity`,`load factor`.capacity定义bucket的数量,initial capacity定义的是初始化bucket数量.load factor(中文名: *加载因子* )是判断哈希表是否需要扩容的阈值,当entries数量超过(load factor * current capacity),哈希表会触发`rehash`操作,内部数据结构会重整,buckets数量会变为之前大约两倍左右

4. 通常情况下,load factor 默认`0.75f`,在时间空间上是很平衡的.值偏高时,空间减少,查找时间上升了(影响大部分的操作,get/put之类的),在设置初始容量时,需要考虑到预期的entries数量和加载因子,以便最小化rehash的数量.如果初始化的容量大于最大数量的entries除以加载因子,不会发生rehash操作.

5. 如果有大量的键值对存到hashmap中,那么创建一个足够大的hashmap来存储要比让他自动rehash扩容来存储的性能要好很多.注意:具有相同hashcode的多个key肯定会影响哈希表的性能.为了改善这种影响,当key是Comparable类型时,可以通过key之间的比较顺序来打破这种关系.

6. 注意hashmap是`Non synchronized`,即 *非线程安全*.如果多线程并发访问hashmap,并且至少有一个线程操作map的结构,在外部必须`synchronized`.(结构修改是指任何关于add或delte的操作,仅仅只是修改key关联的value时则不属于结构修改).通常在将object封装进map做synchronized操作

7. 如果不存在上面的objects,那这个map需要被Collections.synchronizedMap包装下.最好在创建的时候就做好,防止偶然的并发访问.
```java
Map m = Collections.synchronizedMap(new HashMap(...));
```

8. 迭代器的所有方法都是`fail-fast`,如果迭代器创建后,在迭代器里的结构操作必须通过迭代器的方法来操作,否则会抛`ConcurrentModificationException`.因此,面对并发修改,迭代器会快速而干净的失败,而不是在未来的不确定时间冒任意非确定行为的风险.
  
9. 请注意,迭代器的快速失败行为无法得到保证,因为一般来说,在存在不同步的并发修改时,不可能做出任何硬性保证. 快速失败迭代器会尽最大努力抛出ConcurrentModificationException. 因此,编写依赖于此异常的程序以确保其正确性是错误的:迭代器的快速失败行为应该仅用于检测错误.

## 源码
```java
    /**
     * The default initial capacity - MUST be a power of two.
     * 默认的容量16 必须是2的n次方
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * 最大的容量限制
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认的加载因子
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    /**
     * 大于这个值转红黑树
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * 大于这个值小于 TREEIFY_THRESHOLD 不转树
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * hashmap整体容量大于这个值时才能树化
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
     * between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
    
    /**
     * node节点
     * Basic hash bin node, used for most entries.  (See below for
     * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
     */
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

    //hashMap中的静态方法
    /**
     * hash方法详解 blog:http://www.hollischuang.com/archives/2091
     * 扰动算法--使hash分布更均匀
     */
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    
    //取模运算,获得对象存储到bukets的下标
    //实际上就是取模,一般取模使用% 但是考虑到效率问题,采用位运算
    //X % 2^n = X & (2^n-1) 这也是为什么hashmap容量为2的n次方的原因
    static int indexFor(int h, int length) {
	return h & (length-1);
    }
    //返回hashmap的容量 2的n次方 很巧妙的位运算
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
    
    /**
     * 参数都用transient不让序列化的原因:https://segmentfault.com/q/1010000000630486
     */

    //bukets hashmap是链表加数组的结构.此为数组
    transient Node<K,V>[] table;
    
    //保存键值对的Entry
    transient Set<Map.Entry<K,V>> entrySet;
    
    //hashmap的size
    transient int size;
    
    //结构操作次数 可用于快速失败的比较条件 例如并发操作时
    transient int modCount;
    
    //resize的临界点: capacity * load factor 
    int threshold;
    
    //加载因子
    final float loadFactor;

    //公有操作方法

    //构造方法
    /**
     * 根据 initial capactity 和 loadFactor创建空的hashmap
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
	//校验initialCapacity
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
	//容量校验
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
	//校验loadFactor isNaN--> 是否是一个number Not-a-Number
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
	//加载因子赋值
        this.loadFactor = loadFactor;
	//扩容阈值赋值 2的n次方
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * 通过initialCapacity赋值
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    
    /**
     * 根据默认容量和默认加载因子创建空的hashmap
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    /**
     * 根据传进来的map创建一个新的hashMap 
     * initialCapacity 足以装下参数map的数量
     * loadFactor使用默认值
     * Constructs a new <tt>HashMap</tt> with the same mappings as the
     * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
     * default load factor (0.75) and an initial capacity sufficient to
     * hold the mappings in the specified <tt>Map</tt>.
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }

    /**
     * Implements Map.putAll and Map constructor
     *
     * @param m the map
     * @param evict false when initially constructing this map, else
     * true (relayed to method afterNodeInsertion).
     * evict 初始化构建map时 为false 其他情况下为true
     */
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
	    //初次创建hashmap
            if (table == null) { // pre-size
		//计算m所需要的容量
                float ft = ((float)s / loadFactor) + 1.0F;
		//获得真实的容量
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
		//如果比默认的阈值大则计算该 t 对应的capacity
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            else if (s > threshold) // 如果是table不为null 即是后续往map中添加 如果s > 阈值就要重置map了
                resize();//resize操作 后面介绍
	    //确定容量后put操作
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);//
            }
        }
    }

    /*主要调用 putVal */
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * put 操作
     * Implements Map.put and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value 不存在才put<D-[>
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
	//若是新建map的情况下 resize创建指定长度的table 
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
	//取模计算该key对应的数组下标 并判断该坐标下的对象是否为null
	//为null时创建一个新node存入tab[i]
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {//tab[i] != null
            Node<K,V> e; K k;
	    //如果p与存入的key完全相同
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
		//如果是红黑树节点 调用putTreeVal
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
		//普通的put
		//binCount记录了链表的长度
                for (int binCount = 0; ; ++binCount) {
		    //如果当前node的next==null说明就可以往该链上添加一个节点
                    if ((e = p.next) == null) {
			//新建node接到p.next下面
                        p.next = newNode(hash, key, value, null);
			//如果binCount大于设定的红黑树化阈值
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);//红黑树化
                        break;
                    }
		    //如果key与链表中的任意node完全相同break
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
	    //如果存在该key
            if (e != null) { // existing mapping for key
                V oldValue = e.value;//获得旧值
                if (!onlyIfAbsent || oldValue == null)//若没有设置不存在才put或者oldValue=null
                    e.value = value;//赋新值
                afterNodeAccess(e);//LinkedHashMap操作
                return oldValue;//返回旧值
            }
        }
        ++modCount;
        if (++size > threshold)//是否需要扩容
            resize();
        afterNodeInsertion(evict);//LinkedHashMap操作
        return null;
    }

    /**
     * 扩容操作
     * 若是初始化则根据initialCapacity创建一个table
     * 否则,扩容为2的n次方倍
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
	    //超过最大值不会再扩容了
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold 扩成两倍
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // 默认配置 zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
	    //计算新的阈值
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
	    //把old buket 移到新的bukets里
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)//直接添加
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
			    //这个取模很精辟 请结合美团的blog resize 1.8优化学习
			    //因为扩容是2倍扩容,二进制中相当于左移一位
			    /**
			     * 假设一次扩容		    
			     * 扩容前	oldCap = 00010000   oldCap - 1 = 00001111
			     * 扩容后	newCap = 00100000   newCap - 1 = 00011111
			     * 可以看出扩容后 newCap-1 在高位多了1
			     * 计算index时 hash & n-1 = 原位置 + oldCap
			     * 所以只需要判断hash & oldCap是否为1
			     * 为1则把该node的位置移到 oldCap+原位置 
			     * 为 0 还在原位置
			     */
                            if ((e.hash & oldCap) == 0) {//为0说明位置没有变
                                if (loTail == null)//第一次添加时loHead=e
                                    loHead = e;
                                else
                                    loTail.next = e;//直接往后插入
                                loTail = e;
                            }
                            else {//为1 说明位置会+oldCap长度
                                if (hiTail == null)
                                    hiHead = e;//头节点初始化
                                else
                                    hiTail.next = e;//直接插入
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {//放在原位置上
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {//放在原位置+oldCap上
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }

    /**
     * get操作
     * 为null时返回null 这个要注意下
     */
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }


    /**
     * Implements Map.get and related methods
     * get方法
     * 主要是   key相等 或者 key equals的比较
     * @param hash hash for key
     * @param key the key
     * @return the node, or null if none
     */
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {//获得节点
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)//树节点
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

## 1.8红黑树化源码解析
```java
    /**
     * TreeNode extends LinkedHashMap.Entry
     * LinkedHashMap.Entry extends HashMap.Node
     */

    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links 红黑树父节点
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion 删除的时候用来连接前后
        boolean red;//红还是黑
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
    }     
    
    
    /**树化
     * putVal里有用到
     * 将链表重置为红黑树并放到该hash映射的tab下,如果tab过下则resize
     * Replaces all linked nodes in bin at index for given hash unless
     * table is too small, in which case resizes instead.
     */
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)//小于最小树化的容量时不树化而resize  capacity为64,
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;//头尾节点
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);//这个就是返回一个新建的TreeNode对象,内容为e
                if (tl == null)//确定是头结点
                    hd = p;//标记头结点
                else {//非头结点就首尾连接
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;//尾节点一直为p
            } while ((e = e.next) != null);//遍历链表 其实此时形成也还算是个链表
            if ((tab[index] = hd) != null)//将该treeNode挂到table下
                hd.treeify(tab);//完成红黑树化
        }
    }

       /**
         * Forms tree of the nodes linked from this node.
         * @return root of tree
         */
        final void treeify(Node<K,V>[] tab) {
            TreeNode<K,V> root = null;
            for (TreeNode<K,V> x = this, next; x != null; x = next) {//x 从当前节点开始(从treeifyBin里调用看是头结点)
                next = (TreeNode<K,V>)x.next;//获取下个节点
                x.left = x.right = null;
                if (root == null) {//设置root节点并给他黑色
                    x.parent = null;
                    x.red = false;
                    root = x;
                }
                else {
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    //遍历所有节点与当前节点x比较 调整位置 有点像冒泡排序
                    for (TreeNode<K,V> p = root;;) {
                        int dir, ph;
                        K pk = p.key;
                        //比较hash值
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);
                        
                        //根据dir判断x是p的左孩子 还是 右孩子
                        TreeNode<K,V> xp = p;
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                xp.left = x;
                            else
                                xp.right = x;
                            //平衡节点
                            root = balanceInsertion(root, x);
                            break;
                        }
                    }
                }
            }
            moveRootToFront(tab, root);
        }

        /**
         * Returns a list of non-TreeNodes replacing those linked from
         * this node.
         */
        final Node<K,V> untreeify(HashMap<K,V> map) {
            Node<K,V> hd = null, tl = null;
            for (Node<K,V> q = this; q != null; q = q.next) {
                Node<K,V> p = map.replacementNode(q, null);
                if (tl == null)
                    hd = p;
                else
                    tl.next = p;
                tl = p;
            }
            return hd;
        }

        /**
         * 红黑树版put操作
         * Tree version of putVal.
         */
        final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                                       int h, K k, V v) {
            Class<?> kc = null;
            boolean searched = false;
            TreeNode<K,V> root = (parent != null) ? root() : this;//每次从根节点遍历
            for (TreeNode<K,V> p = root;;) {
                int dir, ph; K pk;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    //如果当前节点key相同或equals 返回
                    return p;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0) {
                    //hash值如果相等 但类不相同,只能挨个对比左右孩子
                    if (!searched) {
                        TreeNode<K,V> q, ch;
                        searched = true;
                        if (((ch = p.left) != null &&
                             (q = ch.find(h, k, kc)) != null) ||
                            ((ch = p.right) != null &&
                             (q = ch.find(h, k, kc)) != null))
                            return q;
                    }
                    //哈希值相等 但键无法比较 只能通过其他方法比较
                    dir = tieBreakOrder(k, pk);
                }

                //得到两个节点的大小关系 即dir的值时
                //并判断只有在左孩子或右孩子不能
                TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    Node<K,V> xpn = xp.next;
                    TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    xp.next = x;
                    x.parent = x.prev = xp;
                    if (xpn != null)
                        ((TreeNode<K,V>)xpn).prev = x;
                    //平衡二叉树
                    moveRootToFront(tab, balanceInsertion(root, x));
                    return null;
                }
            }
        }

        /** 查找操作 传入 hash值 和 key值
         * Calls find for root node.
         */
        final TreeNode<K,V> getTreeNode(int h, Object k) {
            return ((parent != null) ? root() : this).find(h, k, null);//判断从当前节点还是root节点开始查找
        }


        /**
         * Finds the node starting at root p with the given hash and key.
         * The kc argument caches comparableClassFor(key) upon first use
         * comparing keys.
         */
        final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
            TreeNode<K,V> p = this;
            do {
                int ph, dir; K pk;
                TreeNode<K,V> pl = p.left, pr = p.right, q;
                //根据hash值查找 当前节点hash值大于h则 查左孩子 否则右孩子 当key相等或者equal时返回
                if ((ph = p.hash) > h)
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;
                else if (pl == null)
                    p = pr;
                else if (pr == null)
                    p = pl;
                else if ((kc != null ||
                          (kc = comparableClassFor(k)) != null) &&
                         (dir = compareComparables(kc, k, pk)) != 0)
                    p = (dir < 0) ? pl : pr;
                else if ((q = pr.find(h, k, kc)) != null)//不相等则从子树继续查找
                    return q;
                else
                    p = pl;
            } while (p != null);
            return null;
        }

```

