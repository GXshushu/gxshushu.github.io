---
title: 【Java】BlockingQueue 阻塞队列
date: 2024-12-25 21:57:34 +0800
categories: [Java]
tags: [Java 学习]
pin: false
author: C9xx

toc: true
comments: true
typora-root-url: ../../gxshushu.github.io
math: false
mermaid: true
---

BlockingQueue是JUC包下的一个阻塞队列接口。

继承自Queue父类，BlockingQueue有两个实现类ArrayBlockingQueue和ListBlockingQueue，分别是数组的队列实现和链表的队列实现。

## ArrayBlockingQueue
ArrayBlockingQueue实现BlockingQueue接口并AbstractQueue。

AbstractQueue提供了一系列队列操作。

分别基于offer、poll、peek来实现add、remove、element方法。

```java
    public boolean add(E e) {
        if (offer(e))
            return true;
        else
            throw new IllegalStateException("Queue full");
    }
    public E remove() {
        E x = poll();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }    
    public E element() {
        E x = peek();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }
    public void clear() {
        while (poll() != null)
            ;
    }
    public boolean addAll(Collection<? extends E> c) {
        if (c == null)
            throw new NullPointerException();
        if (c == this)
            throw new IllegalArgumentException();
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
```

### 成员变量
在ArrayBlockingQueue中比较重要的字段

+ final Object[] items 是一个存储队列元素的数组，是真正存储数据的结构；
+ int takeIndex 是一个读取位置的索引（指针）,指向对头；
+ int putIndex 是一个存入位置的索引，指向队尾的后一位；
+ int count 是队内元素的计数。

除此之外，该类还有用于并发控制的字段

+ final ReentrantLock lock 阻塞队列的主锁，操作该队列时会竞争这把锁；
+ private final Condition notEmpty 队列是否为空的条件变量；
+ private final Condition notFull 队列是否为满的条件变量；
+ transient Itrs itrs 这是一个迭代器可以遍历队列。

### 构造器
ArrayBlockedQueue有三个构造器都是public的。

```java
public ArrayBlockingQueue(int capacity) {
    this(capacity, false);
}
```

可以只传一个队列容量来创建ArrayBlockedQueue，他会嵌套调用两个参数的构造器。

```java
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}
```

在这个构造器中初始化了items、lock以及两个条件变量。items初始化为一个长度为capacity的对象数组，而根据传入的fair去指定lock是否为公平锁。

```java
public ArrayBlockingQueue(int capacity, boolean fair,
                              Collection<? extends E> c) {
        this(capacity, fair);

        final ReentrantLock lock = this.lock;
        lock.lock(); // Lock only for visibility, not mutual exclusion
        try {
            int i = 0;
            try {
                for (E e : c) {
                    checkNotNull(e);
                    items[i++] = e;
                }
            } catch (ArrayIndexOutOfBoundsException ex) {
                throw new IllegalArgumentException();
            }
            count = i;
            putIndex = (i == capacity) ? 0 : i;
        } finally {
            lock.unlock();
        }
    }
```

而三个参数的构造器除了调用两个参数的构造器之外，还传入了另一个容器对象c，并将c中的元素放进items里面，也就是用一个已经存在的容器的元素来初始化这个队列。可以看到在把c中元素全部放进items数组后，将count改为i，并将存入索引改为i（如果存进去的元素刚好放满队列则让索引为0，这是使用数组实现循环队列的策略）。

### 入队
可以使用add、offer、put来将元素放进队列尾部。

```java
// ArrayBlockingQueue.add
public boolean add(E e) {
    return super.add(e);
}

// AbstractQueue
public boolean add(E e) {
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}
```

其实add方法本质上还是调用了offer方法。add方法调用了父类的add方法，而在父类中调用了ArrayBlockingQueue的offer方法，如果offer返回true则返回True，否则抛出队列满的异常。

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

private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
    notEmpty.signal();
}
private static void checkNotNull(Object v) {
    if (v == null)
        throw new NullPointerException();
}
```

主要看offer和put方法，offer方法检查e是否为Null，然后上锁判断队列是否满，满了返回false，否则调用enqueue私有方法将元素e入队。

enqueue是基础入队操作，将元素放在items数组的写入索引的位置，并且自增写入索引（循环自增）和元素计数，并且唤醒等待notEmpty条件变量的线程（因为队列为空而进入WAITING状态的线程，例如取队头元素）。

```java
public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {

        checkNotNull(e);
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length) {
                if (nanos <= 0)
                    return false;
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(e);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

还有一个可定义超时的offer方法，使用了可打断锁，在定义的时间内等待notFull条件变量，等待不到同样是返回false。

```java
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            notFull.await();
        enqueue(e);
    } finally {
        lock.unlock();
    }
}
```

put方法同样先检查e是否为空，但上的是可打断的锁，然后判断队列是否已满，如果队列已满则等待notFull条件变量，同时在外层使用循环判断队满避免虚假唤醒，由于是可打断锁，所以不用担心线程无限等待。当队列不满则调用enqueue方法将e入队。

### 出队
出队基础操作使用私有方法dequeue，外界通过poll和take方法间接调用dequeue方法进行元素出队。

```java
private E dequeue() {
    // assert lock.getHoldCount() == 1;
    // assert items[takeIndex] != null;
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length)
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
    notFull.signal();
    return x;
}
```

dequeue方法将items数组的读取索引指向的元素读取出来，并将items该位置置为空，并自增读取索引、自减元素计数，然后唤醒等待notFull条件变量的进程（比如上述的put方法以及带超时参数的offer方法），并返回出队的元素。

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

public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0) {
            if (nanos <= 0)
                return null;
            nanos = notEmpty.awaitNanos(nanos);
        }
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

与offer相对，poll方法同样有两个版本，瞬时完成的版本和带超时的版本。无参版本上锁后判断队列是否为空为空则返回空指针，不为空则返回dequeue的结果。超时版本的返回值一样，只是会在有限时间内等待notEmpty变量，有一定的容忍度。

```java
public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
```

take方法上一个可打断锁然后无限等待notEmpty变量，然后执行dequeue方法并返回。



### 只取队头不出队
```java
public E peek() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return itemAt(takeIndex); // null when queue is empty
    } finally {
        lock.unlock();
    }
}

@SuppressWarnings("unchecked")
final E itemAt(int i) {
    return (E) items[i];
}
```

直接返回items[takeIndex]。

### 清空
```java
public void clear() {
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        int k = count;
        if (k > 0) {
            final int putIndex = this.putIndex;
            int i = takeIndex;
            do {
                items[i] = null;
                if (++i == items.length)
                    i = 0;
            } while (i != putIndex);
            takeIndex = putIndex;
            count = 0;
            if (itrs != null)
                itrs.queueIsEmpty();
            for (; k > 0 && lock.hasWaiters(notFull); k--)
            notFull.signal();
        }
    } finally {
        lock.unlock();
    }
}
```

将takeIndex和putIndex之间的所有元素都置为空，然后唤醒不多于原来count计数的notFull等待线程。

### 容器状态判断方法
```java
public int size() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return count;
    } finally {
        lock.unlock();
    }
}
public int remainingCapacity() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return items.length - count;
    } finally {
        lock.unlock();
    }
}
```

size方法返回元素计数count，remainingCapacity方法返回数组长度与元素计数的差值（剩余空位）。

```java
public boolean contains(Object o) {
    if (o == null) return false;
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count > 0) {
            final int putIndex = this.putIndex;
            int i = takeIndex;
            do {
                if (o.equals(items[i]))
                    return true;
                if (++i == items.length)
                    i = 0;
            } while (i != putIndex);
        }
        return false;
    } finally {
        lock.unlock();
    }
}
```

上锁之后判断队列不为空就循环从takeIndex查找到putIndex逐个判断是否与参数o相等，有则返回true，否则返回false。

### 内部迭代器类Itr
等写



### 小结
ArrayBlockingQueue使用数组来实现了阻塞队列接口，创建对象的时候主要参数是队列容量和锁公平与否。入队出队如果需要使用出入队操作必须执行，并且可以接受一直等待则可以直接使用put和take，如果想要队满或空执行失败立刻返回false，则使用无超时的offer和poll，允许等待一定时间来保证一定的成功率则使用带超时的offer和poll。并且ArrayBlockingQueue使用了ReentrantLock来保证它的线程安全性。



## LinkedBlockingQueue
等写

