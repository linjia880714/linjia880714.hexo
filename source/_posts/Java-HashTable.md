---
title: HashTable
date: 2018-07-28 00:52:42
tags:
---


## 环境
JDK: 1.8

## 分析
### 1.底层数据结构

可以看出HashTable的底层结构是数组加链表的结构

```java
private transient Entry<?,?>[] table;

private static class Entry<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Entry<K,V> next;
}
```

### 2.默认值
size=11的数组， 数组元素大于 (size * 0.75) 扩容
```java
public Hashtable() {
    this(11, 0.75f);
}

public Hashtable(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal Load: "+loadFactor);

    if (initialCapacity==0)
        initialCapacity = 1;
    this.loadFactor = loadFactor;
    table = new Entry<?,?>[initialCapacity];
    threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
}

```

### 3.put操作
```java
public synchronized V put(K key, V value) {
    
    // 省略...

    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    // hash & 0x7FFFFFFF 作用是把hash的值都转成正数
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    // 如果key存在更新操作
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    addEntry(hash, key, value, index);
    return null;
}


private void addEntry(int hash, K key, V value, int index) {
    modCount++;

    Entry<?,?> tab[] = table;
    if (count >= threshold) {
        // 扩容
        rehash();

        tab = table;
        hash = key.hashCode();
        index = (hash & 0x7FFFFFFF) % tab.length;
    }

    // 获取旧的节点
    Entry<K,V> e = (Entry<K,V>) tab[index];
    // 新的节点放在头部
    tab[index] = new Entry<>(hash, key, value, e);
    count++;
}
```

### 4.rehash 扩容
