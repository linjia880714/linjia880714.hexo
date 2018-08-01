---
title: ArrayList
date: 2018-07-23 13:02:22
tags:
---

<!-- toc -->

## 环境
JDK: 1.7

## 分析

ArrayList底层数据结构就是一个数组，随着数据的添加不断扩容

### add 添加
add操作非常简单
```java
 public boolean add(E e) {
    // 扩容逻辑
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

### remove 删除
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

### grow 扩容
```java
private void grow(int minCapacity) {

    int oldCapacity = elementData.length;
    // 默认第一次初始化长度为 10
    // 数组大小增长逻辑 原本大小 + 原本大小/2 取整数
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 把原有数值的数据拷贝到新数组
    elementData = Arrays.copyOf(elementData, newCapacity);
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

### 循环遍历时安全删除数据
错误示范
```java
// 新建数据塞进10个元素
ArrayList list = new ArrayList();
for(int i=0 ; i< 10; i++)
{
    list.add(i);
}

// 
for(int i=0; i<10;i++)
{
    System.out.println("i="+i+"; res="+list.get(i));
    list.remove(i);
    System.out.println(list);
}

/*
从结果看，是跳着访问，因为执行remove操作，数据会往前一定，size--
i=0; res=0
[1, 2, 3, 4, 5, 6, 7, 8, 9]
i=1; res=2
[1, 3, 4, 5, 6, 7, 8, 9]
i=2; res=4
[1, 3, 5, 6, 7, 8, 9]
i=3; res=6
[1, 3, 5, 7, 8, 9]
i=4; res=8
[1, 3, 5, 7, 9]
java.lang.IndexOutOfBoundsException: Index: 5, Size: 5
    at java.util.ArrayList.rangeCheck(ArrayList.java:653)
    at java.util.ArrayList.get(ArrayList.java:429)
    at org.demo.javabase.base.HasStatic.main(HasStatic.java:27)
*/
```

正确示范
```java
Iterator<Integer> i = list.iterator();         
while (i.hasNext()) {
    int tmp = i.next();
    // 可以执行两次remove看会有什么结果
    i.remove();
    System.out.println(list);
}


// 在ArrayList.java里面
private class Itr implements Iterator<E> {
    // 下一个元素的index
    int cursor;   
    /* 
      1. 已经返回给调用者的index，调用next。 
      2. 执行remove时，remove的元素是elementData[lastRet]
         然后lastRet = -1. 如果没有调用next就继续调用remove，IllegalStateException
    */
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
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

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    // 省略...
}

```


## 总结

ArrayList的结构还是相当简单的，可以稍微注意下下面两点
1. 如果预知List很大，不妨使用`public ArrayList(int initialCapacity)`刚开始就指定数组的大小，避免多次grow

2. Arraylist不是线程安全的