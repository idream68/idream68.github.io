---
title: java ArrayList
date: 2021-05-17 17:53:01
categories:
  - java
  - container
tags:
  - java
  - list
---

# 数据结构

![image](/images/java-arraylist/java-arraylist-1.jpg)

# 顶部注释

> List接口的大小可变数组的实现。实现了所有可选列表操作，并允许包括null在内的所有元素。除了实现List接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小。（此类大致上等同于Vector类，除了此类是不同步的。）
>
>size、isEmpty、get、set、iterator和listIterator操作都以固定时间运行。add操作以分摊的固定时间运行，也就是说，添加n个元素需要O(n)时间。其他所有操作都以线性时间运行（大体上讲）。与用于LinkedList实现的常数因子相比，此实现的常数因子较低。
>
>每个ArrayList实例都有一个容量。该容量是指用来存储列表元素的数组的大小。它总是至少等于列表的大小。随着向ArrayList中不断添加元素，其容量也自动增长。并未指定增长策略的细节，因为这不只是添加元素会带来分摊固定时间开销那样简单。
>
>在添加大量元素前，应用程序可以使用ensureCapacity操作来增加ArrayList实例的容量。这可以减少递增式再分配的数量。
>
>注意，此实现不是同步的。如果多个线程同时访问一个ArrayList实例，而其中至少一个线程从结构上修改了列表，那么它必须保持外部同步。（结构上的修改是指任何添加或删除一个或多个元素的操作，或者显式调整底层数组的大小；仅仅设置元素的值不是结构上的修改。）这一般通过对自然封装该列表的对象进行同步操作来完成。如果不存在这样的对象，则应该使用Collections.synchronizedList方法将该列表“包装”起来。这最好在创建时完成，以防止意外对列表进行不同步的访问：
List list = Collections.synchronizedList(new ArrayList(…));
>
>此类的iterator和listIterator方法返回的迭代器是快速失败的：在创建迭代器之后，除非通过迭代器自身的remove或add方法从结构上对列表进行修改，否则在任何时间以任何方式对列表进行修改，迭代器都会抛出ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。
>
>注意，迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器会尽最大努力抛出ConcurrentModificationException。因此，为提高这类迭代器的正确性而编写一个依赖于此异常的程序是错误的做法：迭代器的快速失败行为应该仅用于检测bug。
>
>此类是Java Collections Framework的成员。

从上面的内容中可以总结出以下几点：

- **底层**：ArrayList是List接口的大小可变数组的实现。
- **是否允许null**：ArrayList允许null元素。
- **时间复杂度**：size、isEmpty、get、set、iterator和listIterator方法都以固定时间运行，时间复杂度为O(1)。add和remove方法需要O(n)时间。与用于LinkedList实现的常数因子相比，此实现的常数因子较低。
- **容量**：ArrayList的容量可以自动增长。
- **是否同步**：ArrayList不是同步的。
- **迭代器**：ArrayList的iterator和listIterator方法返回的迭代器是fail-fast的。

# 定义

先来看看ArrayList的定义：

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>,RandomAccess,Cloneable,java.io.Serializable
```

从中我们可以了解到：

- ArrayList<E>：说明ArrayList支持泛型。
- extends AbstractList<E> ：继承了AbstractList。AbstractList提供List接口的骨干实现，以最大限度地减少“随机访问”数据存储（如ArrayList）实现Llist所需的工作。
- implements List<E>：实现了List。实现了所有可选列表操作。
- implements RandomAccess：表明ArrayList支持快速（通常是固定时间）随机访问。此接口的
- 要目的是允许一般的算法更改其行为，从而在将其应用到随机或连续访问列表时能提供良好的性能。
- implements Cloneable：表明其可以调用clone()方法来返回实例的field-for-field拷贝。
implements java.io.Serializable：表明该类具有序列化功能。

但继承实现信息还不够完整，下图是ArrayList的类结构层次图。

![image](/images/java-arraylist/java-arraylist-2.jpg)

# 域

```java
/**
 * 初始化默认容量。
 */
private static final int DEFAULT_CAPACITY = 10;

/**
 * 指定该ArrayList容量为0时，返回该空数组。
 */
private static final Object[] EMPTY_ELEMENTDATA = {};

/**
 * 当调用无参构造方法，返回的是该数组。刚创建一个ArrayList 时，其内数据量为0。
 * 它与EMPTY_ELEMENTDATA的区别就是：该数组是默认返回的，而后者是在用户指定容量为0时返回。
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 保存添加到ArrayList中的元素。
 * ArrayList的容量就是该数组的长度。
 * 该值为DEFAULTCAPACITY_EMPTY_ELEMENTDATA 时，当第一次添加元素进入ArrayList中时，数组将扩容值DEFAULT_CAPACITY。
 * 被标记为transient，在对象被序列化的时候不会被序列化。
 */
transient Object[] elementData; // non-private to simplify nested class access

/**
 * ArrayList的实际大小（数组包含的元素个数）。
 * @serial
 */
private int size;
```

**思考：elementData被标记为transient，那么它的序列化和反序列化是如何实现的呢？**
ArrayList自定义了它的序列化和反序列化方式。详情请查看writeObject(java.io.ObjectOutputStream s)和readObject(java.io.ObjectOutputStream s)方法。

# 构造方法

1. ArrayList(int initialCapacity)：构造一个指定容量为capacity的空ArrayList。
2. ArrayList()：构造一个初始容量为 10 的空列表。
3. ArrayList(Collection<? extends E> c)：构造一个包含指定 collection 的元素的列表，这些元素是按照该 collection 的迭代器返回它们的顺序排列的。

## ArrayList( int initialCapacity)

```java
/**
 * 构造一个指定初始化容量为capacity的空ArrayList。
 *
 * @param  initialCapacity  ArrayList的指定初始化容量
 * @throws IllegalArgumentException  如果ArrayList的指定初始化容量为负。
 */
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

## ArrayList()

```java
/**
 * 构造一个初始容量为 10 的空列表。
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

## ArrayList(Collection<? extends E> c)

```java
/**
 * 构造一个包含指定 collection 的元素的列表，这些元素是按照该 collection 的迭代器返回它们的顺序排列的。
 *
 * @param c 其元素将放置在此列表中的 collection
 * @throws NullPointerException 如果指定的 collection 为 null
 */
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

# 核心方法
ArrayList有以下核心方法

| 方法名 | 时间复杂度 |
| -- | -- |
| get(int index) | O(1) |
| add(E e) | O(1) |
| add(add(int index, E element)) | O(n) |
| remove(int index) | O(n) |
| set(int index, E element) | O(1) |

## get( int index)

```java
/**
 * 返回list中索引为index的元素
 *
 * @param  index 需要返回的元素的索引
 * @return list中索引为index的元素
 * @throws IndexOutOfBoundsException 如果索引超出size
 */
public E get(int index) {
    //越界检查
    rangeCheck(index);
    //返回索引为index的元素
    return elementData(index);
}

/**
 * 越界检查。
 * 检查给出的索引index是否越界。
 * 如果越界，抛出运行时异常。
 * 这个方法并不检查index是否合法。比如是否为负数。
 * 如果给出的索引index>=size，抛出一个越界异常
 */
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

/**
 * 返回索引为index的元素
 */
@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}
```

从代码中可以看到，因为ArrayList底层是数组，所以它的get方法非常简单，先是判断一下有没有越界，之后就直接通过数组下标来获取元素。get方法的**时间复杂度是O(1)。**

add(E e)

```java
/**
 * 添加元素到list末尾.
 *
 * @param e 被添加的元素
 * @return true
 */
public boolean add(E e) {
    //确认list容量，如果不够，容量加1。注意：只加1，保证资源不被浪费
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

从源码中可以看到，add(E e)有两个步骤：

1. 空间检查，如果有需要进行扩容
2. 插入元素

空间检查和扩容的介绍在下面。

空间的问题解决后，插入过程就显得非常简单。

![image](/images/java-arraylist/java-arraylist-3.jpg)

## 扩容-ensureCapacity等方法

```java
/**
* 增加ArrayList容量。
* 
* @param   minCapacity   想要的最小容量
*/
public void ensureCapacity(int minCapacity) {
    // 如果elementData等于DEFAULTCAPACITY_EMPTY_ELEMENTDATA，最小扩容量为DEFAULT_CAPACITY，否则为0
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)? 0: DEFAULT_CAPACITY;
    //如果想要的最小容量大于最小扩容量，则使用想要的最小容量。
    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
/**
* 数组容量检查，不够时则进行扩容，只供类内部使用。
* 
* @param minCapacity    想要的最小容量
*/
private void ensureCapacityInternal(int minCapacity) {
    // 若elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA，则取minCapacity为DEFAULT_CAPACITY和参数minCapacity之间的最大值
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}
/**
* 数组容量检查，不够时则进行扩容，只供类内部使用
* 
* @param minCapacity 想要的最小容量
*/
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 确保指定的最小容量 > 数组缓冲区当前的长度  
    if (minCapacity - elementData.length > 0)
        //扩容
        grow(minCapacity);
}

/**
 * 分派给arrays的最大容量
 * 为什么要减去8呢？
 * 因为某些VM会在数组中保留一些头字，尝试分配这个最大存储容量，可能会导致array容量大于VM的limit，最终导致OutOfMemoryError。
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
* 扩容，保证ArrayList至少能存储minCapacity个元素
* 第一次扩容，逻辑为newCapacity = oldCapacity + (oldCapacity >> 1);即在原有的容量基础上增加一半。第一次扩容后，如果容量还是小于minCapacity，就将容量扩充为minCapacity。
* 
* @param minCapacity 想要的最小容量
*/
private void grow(int minCapacity) {
    // 获取当前数组的容量
    int oldCapacity = elementData.length;
    // 扩容。新的容量=当前容量+当前容量/2.即将当前容量增加一半。
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //如果扩容后的容量还是小于想要的最小容量
    if (newCapacity - minCapacity < 0)
        //将扩容后的容量再次扩容为想要的最小容量
        newCapacity = minCapacity;
    //如果扩容后的容量大于临界值，则进行大容量分配
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData,newCapacity);
}
/**
* 进行大容量分配
*/
private static int hugeCapacity(int minCapacity) {
    //如果minCapacity<0，抛出异常
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    //如果想要的容量大于MAX_ARRAY_SIZE，则分配Integer.MAX_VALUE，否则分配MAX_ARRAY_SIZE
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

看完了代码，可以对扩容方法总结如下：

1. 进行空间检查，决定是否进行扩容，以及确定最少需要的容量
2. 如果确定扩容，就执行grow(int minCapacity)，minCapacity为最少需要的容量
3. 第一次扩容，逻辑为newCapacity = oldCapacity + (oldCapacity >> 1);即在原有的容量基础上增加一半。
4. 第一次扩容后，如果容量还是小于minCapacity，就将容量扩充为minCapacity。
5. 对扩容后的容量进行判断，如果大于允许的最大容量MAX_ARRAY_SIZE，则将容量再次调整为MAX_ARRAY_SIZE。至此扩容操作结束。
6. **注意：扩容后是一个增加容量的新的数组，只是将原数据复制到新数组中，并非增加原数组的长度**

## add( int index, E element)

```java
/**
 * 在制定位置插入元素。当前位置的元素和index之后的元素向后移一位
 *
 * @param index 即将插入元素的位置
 * @param element 即将插入的元素
 * @throws IndexOutOfBoundsException 如果索引超出size
 */
public void add(int index, E element) {
    //越界检查
    rangeCheckForAdd(index);
    //确认list容量，如果不够，容量加1。注意：只加1，保证资源不被浪费
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 对数组进行复制处理，目的就是空出index的位置插入element，并将index后的元素位移一个位置
    System.arraycopy(elementData, index, elementData, index + 1,size - index);
    //将指定的index位置赋值为element
    elementData[index] = element;
    //实际容量+1
    size++;
}
```

从源码中可以看到，add(E e)有三个步骤：

1. 越界检查
2. 空间检查，如果有需要进行扩容
3. 插入元素

越界检查很简单

```java
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

空间检查和扩容的介绍在上面。

空间的问题解决后，插入过程就显得非常简单。

![image](/images/java-arraylist/java-arraylist-4.jpg)

add(int index, E e)需要先对元素进行移动，然后完成插入操作，也就意味着该方法有着**线性的时间复杂度，即O(n)。**

## remove( int index)

```java
/**
 * 删除list中位置为指定索引index的元素
 * 索引之后的元素向左移一位
 *
 * @param index 被删除元素的索引
 * @return 被删除的元素
 * @throws IndexOutOfBoundsException 如果参数指定索引index>=size，抛出一个越界异常
 */
public E remove(int index) {
    //检查索引是否越界。如果参数指定索引index>=size，抛出一个越界异常
    rangeCheck(index);
    //结构性修改次数+1
    modCount++;
    //记录索引为inde处的元素
    E oldValue = elementData(index);

    // 删除指定元素后，需要左移的元素个数
    int numMoved = size - index - 1;
    //如果有需要左移的元素，就移动（移动后，该删除的元素就已经被覆盖了）
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // size减一，然后将索引为size-1处的元素置为null。为了让GC起作用，必须显式的为最后一个位置赋null值
    elementData[--size] = null; // clear to let GC do its work

    //返回被删除的元素
    return oldValue;
}

/**
 * 越界检查。
 * 检查给出的索引index是否越界。
 * 如果越界，抛出运行时异常。
 * 这个方法并不检查index是否合法。比如是否为负数。
 * 如果给出的索引index>=size，抛出一个越界异常
 */
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

看完代码后，可以将ArrayList删除指定索引的元素的步骤总结为

1. 检查索引是否越界。如果参数指定索引index>=size，抛出一个越界异常
2. 将索引大于index的元素左移一位（左移后，该删除的元素就被覆盖了，相当于被删除了）。
3. 将索引为size-1处的元素置为null（为了让GC起作用）。

注意：为了让GC起作用，必须显式的为最后一个位置赋null值。上面代码中如果不手动赋null值，除非对应的位置被其他元素覆盖，否则原来的对象就一直不会被回收。

![image](/images/java-arraylist/java-arraylist-5.jpg)

## set( int index, E element)

```java
/**
 * 替换指定索引的元素
 *
 * @param 被替换元素的索引
 * @param element 即将替换到指定索引的元素
 * @return 返回被替换的元素
 * @throws IndexOutOfBoundsException 如果参数指定索引index>=size，抛出一个越界异常
 */
public E set(int index, E element) {
    //检查索引是否越界。如果参数指定索引index>=size，抛出一个越界异常
    rangeCheck(index);

    //记录被替换的元素
    E oldValue = elementData(index);
    //替换元素
    elementData[index] = element;
    //返回被替换的元素
    return oldValue;
}
```



参考：

https://blog.csdn.net/panweiwei1994/article/details/76760238