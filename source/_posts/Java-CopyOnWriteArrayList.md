---
title: CopyOnWriteArrayList
date: 2018-07-20 15:03:45
tags:
---

## 环境
JDK: 1.8

## 分析
场景：读操作远大于写

特点：读取不用加锁，写入也不会阻塞读操作，写入-写入需要同步等待


```java
    /** 保护写操作的线程安全 */
    final transient ReentrantLock lock = new ReentrantLock();

    /** 存放数据的数组 */
    private transient volatile Object[] array;
```

get操作，很简单，没有锁
```java
    private E get(Object[] a, int index) {
        return (E) a[index];
    }

    public E get(int index) {
        return get(getArray(), index);
    }
```

add操作，加锁
```java
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            //  重要：把原有的数据copy一份，改完再赋值给array = 写操作不会影响原有数据
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```