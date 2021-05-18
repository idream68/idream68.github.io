---
title: java-linkedlist
date: 2021-05-17 21:21:15
categories:
  - java
  - container
tags:
  - java
  - list
---

LinkedList类层次结构
先来看看LinkedList的定义

```java
public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

从中我们可以了解到

- LinkedList<E>：说明它支持泛型。
- extends AbstractSequentialList<E>
- AbstractSequentialList 继承自AbstractList，但AbstractSequentialList 只支持按次序访问
- 而不像 AbstractList 那样支持随机访问。这是LinkedList随机访问效率低的原因之一。
- implements List<E>：说明它支持集合的一般操作。
- implements Deque<E>：Deque，Double ended queue，双端队列。LinkedList可用作队列或双端队列就是因为实现了它。
- implements Cloneable：表明其可以调用clone()方法来返回实例的field-for-field拷贝。
- implements java.io.Serializable：表明该类是可以序列化的。

与ArrayList对比发现，LinkedList并没有实现RandomAccess，而实现RandomAccess表明其支持快速（通常是固定时间）随机访问。此接口的主要目的是允许一般的算法更改其行为，从而在将其应用到随机或连续访问列表时能提供良好的性能。这是LinkedList随机访问效率低的原因之一。

下图是LinkedList的类结构层次图



![image](/images/java-linkedlist/java-linkedlist-1.jpg)



### 全局变量

```java
/**
 * LinkedList节点个数
 */
transient int size = 0;

/**
 * 指向头节点的指针
 * Invariant: (first == null && last == null) ||
 *            (first.prev == null && first.item != null)
 */
transient Node<E> first;

/**
 * 指向尾节点的指针
 * Invariant: (first == null && last == null) ||
 *            (last.next == null && last.item != null)
 */
transient Node<E> last;
```

关于node的详细介绍请看本文末尾的**内部类—Node.java**一节。Node表示链表每个节点的内部结构，包括一个数据域item，一个后置指针next，一个前置指针prev。

### 构造方法

接下来，看LinkedList提供的构造方法。ArrayList提供了两种构造方法。

1. 构造空LinkedList。

```java
/**
 * 构造一个空链表.
 */
public LinkedList() {
}
```

2. 构造空LinkedList。

```java
/**
 * 根据指定集合c构造linkedList。先构造一个空linkedlist，在把指定集合c中的所有元素都添加到linkedList中。
 *
 * @param  c 指定集合
 * @throws NullPointerException 如果特定指定集合c为null
 */
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

### 方法

#### **以下几个方法是操作链表的底层方法**

##### **linkFirst(E e)**

方法在表头添加指定元素e

```java
/**
 * 在表头添加元素
 */
private void linkFirst(E e) {
    //使节点f指向原来的头结点
    final Node<E> f = first;
    //新建节点newNode，节点的前指针指向null，后指针原来的头节点
    final Node<E> newNode = new Node<>(null, e, f);
    //头指针指向新的头节点newNode 
    first = newNode;
    //如果原来的头结点为null，更新尾指针，否则使原来的头结点f的前置指针指向新的头结点newNode
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}
```

##### **linkLast(E e)**

方法在表尾插入指定元素e

```java
/**
 * 在表尾插入指定元素e
 */
void linkLast(E e) {
    //使节点l指向原来的尾结点
    final Node<E> l = last;
    //新建节点newNode，节点的前指针指向l，后指针为null
    final Node<E> newNode = new Node<>(l, e, null);
    //尾指针指向新的头节点newNode
    last = newNode;
    //如果原来的尾结点为null，更新头指针，否则使原来的尾结点l的后置指针指向新的头结点newNode
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

##### **linkBefore( E e, Node succ)**

方法在指定节点succ之前插入指定元素e

```java
/**
 * 在指定节点succ之前插入指定元素e。指定节点succ不能为null。
 */
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    //获得指定节点的前驱
    final Node<E> pred = succ.prev;
    //新建节点newNode，前置指针指向pred，后置指针指向succ
    final Node<E> newNode = new Node<>(pred, e, succ);
    //succ的前置指针指向newTouch
    succ.prev = newNode;
    //如果指定节点的前驱为null，将newTouch设为头节点。否则更新pred的后置节点
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

##### **unlinkFirst( Node f)**

方法删除并返回头结点f

```java
/**
 * 删除头结点f，并返回头结点的值
 */
private E unlinkFirst( Node<E> f) {
    // assert f == first && f != null;
    // 保存头结点的值
    final E element = f.item;
    // 保存头结点指向的下个节点
    final Node<E> next = f.next;
    //头结点的值置为null
    f.item = null;
    //头结点的后置指针指向null
    f.next = null; // help GC
    //将头结点置为next
    first = next;
    //如果next为null，将尾节点置为null，否则将next的后置指针指向null
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    //返回被删除的头结点的值
    return element;
}
```

##### **unlinkLast( Node l)**

方法删除并返回尾结点f

```java
/**
 * 删除尾节点l.并返回尾节点的值
 */
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    // 保存尾节点的值
    final E element = l.item;
    //获取新的尾节点prev
    final Node<E> prev = l.prev;
    //旧尾节点的值置为null
    l.item = null;
    //旧尾节点的后置指针指向null
    l.prev = null; // help GC
    //将新的尾节点置为prev
    last = prev;
    //如果新的尾节点为null，头结点置为null，否则将新的尾节点的后置指针指向null
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    //返回被删除的尾节点的值
    return element;
}
```

##### **unlink( Node x)**

方法删除指定节点，返回指定元素的值x

```java
/**
 * 删除指定节点，返回指定元素的值
 */
E unlink(Node<E> x) {
    // assert x != null;
    // 保存指定节点的值
    final E element = x.item;
    // 获取指定节点的下个节点next
    final Node<E> next = x.next;
    // 获取指定节点的下个节点prev
    final Node<E> prev = x.prev;
    //如果prev为null，那么next为新的头结点，否则将prev的后置指针指向next，x的前置指针指向null
    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }
    //如果next为null，那么prev为新的尾结点，否则将next的前置指针指向prev，x的后置指针指向null
    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
    //x的值置为null
    x.item = null;
    size--;
    modCount++;
    //返回被删除的节点的值
    return element;
}
```

##### **getFirst()**

```java
/**
 * 返回链表中的头结点的值.
 *
 * @return 返回链表中的头结点的值
 * @throws NoSuchElementException 如果链表为空
 */
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}
```

##### **getLast()**

```java
/**
 * 返回链表中的尾结点的值.
 *
 * @return 返回链表中的头结点的值
 * @throws NoSuchElementException 如果链表为空
 */
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
```

#### **常用的操作方法**

##### **removeFirst**

```java
/**
 * 删除并返回表头元素.
 *
 * @return 表头元素
 * @throws NoSuchElementException 链表为空
 */
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
```

##### **removeLast**

```java
/**
 * 删除并返回表尾元素.
 *
 * @return 表尾元素
 * @throws NoSuchElementException 链表为空
 */
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}
```

##### **addFirst( E e)**

```java
/**
 * 在表头插入指定元素.
 *
 * @param e 插入的指定元素
 */
public void addFirst(E e) {
    linkFirst(e);
}
```

##### **addLast( E e)**

```java
/**
 * 在表尾插入指定元素.
 * 
 * 该方法等价于add()
 *
 * @param e 插入的指定元素
 */ 
public void addLast(E e) {
    linkLast(e);
}
```

##### **contains( Object o)**

```java
/**
 * 判断链表是否包含指定对象o
 * @param o 指定对象
 * @return 是否包含指定对象
 */
public boolean contains(Object o) {
    return indexOf(o) != -1;
}
```

##### **size()**

```java
/**
 * 返回链表元素个数
 *
 * @return 链表元素个数
 */
public int size() {
    return size;
}
```

##### **add( E e)**

```java
/**
 * 在表尾插入指定元素.
 *
 * 该方法等价于addLast
 *
 * @param e 插入的指定元素
 * @return true
 */
public boolean add(E e) {
    linkLast(e);
    return true;
}
```

##### **remove(Object o)**

```java
/**
 * 正向遍历链表，删除出现的第一个值为指定对象的节点
 *
 * @param o 要删除的节点置
 * @return 如果o在链表中存在，返回true
 */
public boolean remove(Object o) {
    //遍历链表，如果o为null，删除第一个值为null的节点，返回true。如果不为null，删除第一个值为o的节点。如果链表中存在o，就返回true。
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

##### **addAll( Collection<? extends E> c)**

```java
/**
 * 插入指定集合到链尾
 *
 * @param c 指定集合
 * @return 如果链表改变，返回true
 * @throws NullPointerException 如果指定集合为null
 */
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}
```

##### **addAll( int index, Collection<? extends E> c)**

```java
/**
 * 插入指定集合到链尾的指定位置
 *
 * @param index 指定的插入位置
 * @param c 插入的指定集合
 * @return 如果链表改变，返回true
 * @throws IndexOutOfBoundsException 如果index<0或index>size
 * @throws NullPointerException 如果指定集合为null
 */
public boolean addAll(int index, Collection<? extends E> c) {
    //检查插入的位置是否合法
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;//如果c是空的话那么就返回false

    //定义两个节点指针，指向插入点前后的节点元素
    Node<E> pred, succ;
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }

    // 插入集合中所有元素
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    // 修改插入后的指针问题
    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

##### **clear()**

```java
/**
 * 删除链表中的所有元素
 */
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

#### **按位操作**

##### **get( int index)**

```java
/**
 * 返回指定索引处的元素
 *
 * @param index 指定索引
 * @return 指定索引处的元素
 * @throws IndexOutOfBoundsException 如果索引index越界
 */
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

##### **set( int index, E element)**

```java
/**
 * 替换指定索引处的元素为指定元素element
 *
 * @param index 被替换的元素的索引
 * @param element 
 * @return 指定元素element
 * @throws IndexOutOfBoundsException 索引越界
 */
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```

##### **add( int index, E element)**

```java
/**
 * 插入指定元素到指定索引处
 *
 * @param index 指定索引
 * @param element 指定元素
 * @throws IndexOutOfBoundsException 索引越界
 */
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
```

##### **remove( int index)**

```java
/**
 * 删除指定索引处的元素
 *
 * @param 指定索引
 * @return 指定索引处的元素
 * @throws IndexOutOfBoundsException 索引越界
 */
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

##### **isElementIndex( int index)**

```java
/**
 * 返回索引是否越界
 */
private boolean isElementIndex(int index) {
    return index >= 0 && index < size;
}
```

##### **isPositionIndex(int index)**

```java
/**
 * 返回插入操作时给定的索引是否合法
 */
private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;
}
```

##### **outOfBoundsMsg( int index)**

```java
/**
 * 索引越界时打印的信息
 */
private String outOfBoundsMsg(int index) {
    return "Index: "+index+", Size: "+size;
}
```

##### **checkElementIndex( int index)**

```java
/**
 * 检查索引是否越界
 */
private void checkElementIndex(int index) {
    if (!isElementIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

##### **checkPositionIndex( int index)**

```java
/**
 * 检查插入操作时给定的索引是否合法
 */
private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

##### **node( int index)**

```java
/**
 * 返回在指定索引处的非空元素
 */
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

#### **查找操作**

##### **indexOf( Object o)**

```java
/**
 * 正向遍历链表，返回指定元素第一次出现时的索引。如果元素没有出现，返回-1.
 * 
 * @param o 需要查找的元素
 * @return 指定元素第一次出现时的索引。如果元素没有出现，返回-1。
 */
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

##### **lastIndexOf( Object o)**

```java
/**
 * 逆向遍历链表，返回指定元素第一次出现时的索引。如果元素没有出现，返回-1.
 * 
 * @param o 需要查找的元素
 * @return 指定元素第一次出现时的索引。如果元素没有出现，返回-1。
 */
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}
```

#### **队列操作**

##### **peek()**

```java
/**
 * 返回头节点的元素，如果链表为空则返回null
 *
 * @return 返回头节点的元素，如果链表为空则返回null
 * @since 1.5
 */
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
```

##### **element()**

```java
/**
 * 获取表头节点的值，头节点为空抛出异常
 *
 * @return 获取表头节点的值
 * @throws NoSuchElementException 如果链表为空
 * @since 1.5
 */
public E element() {
    return getFirst();
}
```

##### **poll()**

```java
/**
 * 返回并删除头节点，如果链表为空则返回null
 *
 * @return 返回并删除头节点，如果链表为空则返回null
 * @since 1.5
 */
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
```

##### **remove()**

```java
/**
 * 删除并返回头节点，如果链表为空，抛出异常
 *
 * @return 头结点
 * @throws NoSuchElementException 链表为空
 * @since 1.5
 */
public E remove() {
    return removeFirst();
}
```

##### **offer(E e)**

```java
/**
 * 添加元素到队列尾部
 *
 * @param e 指定元素
 * @return 
 * @since 1.5
 */
public boolean offer(E e) {
    return add(e);
}
```

#### **双向队列操作**

##### **offerFirst( E e)**

```java
/**
 * 插入指定元素到队列头部.
 *
 * @param e 插入的元素
 * @return  true
 * @since 1.6
 */
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}
```

##### **offerLast( E e)**

```java
/**
 * 插入指定元素到队列尾部.
 *
 * @param e 插入的元素
 * @return  true
 * @since 1.6
 */
public boolean offerLast(E e) {
    addLast(e);
    return true;
}
```

##### **peekFirst()**

```java
/**
* 返回队列的头元素，如果头节点为空则返回空
*
* @return 返回队列的头元素，如果头节点为空则返回空
* @since 1.6
*/
public E peekFirst() {
   final Node<E> f = first;
   return (f == null) ? null : f.item;
}
```

##### **peekLast()**

```java
/**
* 返回队列的尾元素，如果尾节点为空则返回空
*
* @return 返回队列的尾元素，如果尾节点为空则返回空
* @since 1.6
*/
public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}
```

##### **pollFirst()**

```java
/**
 * 删除并返回队列的第一个元素，如果头节点为空，则返回null.
 *
 * @return 删除并返回队列的第一个元素，如果头节点为空，则返回null.
 * @since 1.6
 */
public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
```

##### **pollLast()**

```java
/**
 * 删除并返回队列的最后个元素，如果尾节点为空，则返回null.
 *
 * @return 删除并返回队列的最后一个元素，如果尾节点为空，则返回null.
 * @since 1.6
 */
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}
```

##### **push( E e)**

```java
/**
 * 插入指定元素到栈头
 *
 * 此方法等价于addFirst(e)
 *
 * @param e 指定元素
 * @since 1.6
 */
public void push(E e) {
    addFirst(e);
}
```

##### **pop()**

```java
/**
 * 删除并返回栈头元素
 *
 * 此方法等价于pop()
 *
 * @return 返回栈头元素
 * @throws NoSuchElementException 如果栈为空
 * @since 1.6
 */
public E pop() {
    return removeFirst();
}
```

##### **removeFirstOccurrence( Object o)**

```java
/**
 * 正向遍历栈，删除指定对象第一次出现时，索引对应的元素
 * 
 * @param o 被删除的元素
 * @return true 如果元素出现
 * @since 1.6
 */
public boolean removeFirstOccurrence(Object o) {
    return remove(o);
}
```

原文：https://blog.csdn.net/panweiwei1994/article/details/77110354
