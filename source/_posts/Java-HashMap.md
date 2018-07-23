---
title: Java-HashMap
date: 2018-07-23 22:48:15
tags:
---

## 环境
JDK: 1.8

## 分析
### 底层数据结构

可以看出HashMap的底层结构是数组加链表的结构
（一定条件下会装成数组加树，稍后讲）

```java

transient Node<K,V>[] table

static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```

### table初始化和扩容
```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 在HashMap.resize()
// 第一次初始化
newCap = DEFAULT_INITIAL_CAPACITY;  // 16，新数组长度
newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); // 16 * 0.75，门槛值

// 在HashMap.resize()
// 扩容时，size > threshold 触发扩容
newCap = oldCap << 1  // 翻倍
newThr = oldThr << 1  // 翻倍
```