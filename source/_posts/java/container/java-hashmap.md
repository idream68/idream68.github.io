---
title: java HashMap
date: 2021-05-17 14:30:04
categories:
  - java
  - container
tags:
  - java
  - Map
---

参考的JDK版本为1.8。

# 数据结构

![image](/images/java-hashmap/java-hash-map-1.jpg)

从上图中可以很清楚的看到，HashMap的数据结构是数组+链表+红黑树（红黑树since JDK1.8）。我们常把数组中的每一个节点称为一个**桶**。当向桶中添加一个键值对时，首先计算键值对中key的hash值，以此确定插入数组中的位置，但是可能存在同一hash值的元素已经被放在数组同一位置了，这种现象称为**碰撞**，这时按照尾插法(jdk1.7及以前为头插法)的方式添加key-value到同一hash值的元素的后面，链表就这样形成了。当链表长度超过8(TREEIFY_THRESHOLD)时，链表就转换为红黑树。

# 顶部注释

> HashMap是Map接口基于哈希表的实现。这种实现提供了所有可选的Map操作，并允许key和value为null（除了HashMap是unsynchronized的和允许使用null外，HashMap和HashTable大致相同。）。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。
> 
> 此实现假设哈希函数在桶内适当地分布元素，为基本实现(get 和 put)提供了稳定的性能。迭代 collection 视图所需的时间与 HashMap 实例的“容量”（桶的数量）及其大小（键-值映射关系数）成比例。如果遍历操作很重要，就不要把初始化容量initial capacity设置得太高（或将加载因子load factor设置得太低），否则会严重降低遍历的效率。
> 
> HashMap有两个影响性能的重要参数：初始化容量initial capacity、加载因子load factor。容量是哈希表中桶的数量，初始容量只是哈希表在创建时的容量。加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度。initial capacity*load factor就是当前允许的最大元素数目，超过initial capacity*load factor之后，HashMap就会进行rehashed操作来进行扩容，扩容后的的容量为之前的两倍。
> 
> 通常，默认加载因子 (0.75) 在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查询成本（在大多数 HashMap类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少rehash操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生rehash 操作。
> 
> 如果很多映射关系要存储在 HashMap 实例中，则相对于按需执行自动的 rehash 操作以增大表的容量来说，使用足够大的初始容量创建它将使得映射关系能更有效地存储。
> 
> 注意，此实现不是同步的。如果多个线程同时访问一个哈希映射，而其中至少一个线程从结构上修改了该映射，则它必须保持外部同步。（结构上的修改是指添加或删除一个或多个映射关系的任何操作；仅改变与实例已经包含的键关联的值不是结构上的修改。）这一般通过对自然封装该映射的对象进行同步操作来完成。如果不存在这样的对象，则应该使用 Collections.synchronizedMap 方法来“包装”该映射。最好在创建时完成这一操作，以防止对映射进行意外的非同步访问，如下所示：
> 
> Map m = Collections.synchronizedMap(new HashMap(…));
> 
> 由所有此类的“collection 视图方法”所返回的迭代器都是fail-fast 的：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器本身的remove方法，其他任何时间任何方式的修改，迭代器都将抛出 ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不冒在将来不确定的时间发生任意不确定行为的风险。
> 
> 注意，迭代器的快速失败行为不能得到保证，一般来说，存在非同步的并发修改时，不可能作出任何坚决的保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测bug。
>
> 此类是 Java Collections Framework 的成员。
>
> @author Doug Lea
> @author Josh Bloch
> @author Arthur van Hoff
> @author Neal Gafter
> @see Object#hashCode()
> @see Collection
> @see Map
> @see TreeMap

从上面的内容中可以总结出以下几点：

- **底层**：HashMap是Map接口基于哈希表的实现。
- **是否允许null**：HashMap允许key和value为null。
- **是否有序**：HashMap不保证映射的顺序，特别是它不保证该顺序恒久不变。
- **何时rehash**：超出当前允许的最大容量。initial capacity*load factor就是当前允许的最大元素数目，超过initial capacity*load factor之后，HashMap就会进行rehashed操作来进行扩容，扩容后的的容量为之前的两倍。
- **初始化容量对性能的影响**：不应设置地太小，设置地小虽然可以节省空间，但会频繁地进行rehash操作。rehash会影响性能。总结：小了会增大时间开销（频繁rehash）；大了会增大空间开销（占用了更多空间）和时间开销（影响遍历）。
- **加载因子对性能的影响**：加载因子过高虽然减少了空间开销，但同时也增加了查询成本。0.75是个折中的选择。总结：小了会增大时间开销（频繁rehash）；大了会也增大时间开销（影响遍历）。
- **是否同步**：HashMap不是同步的。
- **迭代器**：迭代器是fast-fail的。

# 定义

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable
```

- HashMap<K,V>：HashMap是以key-value形式存储数据的。
- extends AbstractMap<K,V>：继承了AbstractMap，大大减少了实现Map接口时需要的工作量。
- implements Map<K,V>：实现了Map，提供了所有可选的Map操作。
- implements Cloneable：表明其可以调用clone()方法来返回实例的field-for-field拷贝。
- implements Serializable：表明该类是可以序列化的。

下图是HashMap的类结构层次图。

![image](/images/java-hashmap/java-hash-map-2.jpg)

# 静态全局变量

```java
/**
 * 默认初始化容量，值为16
 * 必须是2的n次幂.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * 最大容量, 容量不能超出这个值。如果一个更大的初始化容量在构造函数中被指定，将被MAXIMUM_CAPACITY替换.
 * 必须是2的倍数。最大容量为1<<30，即2的30次方。
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * 默认的加载因子。
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * 将链表转化为红黑树的临界值。
 * 当添加一个元素被添加到有至少TREEIFY_THRESHOLD个节点的桶中，桶中链表将被转化为树形结构。
 * 临界值最小为8
 */
static final int TREEIFY_THRESHOLD = 8;

/**
 * 恢复成链式结构的桶大小临界值
 * 小于TREEIFY_THRESHOLD，临界值最大为6
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * 桶可能被转化为树形结构的最小容量。当哈希表的大小超过这个阈值，才会把链式结构转化成树型结构，否则仅采取扩容来尝试减少冲突。
 * 应该至少4*TREEIFY_THRESHOLD来避免扩容和树形结构化之间的冲突。
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```

# 静态内部类Node

```java
/**
 * HashMap的节点类型。既是HashMap底层数组的组成元素，又是每个单向链表的组成元素
 */
static class Node<K,V> implements Map.Entry<K,V> {
    //key的哈希值
    final int hash;
    final K key;
    V value;
    //指向下个节点的引用
    Node<K,V> next;
    //构造函数
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

# 核心方法

## get( Object key)

```java
/**
 * 返回指定的key映射的value，如果value为null，则返回null。
 *
 * @see #put(Object, Object)
 */
public V get(Object key) {
    Node<K,V> e;
    //如果通过key获取到的node为null，则返回null，否则返回node的value。getNode方法的实现就在下面。
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

从源码中可以看到，get(E e)可以分为三个步骤：

1. 通过hash(Object key)方法计算key的哈希值hash。
2. 通过getNode( int hash, Object key)方法获取node。
3. 如果node为null，返回null，否则返回node.value。
先来看看哈希值是如何计算的。

不管增加、删除、查找键值对，定位到哈希桶数组的位置都是很关键的第一步。计算位置的方法如下

```java
(n - 1) & hash
```

其中的n为数组的长度，hash为hash(key)计算得到的值。

```java
/**
 * 计算key的哈希值。
 */
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

从代码中可以看到，计算位置分为三步，第一步，取key的hashCode，第二步，key的hashCode高16位异或低16位，第三步，将第一步和第二部得到的结果进行取模运算。

![image](/images/java-hashmap/java-hash-map-3.jpg)

看到这里有个疑问，**为什么要做异或运算？**

设想一下，如果n很小，假设为16的话，那么n-1即为15（0000 0000 0000 0000 0000 0000 0000 1111），这样的值如果跟hashCode()直接做与操作，实际上只使用了哈希值的后4位。如果当哈希值的高位变化很大，低位变化很小，这样很容易造成碰撞，所以把高低位都参与到计算中，从而解决了这个问题，而且也不会有太大的开销。

看完哈希值是如何计算之后，看看如何通过key和hash获取node。

## getNode( int hash, Object key)

```java
/**
 * 根据key的哈希值和key获取对应的节点
 * 
 * @param hash 指定参数key的哈希值
 * @param key 指定参数key
 * @return 返回node，如果没有则返回null
 */
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //如果哈希表不为空，而且key对应的桶上不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //如果桶中的第一个节点就和指定参数hash和key匹配上了
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            //返回桶中的第一个节点
            return first;
        //如果桶中的第一个节点没有匹配上，而且有后续节点
        if ((e = first.next) != null) {
            //如果当前的桶采用红黑树，则调用红黑树的get方法去获取节点
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            //如果当前的桶不采用红黑树，即桶中节点结构为链式结构
            do {
                //遍历链表，直到key匹配
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    //如果哈希表为空，或者没有找到节点，返回null
    return null;
}
```

### get方法总结

从源码中可以看到，get(E e)可以分为三个步骤：

1. 通过hash(Object key)方法计算key的哈希值hash。
2. 通过getNode( int hash, Object key)方法获取node。
3. 如果node为null，返回null，否则返回node.value。

hash方法又可分为三步：
1. 取key的hashCode第二步
2. key的hashCode高16位异或低16位
3. 将第一步和第二部得到的结果进行取模运算。

getNode方法又可分为以下几个步骤：

1. 如果哈希表为空，或key对应的桶为空，返回null
2. 如果桶中的第一个节点就和指定参数hash和key匹配上了，返回这个节点。
3. 如果桶中的第一个节点没有匹配上，而且有后续节点
   1. 如果当前的桶采用红黑树，则调用红黑树的get方法去获取节点
   2. 如果当前的桶不采用红黑树，即桶中节点结构为链式结构，遍历链表，直到key匹配
4. 找到节点返回值，否则返回null。

## put( K key, V value)

```java
/**
 * 将指定参数key和指定参数value插入map中，如果key已经存在，那就替换key对应的value
 * 
 * @param key 指定key
 * @param value 指定value
 * @return 如果value被替换，则返回旧的value，否则返回null。当然，可能key对应的value就是null。
 */
public V put(K key, V value) {
    //putVal方法的实现就在下面
    return putVal(hash(key), key, value, false, true);
}
```

从源码中可以看到，put(K key, V value)可以分为三个步骤：

1. 通过hash(Object key)方法计算key的哈希值。
2. 通过putVal(hash(key), key, value, false, true)方法实现功能。
3. 返回putVal方法返回的结果。

哈希值是如何计算的上面已经写了。下面看看putVal方法是如何实现的。

## putVal( int hash, K key, V value, boolean onlyIfAbsent,boolean evict)

```java
/**
 * Map.put和其他相关方法的实现需要的方法
 * 
 * @param hash 指定参数key的哈希值
 * @param key 指定参数key
 * @param value 指定参数value
 * @param onlyIfAbsent 如果为true，即使指定参数key在map中已经存在，也不会替换value
 * @param evict 如果为false，数组table在创建模式中
 * @return 如果value被替换，则返回旧的value，否则返回null。当然，可能key对应的value就是null。
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //如果哈希表为空，调用resize()创建一个哈希表，并用变量n记录哈希表长度
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //如果指定参数hash在表中没有对应的桶，即为没有碰撞
    if ((p = tab[i = (n - 1) & hash]) == null)
        //直接将键值对插入到map中即可
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        //如果碰撞了，且桶中的第一个节点就匹配了
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            //将桶中的第一个节点记录起来
            e = p;
        //如果桶中的第一个节点没有匹配上，且桶内为红黑树结构，则调用红黑树对应的方法插入键值对
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //不是红黑树结构，那么就肯定是链式结构
        else {
            //遍历链式结构
            for (int binCount = 0; ; ++binCount) {
                //如果到了链表尾部
                if ((e = p.next) == null) {
                    //在链表尾部插入键值对
                    p.next = newNode(hash, key, value, null);
                    //如果链的长度大于TREEIFY_THRESHOLD这个临界值，则把链变为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    //跳出循环
                    break;
                }
                //如果找到了重复的key，判断链表中结点的key值与插入的元素的key值是否相等，如果相等，跳出循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                //用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        //如果key映射的节点不为null
        if (e != null) { // existing mapping for key
            //记录节点的vlaue
            V oldValue = e.value;
            //如果onlyIfAbsent为false，或者oldValue为null
            if (!onlyIfAbsent || oldValue == null)
                //替换value
                e.value = value;
            //访问后回调
            afterNodeAccess(e);
            //返回节点的旧值
            return oldValue;
        }
    }
    //结构型修改次数+1
    ++modCount;
    //判断是否需要扩容
    if (++size > threshold)
        resize();
    //插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

putVal方法可以分为下面的几个步骤：

1. 如果哈希表为空，调用resize()创建一个哈希表。
2. 如果指定参数hash在表中没有对应的桶，即为没有碰撞，直接将键值对插入到哈希表中即可。
3. 如果有碰撞，遍历桶，找到key映射的节点
   1. 桶中的第一个节点就匹配了，将桶中的第一个节点记录起来。
   2. 如果桶中的第一个节点没有匹配，且桶中结构为红黑树，则调用红黑树对应的方法插入键值对。
   3. 如果不是红黑树，那么就肯定是链表。遍历链表，如果找到了key映射的节点，就记录这个节点，退出循环。如果没有找到，在链表尾部插入节点。插入后，如果链的长度大于TREEIFY_THRESHOLD这个临界值，则使用treeifyBin方法把链表转为红黑树。
4. 如果找到了key映射的节点，且节点不为null
   1. 记录节点的vlaue。
   2. 如果参数onlyIfAbsent为false，或者oldValue为null，替换value，否则不替换。
   3. 返回记录下来的节点的value。
5. 如果没有找到key映射的节点（2、3步中讲了，这种情况会插入到hashMap中），插入节点后size会加1，这时要检查size是否大于临界值threshold，如果大于会使用resize方法进行扩容。

## resize()
向hashMap对象里不停的添加元素，而HashMap对象内部的数组无法装载更多的元素时，hashMap就需要扩大数组的长度，以便能装入更多的元素。当然数组是无法自动扩容的，扩容方法使用一个新的数组代替已有的容量小的数组。

resize方法非常巧妙，因为每次扩容都是翻倍，与原来计算（n-1）&hash的结果相比，节点要么就在原来的位置，要么就被分配到“原位置+旧容量”这个位置。

```java
/**
 * 对table进行初始化或者扩容。
 * 如果table为null，则对table进行初始化
 * 如果对table扩容，因为每次扩容都是翻倍，与原来计算（n-1）&hash的结果相比，节点要么就在原来的位置，要么就被分配到“原位置+旧容量”这个位置。
 */
final Node<K,V>[] resize() {
    //新建oldTab数组保存扩容前的数组table
    Node<K,V>[] oldTab = table;
    //使用变量oldCap扩容前table的容量
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    //保存扩容前的临界值
    int oldThr = threshold;
    int newCap, newThr = 0;
    //如果扩容前的容量 > 0
    if (oldCap > 0) {
        //如果当前容量>=MAXIMUM_CAPACITY
        if (oldCap >= MAXIMUM_CAPACITY) {
            //扩容临界值提高到正无穷
            threshold = Integer.MAX_VALUE;
            //无法进行扩容，返回原来的数组
            return oldTab;
        }
        //如果现在容量的两倍小于MAXIMUM_CAPACITY且现在的容量大于DEFAULT_INITIAL_CAPACITY
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&oldCap >= DEFAULT_INITIAL_CAPACITY)
            //临界值变为原来的2倍
            newThr = oldThr << 1; 
    }//如果旧容量 <= 0，而且旧临界值 > 0
    else if (oldThr > 0) 
        //数组的新容量设置为老数组扩容的临界值
        newCap = oldThr;
    else {//如果旧容量 <= 0，且旧临界值 <= 0，新容量扩充为默认初始化容量，新临界值为DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {//在当上面的条件判断中，只有oldThr > 0成立时，newThr == 0
        //ft为临时临界值，下面会确定这个临界值是否合法，如果合法，那就是真正的临界值
        float ft = (float)newCap * loadFactor;
        //当新容量< MAXIMUM_CAPACITY且ft < (float)MAXIMUM_CAPACITY，新的临界值为ft，否则为Integer.MAX_VALUE
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    //将扩容后hashMap的临界值设置为newThr
    threshold = newThr;
    //创建新的table，初始化容量为newCap
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    //修改hashMap的table为新建的newTab
    table = newTab;
    //如果旧table不为空，将旧table中的元素复制到新的table中
    if (oldTab != null) {
        //遍历旧哈希表的每个桶，将旧哈希表中的桶复制到新的哈希表中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            //如果旧桶不为null，使用e记录旧桶
            if ((e = oldTab[j]) != null) {
                //将旧桶置为null
                oldTab[j] = null;
                //如果旧桶中只有一个node
                if (e.next == null)
                    //将e也就是oldTab[j]放入newTab中e.hash & (newCap - 1)的位置
                    newTab[e.hash & (newCap - 1)] = e;
                //如果旧桶中的结构为红黑树
                else if (e instanceof TreeNode)
                    //将树中的node分离
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { //如果旧桶中的结构为链表，讲旧桶中元素按照hash值分装到高位和低位两个桶中
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    //遍历整个链表中的节点
                    do {
                        next = e.next;
                        //
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

从代码中可以看到，扩容很耗性能。所以在使用HashMap的时候，先估算map的大小，初始化的时候给一个大致的数值，避免map进行频繁的扩容。

看完代码后，可以将resize的步骤总结为

1. 计算扩容后的容量，临界值。
2. 将hashMap的临界值修改为扩容后的临界值
3. 根据扩容后的容量新建数组，然后将hashMap的table的引用指向新数组。
4. 将旧数组的元素复制到table中。

## remove( Object key)

```java
/**
 * 删除hashMap中key映射的node
 *
 * @param  key 参数key
 * @return 如果没有映射到node，返回null，否则返回对应的value。
 */
public V remove(Object key) {
    Node<K,V> e;
    //根据key来删除node。removeNode方法的具体实现在下面
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
```

从源码中可以看到，remove方法的实现可以分为三个步骤：

1. 通过hash(Object key)方法计算key的哈希值。
2. 通过removeNode方法实现功能。
3. 返回被删除的node的value。

下面看看removeNode方法的具体实现

## removeNode( int hash, Object key, Object value,boolean matchValue, boolean movable)

```java
/**
 * Map.remove和相关方法的实现需要的方法
 * 删除node
 * 
 * @param hash key的哈希值
 * @param key 参数key
 * @param value 如果matchValue为true，则value也作为确定被删除的node的条件之一，否则忽略
 * @param matchValue 如果为true，则value也作为确定被删除的node的条件之一
 * @param movable 如果为false，删除node时不会删除其他node
 * @return 返回被删除的node，如果没有node被删除，则返回null（针对红黑树的删除方法）
 */
final Node<K,V> removeNode(int hash, Object key, Object value,boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    //如果数组table不为空且key映射到的桶不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        //
        Node<K,V> node = null, e; K k; V v;
        //如果桶上第一个node的就是要删除的node
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            //记录桶上第一个node
            node = p;
        else if ((e = p.next) != null) {//如果桶内不止一个node
            if (p instanceof TreeNode)//如果桶内的结构为红黑树
                //记录key映射到的node
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {//如果桶内的结构为链表
                do {//遍历链表，找到key映射到的node
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        //记录key映射到的node
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        //如果得到的node不为null且(matchValue为false||node.value和参数value匹配)
        if (node != null && (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) {
            //如果桶内的结构为红黑树
            if (node instanceof TreeNode)
                //使用红黑树的删除方法删除node
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)//如果桶的第一个node的就是要删除的node
                //删除node
                tab[index] = node.next;
            else//如果桶内的结构为链表，使用链表删除元素的方式删除node
                p.next = node.next;
            //结构性修改次数+1
            ++modCount;
            //哈希表大小-1
            --size;
            afterNodeRemoval(node);
            //返回被删除的node
            return node;
        }
    }
    //如果数组table为空或key映射到的桶为空，返回null。
    return null;
}
```

看完代码后，可以将removeNode方法的步骤总结为

1. 如果数组table为空或key映射到的桶为空，返回null。
2. 如果key映射到的桶上第一个node的就是要删除的node，记录下来。
3. 如果桶内不止一个node，且桶内的结构为红黑树，记录key映射到的node。
4. 桶内的结构不为红黑树，那么桶内的结构就肯定为链表，遍历链表，找到key映射到的node，记录下来。
5. 如果被记录下来的node不为null，删除node，size-1被删除。
6. 返回被删除的node。

# 静态公用方法

## hash( Object key)

该方法已在【核心方法】中详细讲解了，这里不做重复讲解。

## comparableClassFor( Object x)

```java
/**
 * 如果参数x实现了Comparable接口，返回参数x的类名，否则返回null
 */
static Class<?> comparableClassFor(Object x) {
    if (x instanceof Comparable) {
        Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
        if ((c = x.getClass()) == String.class) // bypass checks
            return c;
        if ((ts = c.getGenericInterfaces()) != null) {
            for (int i = 0; i < ts.length; ++i) {
                if (((t = ts[i]) instanceof ParameterizedType) &&
                    ((p = (ParameterizedType)t).getRawType() ==
                     Comparable.class) &&
                    (as = p.getActualTypeArguments()) != null &&
                    as.length == 1 && as[0] == c) // type arg is c
                    return c;
            }
        }
    }
    return null;
}
```

## compareComparables( Class<?> kc, Object k, Object x)

```java
/**
 * 如果x的类型为kc，则返回k.compareTo(x)，否则返回0.
 */
@SuppressWarnings({"rawtypes","unchecked"}) // for cast to Comparable
static int compareComparables(Class<?> kc, Object k, Object x) {
    return (x == null || x.getClass() != kc ? 0 :
            ((Comparable)k).compareTo(x));
}
```



## tableSizeFor( int cap)

```java
/**
 * 返回大于等于cap的最小的二次幂数值。
 */
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

# 域

```java
/**
 * 存储键值对的数组，一般是2的幂
 */
transient Node<K,V>[] table;

/**
 * 键值对缓存，它们的映射关系集合保存在entrySet中。即使Key在外部修改导致hashCode变化，缓存中还可以找到映射关系
 */
transient Set<Map.Entry<K,V>> entrySet;

/**
 * 键值对的实际个数
 */
transient int size;

/**
 * 记录HashMap被修改结构的次数。
 * 修改包括改变键值对的个数或者修改内部结构，比如rehash
 * 这个域被用作HashMap的迭代器的fail-fast机制中（参考ConcurrentModificationException）
 */
transient int modCount;

/**
 * 扩容的临界值，通过capacity * load factor可以计算出来。超过这个值HashMap将进行扩容
 * @serial
 */

int threshold;

/**
 * 加载因子
 *
 * @serial
 */
final float loadFactor;
```

# 构造函数
## HashMap( int initialCapacity, float loadFactor)

```java
/**
 * 使用指定的初始化容量initial capacity 和加载因子load factor构造一个空HashMap
 *
 * @param  initialCapacity 初始化容量
 * @param  loadFactor      加载因子
 * @throws IllegalArgumentException 如果指定的初始化容量为负数或者加载因子为非正数。
 */
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

## HashMap( int initialCapacity)

```java
/**
 * 使用指定的初始化容量initial capacity和默认加载因子DEFAULT_LOAD_FACTOR（0.75）构造一个空HashMap
 *
 * @param  initialCapacity 初始化容量
 * @throws IllegalArgumentException 如果指定的初始化容量为负数
 */
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

## HashMap()

```java
/**
 * 使用指定的初始化容量（16）和默认加载因子DEFAULT_LOAD_FACTOR（0.75）构造一个空HashMap
 */
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

## HashMap( Map<? extends K, ? extends V>m)

```java
/**
 * 使用指定Map m构造新的HashMap。使用指定的初始化容量（16）和默认加载因子DEFAULT_LOAD_FACTOR（0.75）
 * @param   m 指定的map
 * @throws  NullPointerException 如果指定的map是null
 */
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

# 常用方法

## putMapEntries(Map<? extends K, ? extends V> m, boolean evict)

```java
/**
 * Map.putAll and Map constructor的实现需要的方法。
 * 将m的键值对插入本map中
 * 
 * @param m the map
 * @param evict 初始化map时使用false，否则使用true
 */
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    //如果参数map不为空
    if (s > 0) {
        //如果table没有初始化
        if (table == null) { // pre-size
            //前面讲到，initial capacity*load factor就是当前hashMap允许的最大元素数目。那么不难理解，s/loadFactor+1即为应该初始化的容量。
            float ft = ((float)s / loadFactor) + 1.0F;
            //如果ft小于最大容量MAXIMUM_CAPACITY，则容量为ft，否则容量为最大容量MAXIMUM_CAPACITY
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            //如果容量大于临界值
            if (t > threshold)
                //根据容量初始化临界值
                threshold = tableSizeFor(t);
        }
        //table已经初始化，并且map的大小大于临界值
        else if (s > threshold)
            //扩容处理
            resize();
        //将map中所有键值对添加到hashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            //putVal方法的实现在下面
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

## size()

```java
/**
 * 返回map中键值对映射的个数
 * 
 * @return map中键值对映射的个数
 */
public int size() {
    return size;
}
```

## isEmpty()

```java
/**
 * 如果map中没有键值对映射，返回true
 * 
 * @return <如果map中没有键值对映射，返回true
 */
public boolean isEmpty() {
    return size == 0;
}
```

## get( Object key)

该方法已在【核心方法】中详细讲解了，这里不做重复讲解。

## getNode( int hash, Object key)

该方法已在【核心方法】中详细讲解了，这里不做重复讲解。

## containsKey( Object key)

```java
/**
 * 如果map中含有key为指定参数key的键值对，返回true
 * 
 * @param   key   指定参数key
 * @return 如果map中含有key为指定参数key的键值对，返回true
 */
public boolean containsKey(Object key) {
    return getNode(hash(key), key) != null;
}
```

## put( K key, V value)

该方法已在【核心方法】中详细讲解了，这里不做重复讲解。

## putVal( int hash, K key, V value, boolean onlyIfAbsent,boolean evict)

该方法已在【核心方法】中详细讲解了，这里不做重复讲解。

## 扩容resize()

该方法已在【核心方法】中详细讲解了，这里不做重复讲解。

## treeifyBin( Node<K,V>[] tab, int hash)

```java
/**
 * 将链表转化为红黑树
 */
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    //如果桶数组table为空，或者桶数组table的长度小于MIN_TREEIFY_CAPACITY，不符合转化为红黑树的条件
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        //扩容
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {//如果符合转化为红黑树的条件，而且hash对应的桶不为null
        TreeNode<K,V> hd = null, tl = null;
        //遍历链表
        do {
            //替换链表node为树node，建立双向链表
            TreeNode<K,V> p = replacementTreeNode(e, null);
            //
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        //遍历链表插入每个节点到红黑树
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

## putAll( Map<? extends K, ? extends V> m)

```java
/**
 * 将参数map中的所有键值对映射插入到hashMap中，如果有碰撞，则覆盖value。
 * @param m 参数map
 * @throws NullPointerException 如果map为null
 */
public void putAll(Map<? extends K, ? extends V> m) {
    putMapEntries(m, true);
}
```

## remove( Object key)

该方法已在【核心方法】中详细讲解了，这里不做重复讲解。

## removeNode( int hash, Object key, Object value,boolean matchValue, boolean movable)

该方法已在【核心方法】中详细讲解了，这里不做重复讲解。

## clear()

```java
/**
 * 删除map中所有的键值对
 */
public void clear() {
    Node<K,V>[] tab;
    modCount++;
    if ((tab = table) != null && size > 0) {
        size = 0;
        for (int i = 0; i < tab.length; ++i)
            tab[i] = null;
    }
}
```

## containsValue( Object value)

```java
/**
 * 如果hashMap中的键值对有一对或多对的value为参数value，返回true
 *
 * @param value 参数value
 * @return 如果hashMap中的键值对有一对或多对的value为参数value，返回true
 */
public boolean containsValue(Object value) {
    Node<K,V>[] tab; V v;
    //
    if ((tab = table) != null && size > 0) {
        //遍历数组table
        for (int i = 0; i < tab.length; ++i) {
            //遍历桶中的node
            for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                if ((v = e.value) == value ||
                    (value != null && value.equals(v)))
                    return true;
            }
        }
    }
    return false;
}
```

# 视图

## keySet()

```java
/**
 * 返回hashMap中所有key的视图。
 * 改变hashMap会影响到set，反之亦然。
 * 如果当迭代器迭代set时，hashMap被修改(除非是迭代器自己的remove()方法)，迭代器的结果是不确定的。
 * set支持元素的删除，通过Iterator.remove、Set.remove、removeAll、retainAll、clear操作删除hashMap中对应的键值对。不支持add和addAll方法。
 *
 * @return 返回hashMap中所有key的set视图
 */
public Set<K> keySet() {
    //
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
}

/**
 * 内部类KeySet
 */
final class KeySet extends AbstractSet<K> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator<K> iterator()     { return new KeyIterator(); }
    public final boolean contains(Object o) { return containsKey(o); }
    public final boolean remove(Object key) {
        return removeNode(hash(key), key, null, false, true) != null;
    }
    public final Spliterator<K> spliterator() {
        return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
    }
    public final void forEach(Consumer<? super K> action) {
        Node<K,V>[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e.key);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
}
```

## values()

```java
/**
 * 返回hashMap中所有value的collection视图
 * 改变hashMap会改变collection，反之亦然。
 * 如果当迭代器迭代collection时，hashMap被修改（除非是迭代器自己的remove()方法），迭代器的结果是不确定的。
 * collection支持元素的删除，通过Iterator.remove、Collection.remove、removeAll、retainAll、clear操作删除hashMap中对应的键值对。不支持add和addAll方法。
 *
 * @return 返回hashMap中所有key的collection视图
 */
public Collection<V> values() {
    Collection<V> vs = values;
    if (vs == null) {
        vs = new Values();
        values = vs;
    }
    return vs;
}

/**
 * 内部类Values
 */
final class Values extends AbstractCollection<V> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator<V> iterator()     { return new ValueIterator(); }
    public final boolean contains(Object o) { return containsValue(o); }
    public final Spliterator<V> spliterator() {
        return new ValueSpliterator<>(HashMap.this, 0, -1, 0, 0);
    }
    public final void forEach(Consumer<? super V> action) {
        Node<K,V>[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e.value);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
}
```

## entrySet()

```java
/**
 * 返回hashMap中所有键值对的set视图
 * 改变hashMap会影响到set，反之亦然。
 * 如果当迭代器迭代set时，hashMap被修改(除非是迭代器自己的remove()方法)，迭代器的结果是不确定的。
 * set支持元素的删除，通过Iterator.remove、Set.remove、removeAll、retainAll、clear操作删除hashMap中对应的键值对。不支持add和addAll方法。
 *
 * @return 返回hashMap中所有键值对的set视图
 */
public Set<Map.Entry<K,V>> entrySet() {
    Set<Map.Entry<K,V>> es;
    return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
}

/**
 * 内部类EntrySet
 */
final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator<Map.Entry<K,V>> iterator() {
        return new EntryIterator();
    }
    public final boolean contains(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>) o;
        Object key = e.getKey();
        Node<K,V> candidate = getNode(hash(key), key);
        return candidate != null && candidate.equals(e);
    }
    public final boolean remove(Object o) {
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>) o;
            Object key = e.getKey();
            Object value = e.getValue();
            return removeNode(hash(key), key, value, true, true) != null;
        }
        return false;
    }
    public final Spliterator<Map.Entry<K,V>> spliterator() {
        return new EntrySpliterator<>(HashMap.this, 0, -1, 0, 0);
    }
    public final void forEach(Consumer<? super Map.Entry<K,V>> action) {
        Node<K,V>[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
}
```

# JDK8重写的方法

## getOrDefault( Object key, V defaultValue)

```java
/**
 * 通过key映射到对应node，如果没映射到则返回默认值defaultValue
 *
 * @return key映射到对应的node，如果没映射到则返回默认值defaultValue
 */
@Override
public V getOrDefault(Object key, V defaultValue) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? defaultValue : e.value;
}
```

## putIfAbsent( K key, V value)

```java
/**
 * 在hashMap中插入参数key和value组成的键值对，如果key在hashMap中已经存在，不替换value
 *
 * @return 如果key在hashMap中不存在，返回旧value
 */
@Override
public V putIfAbsent(K key, V value) {
    return putVal(hash(key), key, value, true, true);
}
```

## remove( Object key, Object value)

```java
/**
 * 删除hashMap中key为参数key，value为参数value的键值对。如果桶中结构为树，则级联删除
 *
 * @return 删除成功，返回true
 */
@Override
public boolean remove(Object key, Object value) {
    return removeNode(hash(key), key, value, true, true) != null;
}
```

## replace( K key, V oldValue, V newValue)

```java
/**
 * 使用newValue替换key和oldValue映射到的键值对中的value
 * 
 * @return 替换成功，返回true
 */
@Override
public boolean replace(K key, V oldValue, V newValue) {
    Node<K,V> e; V v;
    if ((e = getNode(hash(key), key)) != null &&
        ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
        e.value = newValue;
        afterNodeAccess(e);
        return true;
    }
    return false;
}
```

## replace( K key, V value)

```java
/**
 * 使用参数value替换key映射到的键值对中的value
 * 
 * @return 替换成功，返回true
 */
@Override
public V replace(K key, V value) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) != null) {
        V oldValue = e.value;
        e.value = value;
        afterNodeAccess(e);
        return oldValue;
    }
    return null;
}
```

## clone()

```java
/**
 * 浅拷贝。
 * clone方法虽然生成了新的HashMap对象，新的HashMap中的table数组虽然也是新生成的，但是数组中的元素还是引用以前的HashMap中的元素。
 * 这就导致在对HashMap中的元素进行修改的时候，即对数组中元素进行修改，会导致原对象和clone对象都发生改变，但进行新增或删除就不会影响对方，因为这相当于是对数组做出的改变，clone对象新生成了一个数组。
 *
 * @return hashMap的浅拷贝
 */
@SuppressWarnings("unchecked")
@Override
public Object clone() {
    HashMap<K,V> result;
    try {
        result = (HashMap<K,V>)super.clone();
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
    result.reinitialize();
    result.putMapEntries(this, false);
    return result;
}
```

## loadFactor()

```java
// These methods are also used when serializing HashSets
final float loadFactor() { return loadFactor; }
```

## capacity()

```java
// These methods are also used when serializing HashSets
final int capacity() {
    return (table != null) ? table.length :
        (threshold > 0) ? threshold :
        DEFAULT_INITIAL_CAPACITY;
}
```

## writeObject( java.io.ObjectOutputStream s)

```java
/**
 * 序列化hashMap到ObjectOutputStream中
 * 将hashMap的总容量capacity、实际容量size、键值对映射写入到ObjectOutputStream中。键值对映射序列化时是无序的。
 */
private void writeObject(java.io.ObjectOutputStream s)
    throws IOException {
    int buckets = capacity();
    // Write out the threshold, loadfactor, and any hidden stuff
    s.defaultWriteObject();
    //写入总容量
    s.writeInt(buckets);
    //写入实际容量
    s.writeInt(size);
    //写入键值对
    internalWriteEntries(s);
}

// 写入hashMap键值对到ObjectOutputStream中
void internalWriteEntries(java.io.ObjectOutputStream s) throws IOException {
    Node<K,V>[] tab;
    if (size > 0 && (tab = table) != null) {
        for (int i = 0; i < tab.length; ++i) {
            for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                s.writeObject(e.key);
                s.writeObject(e.value);
            }
        }
    }
}
```

## readObject( java.io.ObjectInputStream s)

```java
/**
 * 到ObjectOutputStream中读取hashMap
 * 将hashMap的总容量capacity、实际容量size、键值对映射读取出来
 */
private void readObject(java.io.ObjectInputStream s) throws IOException, ClassNotFoundException {
    // 将hashMap的总容量capacity、实际容量size、键值对映射读取出来
    s.defaultReadObject();
    //重置hashMap
    reinitialize();
    //如果加载因子不合法，抛出异常
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new InvalidObjectException("Illegal load factor: " + loadFactor);
    //读出桶的数量，忽略
    s.readInt();                // Read and ignore number of buckets
    //读出实际容量size
    int mappings = s.readInt(); // Read number of mappings (size)
    //如果读出的实际容量size小于0，抛出异常
    if (mappings < 0)
        throw new InvalidObjectException("Illegal mappings count: " + mappings);
    else if (mappings > 0) { // (if zero, use defaults)
        //调整hashMap大小
        // Size the table using given load factor only if within range of 0.25...4.0。为什么？
        // 加载因子
        float lf = Math.min(Math.max(0.25f, loadFactor), 4.0f);
        //初步得到的总容量，后续还会处理
        float fc = (float)mappings / lf + 1.0f;
        //处理初步得到的容量，确认最终的总容量
        int cap = ((fc < DEFAULT_INITIAL_CAPACITY) ?
                   DEFAULT_INITIAL_CAPACITY :
                   (fc >= MAXIMUM_CAPACITY) ?
                   MAXIMUM_CAPACITY :
                   tableSizeFor((int)fc));
        //计算临界值，得到初步的临界值
        float ft = (float)cap * lf;
        //得到最终的临界值
        threshold = ((cap < MAXIMUM_CAPACITY && ft < MAXIMUM_CAPACITY) ? (int)ft : Integer.MAX_VALUE);
        @SuppressWarnings({"rawtypes","unchecked"})
        //新建桶数组table
            Node<K,V>[] tab = (Node<K,V>[])new Node[cap];
        table = tab;

        // 读出key和value，并组成键值对插入hashMap中
        for (int i = 0; i < mappings; i++) {
            @SuppressWarnings("unchecked")
                K key = (K) s.readObject();
            @SuppressWarnings("unchecked")
                V value = (V) s.readObject();
            putVal(hash(key), key, value, false, false);
        }
    }
}
```

原文链接：https://blog.csdn.net/panweiwei1994/article/details/77244920