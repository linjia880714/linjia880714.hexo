---
title: 面试-vivo
date: 2011-05-13 0:0:0
tags:
---

简单的自我介绍

### Java虚拟机

- JVM垃圾回收的算法？
标记整理

如何判断一个对象要被垃圾回收？GC Roots是怎么判断的?

垃圾回收器有哪些？有啥区别？

垃圾回收器CMS的原理？

常用垃圾收集方法？

常用的垃圾收集器？

CMS的缺点？

G1的特点

说下happen before的原理

虚拟机的内存模型是啥？

- 虚拟机堆栈的主要参数有哪些？
-Xms and -Xmx 指定JVM的初始和最大堆内存大小，两值可以设置相同，以避免每次垃圾回收完成后JVM重新分配内存。
-XX:PermSize and -XX:MaxPermSize:设置永久代大小的初始值和最大值（默认：最小值为物理内存的1/64，最大值为物理内存的1/16，永久代在堆内存中是一块独立的区域，这里设置的永久代大小并不会被包括在使用参数-XX:MaxHeapSize 设置的堆内存大小中）
-Xmn：设置年轻代大小。整个堆大小=年轻代大小 + 年老代大小 + 持久代大小。所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8
-Xss：设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。
-XX:PretenureSizeThreshold ：令大于这个设置值的对象直接在老年代分配。这样做的目的是避免在Eden区及两个Survivor区之间发生大量的内存复制
-XX:OnOutOfMemoryError：当内存溢发生时，我们甚至可以可以执行一些指令，比如发个E-mail通知管理员或者执行一些清理工作（$ java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/heapdump.hprof -XX:OnOutOfMemoryError ="sh ~/cleanup.sh" MyApp）
-XX:+HeapDumpOnOutOfMemoryError and -XX:HeapDumpPath：让JVM在发生内存溢出时自动的生成堆内存快照（堆内存快照文件有可能很庞大，推荐将堆内存快照生成路径指定到一个拥有足够磁盘空间的地方。）

### 中间件

常用消息中间件的区别？

kafka的实现原理？

KAFKA是否可以作为订单等特别重要业务场景的中间件？

DUBBO的原理、集群方式、负载集群方式、服务发现方式？


### spring
Spring使用到的设计模式？

Spring注解的实现原理？


### Java基础
说下Synchronized和ReentrantLock的区别和实现原理？

说下AQS的实现原理？如何解决并发获取锁的问题？

- 线程池的种类、实现原理、主要参数、队列的类型？
1. fixedThreadPool(10), n~n，`LinkBlockingQueue`
2. cachedThreadPool， 0~Integer.MAX, `SynchronousQueue`
3. singleThreadExecutor, 1 , `LinkBlockingQueue`
4. ScheduledThreadPool, n~Integer.MAX, `DelayedWorkQueue`

使用什么连接池？连接池的实现原理？

哪些情况会出现OutMemory

说下变量的原子性、可见性？

- volatile变量的作用
数据可见性
指令重排

### Linux

- 如何在LINUX中查询虚拟机的堆栈信息、垃圾回收情况？
jstack主要用来查看Java进程内的线程堆栈信息，常用于追踪多线程任务调度过程、对象lock（或者死锁）、并发同步阻塞、IO线程执行状态等；比如排查某个线程为何wait（假死，阻塞等）。

- 如何使用kill命令干掉进程时，可以让程序先执行完。
1   SIGHUP  启动被终止的程序，可让该进程重新读取自己的配置文件，类似重新启动。
2   SIGINT  相当于用键盘输入 [ctrl]-c 来中断一个程序的进行。
9   SIGKILL 代表强制中断一个程序的进行，如果该程序进行到一半，那么尚未完成的部分可能会有“半产品”产生，类似 vim会有 .filename.swp 保留下来。
15  SIGTERM 以正常的方式来终止该程序。由于是正常的终止，所以后续的动作会将他完成。不过，如果该程序已经发生问题，就是无法使用正常的方法终止时，输入这个 signal 也是没有用的。
19  SIGSTOP 相当于用键盘输入 [ctrl]-z 来暂停一个程序的进行。
```bash
// = kill pid
kill -15 pid
```

### 其他

如果接口的并发量比较大，导致性能跟不上，如何预热接口。

有没阅读常用框架的源码？