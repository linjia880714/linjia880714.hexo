---
title: Vector
date: 2018-07-23 14:21:10
tags:
---
<!-- toc -->

## 环境
JDK: 1.8

## 分析

Vector数据结构逻辑和Arraylist差不多，Vector是线程安全的

区别
1. 新建对象Vector会默认生成长度为10的数组，Arraylist是长度为0的数组（第一次add扩容）

2. grow方法有区别
```java
private void grow(int minCapacity) {
   
    int oldCapacity = elementData.length;

    // 1. 有指定capacityIncrement变量，新的长度 = 旧长度 + capacityIncrement
    // 2. 没有指定capacityIncrement变量，新的长度 = 旧长度 * 2
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```