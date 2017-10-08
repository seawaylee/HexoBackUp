---
title: Java Collections Framework - ArrayList
date: 2017-03-22 10:34:41
tags: [Java基础]
---

## 定义

>ArrayList底层以数组实现，允许重复，默认第一次插入元素时创建数组的大小为10，超出限制时会增加50%的容量，
每次扩容都底层采用System.arrayCopy()复制到新的数组，初始化时最好能给出数组大小的预估值。

```java
package java.util;
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable { 
    private static final int DEFAULT_CAPACITY = 10; 
    private static final Object[] EMPTY_ELEMENTDATA = {}; 
    private transient Object[] elementData; 
    private int size;
    //其余省略
}


```
<!--more-->

## 概述

按数组下标访问元素—get(i)/set(i,e) 的性能很高，这是数组的基本优势。

```java
public E get(int index) {    
    rangeCheck(index);    
    return elementData(index);
}

public E set(int index, E element) {    
    rangeCheck(index);    
    E oldValue = elementData(index);    
    elementData[index] = element;    
    return oldValue;
}
```
直接在数组末尾加入元素—add(e)的性能也高，但如果按下标插入、删除元素—add(i,e), remove(i), remove(e)，则要用System.arraycopy()来移动部分受影响的元素，性能就变差了，这是劣势。

ArrayList中有一个方法trimToSize()用来缩小elementData数组的大小，这样可以节约内存：

```java
public void trimToSize() { 
    modCount++; 
    if (size < elementData.length) { 
        elementData = Arrays.copyOf(elementData, size); 
    } 
}
```
考虑这样一种情形，当某个应用需要，一个ArrayList扩容到比如size=10000，之后经过一系列remove操作size=15，在后面的很长一段时间内这个ArrayList的size一直保持在<100以内，那么就造成了很大的空间浪费，这时候建议显式调用一下trimToSize()这个方法，以优化一下内存空间。   
或者在一个ArrayList中的容量已经固定，但是由于之前每次扩容都扩充50%，所以有一定的空间浪费，可以调用trimToSize()消除这些空间上的浪费。

## RandomAccess

这个接口有什么用？
实现RandomAccess接口的集合有：ArrayList, AttributeList, CopyOnWriteArrayList, RoleList, RoleUnresolvedList, Stack, Vector等。
在RandomAccess接口的注释中有这么一段话：

```java
for (int i=0, n=list.size(); i < n; i++) {     
    list.get(i);
}
runs faster than this loop:
for (Iterator i=list.iterator(); i.hasNext(); ) { 
   i.next();
}
```

说明实现了RandomAccess接口的集合，在数据量很大的情况下，采用迭代器遍历比较慢。

## 和LinkedList的区别

1、ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
2、对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
3、对于新增和删除操作add和remove（不是在尾部添加删除），LinkedList比较占优势，因为ArrayList要移动数据。

## 和Vector的区别

1、Vector和ArrayList几乎是完全相同的,唯一的区别在于Vector是同步类(synchronized)，属于强同步类。因此开销就比ArrayList要大，访问要慢。正常情况下,大多数的Java程序员使用ArrayList而不是Vector,因为同步完全可以由程序员自己来控制。
2、Vector每次扩容请求其大小的2倍空间，而ArrayList是1.5倍。
3、Vector还有一个子类Stack.

