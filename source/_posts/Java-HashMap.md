---
title: HashMap
date: 2018-07-23 22:48:15
tags:
---

<!-- toc -->

## 环境
JDK: 1.8

## 分析
### 1.底层数据结构

可以看出HashMap的底层结构是数组加链表的结构
（一定条件下会转成数组加树，稍后讲）

```java

transient Node<K,V>[] table

static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    // 忽略
}
```

### 2.table初始化和扩容
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

### 3.扩容之后的数据整理（链表）
需要看懂下面的逻辑代码才好看些，可能有点难以理解，这也是HashMap的优化

{% asset_img 01.png %}

```java
final Node<K,V>[] resize() {
    
    // 省略 。。。

    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 如果没有next，重新获取index
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 如果是TreeNode的结构（先不看）
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 如果有next
                else { 
                    // 不需要改变index的node
                    Node<K,V> loHead = null, loTail = null;
                    // 需要改变index（j+oldCap）的node
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // “不需要改变index”的判断
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                         // 其他就是“需要改变index”
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    // 循环结构，构成链表
                    } while ((e = next) != null);

                    if (loTail != null) {
                        loTail.next = null;
                        // 不需要改变index的node还是放在原来的位置
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        // 需要改变index的node放在 j + oldCap 的位置
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```