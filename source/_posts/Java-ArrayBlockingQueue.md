---
title: ArrayBlockingQueue
date: 2018-07-20 15:54:29
tags:
---

<!-- toc -->

## 环境
JDK: 1.8

## 分析

```java

    /*
     * takeIndex和putIndex是items上面的索引，控制每次出队入队对应的inex
     * 
     * 具体使用可以查看dequeue()，enqueue()
     */
    final Object[] items;
    int takeIndex;
    int putIndex;
 
    final ReentrantLock lock;
    private final Condition notEmpty;
    private final Condition notFull;
```

### 入队
put(E e), 会阻塞线程
```java
	public void put(E e) throws InterruptedException {
	    checkNotNull(e);
	    final ReentrantLock lock = this.lock;
	    lock.lockInterruptibly();
	    try {
	    	// 如果队列已满
	        while (count == items.length)
	        	// 线程等待
	        	/* 
	        		为什么会套在while循环
	        		1. 当前线程被唤醒后，继续检查队列是否已经满了，满了继续等待
	        	*/
	            notFull.await();
	        enqueue(e);
	    } finally {
	        lock.unlock();
	    }
	}

	private void enqueue(E x) {
		final Object[] items = this.items;
		items[putIndex] = x;
		if (++putIndex == items.length)
		    putIndex = 0;
		count++;
		// 唤醒获取操作时，队列为空而进入等待的线程
		notEmpty.signal();
	}
```


offer(E e)， 不会阻塞线程，队列满了就直接 return false
```java
    public boolean offer(E e) {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            if (count == items.length)
                return false;
            else {
                enqueue(e);
                return true;
            }
        } finally {
            lock.unlock();
        }
    }
```

offer(E e, long timeout, TimeUnit unit), 队列满时限时等待，超时 return false
```java
    public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {

        checkNotNull(e);
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length) {
            	// 如果队列满了且等待时间耗尽，直接return false
                if (nanos <= 0)
                    return false;
                // 线程限时等待
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(e);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

### 出队
take()，会阻塞线程
```java
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
        	// 队列空
            while (count == 0)
                // 线程等待
                /* 
                    为什么会套在while循环
                    1. 当前线程被唤醒后，继续检查队列是否还是空，空继续等待
                */         
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }

    private E dequeue() {
        final Object[] items = this.items;
        E x = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
        // 唤醒插入操作时，队列已满而进入等待的线程
        notFull.signal();
        return x;
    }
```

poll(), 不会阻塞线程，队列为空直接 return null
```java
    public E poll() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return (count == 0) ? null : dequeue();
        } finally {
            lock.unlock();
        }
    }
```

poll(long timeout, TimeUnit unit), 队列空时限时等待，超时 return null;
```java
    public E oll(long timeout, TimeUnit unit) pthrows InterruptedException {
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
        	// 队列空
            while (count == 0) {
            	// 如果队列空且等待时间耗尽，直接return null
                if (nanos <= 0)
                    return null;
                // 线程限时等待
                nanos = notEmpty.awaitNanos(nanos);
            }
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
```