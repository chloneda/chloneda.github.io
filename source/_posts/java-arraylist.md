---
title: Java ArrayList源码分析
tags: Java
categories: Java
keywords: Java
comments: false
---

**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)



# 前言

**Java系列，尽量采用通俗易懂、循序渐进的方式，让大家真正理解JDK源码的精髓！**

关于JDK源码中的ArrayList类，官方文档这样描述：

> **Resizable-array** implementation of the List interface.  Implements all optional list operations, and permits all elements, including null. In addition to implementing the List interface, this class provides methods to manipulate the size of the array that is used internally to store the list. (This class is roughly equivalent to Vector, except that it is unsynchronized.)

ArrayList底层实现是数组，自然就具备了数组的基本性质：ArrayList查找、更新操作效率高，速度快；插入、删除操作时需要移动大部分元素，性能较低。

ArrayList是一种相对比较简单的数据结构，它的**基本问题**是：
- 是否允许空?
- 是否允许重复数据?
- 是否有序?
- 是否线程安全?

这几个问题上面的官方文档已经说得很清楚了！ArrayList可调整大小的数组实现，本身是有序的，并允许添加任何元素，包括null，同时与Vector大致等效，但不是线程安全的！

值得注意的是，ArrayList的**核心问题**是：
- 如何进行自动扩容，实现**动态数组**功能?

我们循序渐进地来聊聊这个核心问题吧！



# ArrayList

我们先来看看 **ArrayList** 类有哪些属性！
```
// ArrayList默认初始容量为10
private static final int DEFAULT_CAPACITY = 10;
// 空数组，当传入的容量为0的时候使用，通过new ArrayList(0)创建时用的是这个空数组。
private static final Object[] EMPTY_ELEMENTDATA = new Object[0];
// 空数组，这种是通过new ArrayList()创建时用的是这个空数组，与EMPTY_ELEMENTDATA的区别是在添加第一个元素时使用这个空数组的会初始化为DEFAULT_CAPACITY（10）个元素。
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = new Object[0];
// 存储元素的数组,真正存放元素的地方，使用transient是为了不序列化这个字段。
transient Object[] elementData;
// ArrayList中真正存储元素的个数
private int size;
// ArrayList容量的最大值
private static final int MAX_ARRAY_SIZE = 2147483639;
```

从 **transient Object[] elementData**，这个语句可以看到，ArrayList类是Object类型实现的数组，初始化数组时默认容量为10。接着我们看一段简单的代码：
```
ArrayList<String> list = new ArrayList<String>();
list.add("月亮1");
list.add("地球2");
list.add("太阳3");
list.remove(0);
```

在idea中采用Debug模式执行这几条语句时，是这么变化的：
![java-arraylist-element](/uploads/java-arraylist-element.png)

可以看到，add操作直接将数组元素添加在数据末尾，remove操作删除index为0的节点后，会将后面元素移到index为0的地方。



# add()函数

在ArrayList中，当我们增加元素的时候，会使用 **add()** 函数，它会将元素放到末尾。具体实现：
```
/**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return true (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

这里我们可以看到ArrayList最核心的问题 **自动扩容机制** 的实现方法 **ensureCapacityInternal()**。让我们依次来看一下它的具体实现。
```
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 扩展为原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果扩为1.5倍还不满足需求，直接扩为需求值
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

当增加数据的时候，如果ArrayList的大小已经不满足需求时，那么就将数组变为原长度的1.5倍，之后的操作就是把老的数组拷到新的数组里面。例如，默认的数组大小是10，也就是说当我们add10个元素之后，再进行一次add时，就会发生自动扩容，数组长度由10变为了15。



# get()函数

接下来get()函数就比较简单了，先做index检查，然后执行访问操作。
```
/**
 * Returns the element at the specified position in this list.
 *
 * @param  index index of the element to return
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```



# set()函数

set()函数与get()函数差不多，先做index检查，然后执行赋值操作。
```
/**
 * Replaces the element at the specified position in this list with
 * the specified element.
 *
 * @param index index of the element to replace
 * @param element element to be stored at the specified position
 * @return the element previously at the specified position
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```



# remove()函数

接下来看看 **remove()** 函数的运用！ 

```
/**
 * Removes the element at the specified position in this list.
 * Shifts any subsequent elements to the left (subtracts one from their
 * indices).
 *
 * @param index the index of the element to be removed
 * @return the element that was removed from the list
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 把指定元素后面位置的所有元素，利用System.arraycopy方法整体向前移动一个位置
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    // 最后一个位置的元素指定为null，这样让gc可以去回收它
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

上面关键操作就是删除指定元素后，后续所有元素向前移动的实现了！



# contains()函数

**contains()** 函数就不用多说了，就简单地循环判断一下存不存在某个元素，代码如下。

```
/**
 * Returns true if this list contains the specified element.
 * More formally, returns true if and only if this list contains
 * at least one element e such that (o==null;o.equals(e)).
 *
 * @param o element whose presence in this list is to be tested
 * @return true if this list contains the specified element
 */
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}

public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```



# 新发现的疑惑

在看源码的过程中，发现elementData数组是用transient修饰的！
```
transient Object[] elementData; 
```

这是为什么？后来发现ArrayList实现了Serializable接口，代码如下。
```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

这是不是和序列化有关呢？没错！

这意味着ArrayList是可以被序列化的，用transient修饰elementData意味着我不希望elementData数组被序列化。这是为什么？因为序列化ArrayList的时候，ArrayList里面的elementData未必是满的，比方说elementData有10的大小，但是我只用了其中的3个，那么是否有必要序列化整个elementData呢？显然没有这个必要，因此ArrayList中重写了writeObject方法：

```
/**
 * Save the state of the ArrayList instance to a stream (that is, serialize it).
 *
 * @serialData The length of the array backing the ArrayList
 *             instance is emitted (int), followed by all of its elements
 *             (each an Object) in the proper order.
 */
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

每次序列化的时候调用这个方法，先调用defaultWriteObject()方法序列化ArrayList中的非transient元素，elementData不去序列化它，然后遍历elementData，只序列化那些有的元素，这样的好处是：
- 加快了序列化的速度
- 减小了序列化之后的文件大小

不得不承认JDK的源码是经过千锤百炼的，同时也提醒我们，在以后开发过程中，如果遇到类似的情况，不失为一种借鉴的方法！



# 小结

其实Java中的 ArrayList 实现就是 数据结构- **数组** 的实现，核心功能是可以进行 **自动扩容**，我们可以从中了解数组实现的核心思想！

关于JDK中数组的实现ArrayList的源码就分析就到这里了！




------
