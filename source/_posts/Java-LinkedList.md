---
title: LinkedList
date: 2018-07-23 17:49:42
tags:
---
<!-- toc -->

## 环境
JDK: 1.7

## 分析

### 底层数据结构
从代码上看可以看出是个双向链表
```java
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

### add
add的逻辑还是很简单的，比起ArrayList要扩容，拷贝数据要简单很多
1. last为空，first,list就指向第一个插入的数
2. last不为空，list.next指向新插入的元素

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

### get
双向链表的get需要从第一个元素/最后一个元素开始遍历。所以对于读操作很重的，LinkedList不是个好选择

这里有个小优化就是，index离first比较近从头部开始遍历，离last比较近从尾部开始遍历, 只有双向链表才可以

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}

Node<E> node(int index) {
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

### remove
remove的逻辑也很简单，就把元素的prev，next从新指向就好了
```java
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}

E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}    
```

### modCount
作用和{% post_link Java-ArrayList ArrayList %}一样

## 总结
LinkedList在插入删除有很大的优势，在读取就要遍历链表