---
title: ArrayList
date: 2018-07-23 13:02:22
tags:
---

<!-- toc -->

## 环境
JDK: 1.8

## 分析

### add
```java
private void grow(int minCapacity) {

    int oldCapacity = elementData.length;
    // 数组大小增长逻辑
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 把原有数值的数据拷贝到新数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### remove
```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 把删除的index后面的数据往前移动
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 最后一个数据设置为空
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

### foreach时不能修改数据
只要Arraylist有任何的修改操作，modCount都会+1

当调用next会见检查modCount是否已经变化，就会抛出`ConcurrentModificationException`

```java
public E next() {
    checkForComodification();
    int i = cursor;
    if (i >= size)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    cursor = i + 1;
    return (E) elementData[lastRet = i];
}

final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

## 总结

ArrayList的结构还是相当简单的，可以稍微注意下下面两点
1. 如果预知List很大，不妨使用`public ArrayList(int initialCapacity)`刚开始就指定数组的大小，避免多次grow

2. Arraylist不是线程安全的