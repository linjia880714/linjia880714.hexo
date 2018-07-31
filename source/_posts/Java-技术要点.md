---
title: Java技术要点
date: 2011-5-13 00:00:00
tags:
---

<!-- toc -->

## 引用
- Strong Reference: Object obj = new Object();
- Soft Reference: 内存溢出异常前，软引用进行第二次回收
- Weak Reference: 只能生存到下次gc
- Phantom Reference:

----

## 对象是否该死
经历两次标记
1. 没有GC Root相连接的引用
2. 对象是否有必要执行finalize(), 同一个对象的finalize()只会被调用一次

----

## 垃圾收集算法

----

### 标记清除算法(Mark-Sweep)
缺点
1. 效率不高，标记和清除的过程效率都不高`why?`
2. 产生大量的不连续的内存碎片


----

### 复制算法(Copying)
过程
1. 内存分成两块A和B
2. A内存用完，把活的对象复制到B，只要移动到堆顶指针，按循序分配内存
3. 清理A的内存

缺点
1. 内存代价过高

现实应用
1. 98%的对象朝闻夕死，没有必要1：1去分配空间
2. Eden：Survivor1：Survivor2 8：1：1
3. 回收是Eden和Survivor1存活对象复制到Surivoro2
4. Eden和Survivor1一次清空
5. Survivor2不一定有足够的空间放下存活的对象，需要老年代分配担保(Handle Promotion)

----

### 标记整理算法(Mark-Compact)
如果对象存活率较高，需要大量的复制，而且还需要额外的分配担保，所以老年代不适用这种算法

过程
1. 标记
2. 存活对象向一段移动
3. 清理边界以外的内存

----

## 收集器

### Serial['sɪriəl]
过程
1. 单线程
2. 必须暂停所有的工作线程

算法
1. 新生代 - 复制算法
2. 老年代 - 标记整理算法

### ParNew
Serial的多线程版本，其他没太多区别

### Parallel Scavenge Parallel ['pærəlel] ['skævɪndʒə(r)] 
