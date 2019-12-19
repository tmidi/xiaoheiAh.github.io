# Collections-Linkedlist


<!--more-->

实现了`List`和`Deque`接口的双向队列,允许插入`null`值

#### 主要属性
```java
    transient int size = 0;//大小

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first; //头结点

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last; //尾节点
    //Node节点长这样:item实体对象,prev/next指向前一个后一个节点
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```
#### 构造函数
主要介绍带参数的构造函数
```java
    //c == null 时会抛空指针异常
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
    //最终会走到 addAll方法
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);//判断index是否非法 index<0 || index>size
        //集合转数组
        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) { //默认会从最后一个节点追加
            succ = null;
            pred = last;
        } else { //自定义index后 会先取到该index对应的对象
            succ = node(index);//调用node方法取得index位置下的node
            pred = succ.prev;
        }
        
        //遍历需要赋值的数组
        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);//不要next节点是因为迭代时可以自行设置
            if (pred == null)//说明是从头开始
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }
        
        //尾部node双向绑定
        if (succ == null) {//不是指定index地方插入时,即从尾部插入,没有succ节点
            last = pred;
        } else {//从指定index插入时,需要与尾部节点连接
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }

    Node<E> node(int index) {
        // assert isElementIndex(index);
        //掰成两半儿查找
        if (index < (size >> 1)) {//小于size的一半时从头开始查
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {//index大于size的一半时,从后往前找
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

### 主要方法
1.get
```java
    public E get(int index) {
        checkElementIndex(index);//检查index界限,会抛下标越界异常
        return node(index).item;//遍历取得指定index下的node
    }
```
2.set
```java
    public E set(int index, E element) {
        //检查下标
        checkElementIndex(index);
        Node<E> x = node(index);//获取对应的node
        E oldVal = x.item;//取出旧item
        x.item = element;//赋值新item
        return oldVal;
    }
```
3. add
```java
    public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)//index等于size大小时在最后追加
            linkLast(element);
        else
            linkBefore(element, node(index));//在该index直接添加
    }
    
    //在尾部连接一个对象
    void linkLast(E e) {
        final Node<E> l = last;//获得当前的最后一个节点
        final Node<E> newNode = new Node<>(l, e, null);//新建一个节点,当前最后一个节点作为prev节点
        last = newNode;
        if (l == null)//若前一个节点为null list还没有值 则头尾都用该节点
            first = newNode;
        else
            l.next = newNode;//将上一个节点与新节点连接起来
        size++;//链表长度+1
        modCount++;//操作链表次数+1
    }

    //官方文档上这么说:在一个非空节点前插入新节点 
    //但是其实没有做非空校验了
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null; 非空校验注释掉了
        final Node<E> pred = succ.prev;//中间插入 那就是succ的prev节点要重新与新节点的prev连接,新节点的next节点为succ节点
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;//succ节点与新节点连接
        if (pred == null)//上一个节点为空时则首尾节点都是该节点
            first = newNode;
        else
            pred.next = newNode;
        size++;//链表长度+1
        modCount++;//操作次数+1
    }
```
4.remove
除了各种逻辑最后都会用到unlink方法:大致是将需要删除的对象的prev和next节点重新连接起来,在将该对象置空让gc回收
```java
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;//获取该节点下一个节点
        final Node<E> prev = x.prev;//获取该节点上一个节点

        if (prev == null) {//上一个节点为null时则删除的是头结点 将下一个节点变为头结点
            first = next;
        } else {
            prev.next = next;//prev不为null时 将prev的next改为x的next
            x.prev = null;//将x的prev置空 让gc回收
        }

        if (next == null) {//x的next为null时说明该节点是尾节点
            last = prev;
        } else {
            next.prev = prev;//next的prev挂靠到x的prev
            x.next = null;//x的next置空回收
        }

        x.item = null;//item置空回收
        size--;//长度减1
        modCount++;//操作次数+1
        return element;
    }
```
5. clear操作
```java
    //清空所有的节点
    public void clear() {
        // Clearing all of the links between nodes is "unnecessary", but:
        // - helps a generational GC if the discarded nodes inhabit
        //   more than one generation
        // - is sure to free memory even if there is a reachable Iterator
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }

```
###  查询操作
1.indexOf
容易看懂就不解释了,查不到时返回-1,可以查null
```java
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```
### 队列类的操作
1. peek
```java
    //获取头节点 但不删除 可以为null
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
```
2. element 和peek一样也是获取头结点 但是若为null会抛空指针异常
3. poll 就是队列里的pop操作,即出队
```java
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
```
4.offer入队
队尾插入元素
```java
    public boolean offer(E e) {
        return add(e);
    }
```

### LinkedList中的迭代器
```java
    //该迭代器是快速失败的,如果在创建迭代器后操作了链表(add/remove),不是迭代器中的操作(add/remove),就会抛ConcurrentModificationException异常.
    //原因是迭代器中维护了expectedModCount每次操作前都会比较该值与modCount是否一致,不一致就抛,所以在迭代中增删节点时还是要通过迭代器的操作比较好
    private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;//用作返回节点
        private Node<E> next;//记录下一个节点
        private int nextIndex;//下一个index
        private int expectedModCount = modCount;//初始化期望操作值

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : node(index);//若index==size则是尾节点
            nextIndex = index;
        }

        public boolean hasNext() {
            return nextIndex < size;//通过坐标值判断是否有下一个
        }

        public E next() {
            checkForComodification();//校验操作避免使用list的add/remove操作
            if (!hasNext())
                throw new NoSuchElementException();

            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }

        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        public E previous() {
            checkForComodification();
            if (!hasPrevious())
                throw new NoSuchElementException();

            lastReturned = next = (next == null) ? last : next.prev;
            nextIndex--;
            return lastReturned.item;
        }

        public int nextIndex() {
            return nextIndex;
        }

        public int previousIndex() {
            return nextIndex - 1;
        }
        //迭代器的删除节点操作
        public void remove() {
            checkForComodification();//校验操作次数
            if (lastReturned == null)//状态校验
                throw new IllegalStateException();

            Node<E> lastNext = lastReturned.next;//获得当前节点的下一个节点
            unlink(lastReturned);//断开当前节点,将当前节点的前后节点相连接 size会减1 modCount会+1
            if (next == lastReturned)
                next = lastNext;
            else
                nextIndex--;//坐标值减1
            lastReturned = null;
            expectedModCount++;//保持与modCount一致
        }
        //设置当前的节点的item
        public void set(E e) {
            if (lastReturned == null)
                throw new IllegalStateException();
            checkForComodification();
            lastReturned.item = e;
        }
        //和普通add操作一样 主要是需要将nextIndex和expectedModCount都+1
        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            if (next == null)
                linkLast(e);
            else
                linkBefore(e, next);
            nextIndex++;
            expectedModCount++;
        }
        //挺方便的迭代 重写下accept方法 可以用lambda表达式
        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (modCount == expectedModCount && nextIndex < size) {
                action.accept(next.item);
                lastReturned = next;
                next = next.next;
                nextIndex++;
            }
            checkForComodification();
        }
        //操作校验
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```


