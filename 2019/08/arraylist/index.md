# Collections-Arraylist


<!--more-->

## ArrayList和LinkedList和Vector的区别 

简单讲:
1. ArrayList和LinkedList是线程不安全的,而Vector在增删改的操作上都有synchronized关键字修饰,是线性的,但是效率不高
2. ArrayList和Vector都是基于数组的实现,扩容时,ArrayList扩容为1.5倍,Vector扩容为2倍,查找快,增删慢.LinkedList是一个双向链表,增删快,查找慢.
> [参考blog](https://blog.csdn.net/csdn15698845876/article/details/79480066)

## SynchronizedList和Vector的区别 
1. SynchronizedList有很好的扩展和兼容功能。他可以将所有的List的子类转成线程安全的类。
2. 使用SynchronizedList的时候，进行遍历时要手动进行同步处理。 
3. SynchronizedList可以指定锁定的对象。
> [Hollis的blog](http://www.hollischuang.com/archives/498)

## 参数

1. static final int DEFAULT_CAPACITY=10 默认大小 
2. transient object[] elementData; arraylist存储对象的数组.当空ArrayList第一次添加对象时,容量会扩展成`DEFAULT_CAPACITY`
3. size  elementData中实际的对象数

## 构造函数
> 看几个主要的
1. 带初始化参数,参数违法时会抛RuntimeException
```java
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```
2. 从其他集合中导入,collection需要notNull，否则会抛`空指针` 
```java
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```
## 主要方法
1. get
```java
    public E get(int index) {
	//判断index是否在范围内 不在会抛 IndexOutOfBoundsException
        rangeCheck(index);
	//获取对象值
        return elementData(index);
    }
```
2. set 替换指定位置的值
```java
    public E set(int index, E element) {
        rangeCheck(index);//范围检查

        E oldValue = elementData(index);//获取对象旧值
        elementData[index] = element; //赋新值
        return oldValue; //返回旧值
    }
```
3. add 在`elementData`尾部添加一个对象
```java
    public boolean add(E e) {
	//确保在容量范围内,不在则扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;//操作list的次数
	
	//若最小容量大于 elementData的长度 则扩容
        // overflow-conscious code 
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1); //扩容大概是1.5倍
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

```
4. add 指定位置添加对象
```java
    public void add(int index, E element) {
	//专门的add操作范围检查  主要是保证 0 < index < size
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```













