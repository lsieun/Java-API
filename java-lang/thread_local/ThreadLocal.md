# ThreadLocal

- 第一，从“概念”角度来理解，是什么
- 第二，从“代码”角度来理解，有哪些方法
- 第三，从“使用”角度来理解，怎么用、怎么验证

<!-- TOC -->

- [1. Concept View](#1-concept-view)
- [2. Code View](#2-code-view)
- [3. Use View](#3-use-view)
- [4. More](#4-more)

<!-- /TOC -->

## 1. Concept View

This class provides a convenient way to create **thread-local variables**<sub>【注：为了理解这个概念，下面用三个点来说】</sub>.

- When you declare a `static` field in a class, there is only one value for that field, shared by all objects of the class.
- When you declare a `nonstatic` instance field in a class, every object of the class has its own separate copy of that variable.
- `ThreadLocal` provides an option between these two extremes. If you declare a `static` field to hold a `ThreadLocal` object, that `ThreadLocal` holds a different value for each thread.

理解三个概念：Field、Class Instance、Thread

- Field。面向对象的概念中，“封装”是将
- Class/Instance。比Field更大的一层概念是Class或Instance。
- Thread。比Class/Instance更大的概念是Thread。

在JVM运行的时候，code是在线程（Thread）中执行的。有一种说法，线程（Thread）是代码的执行路径。在线程（Thread）中，有许多的Class和Instance。JVM就像一个虚拟的世界，Thread就像一条条河流，而Class和Instance就像一条小船，而Field就像船上的人。

整理一下：

- static field。这个static field，被所有的instance共享；这个static field，被所有的Thread共享，这是一种“公产”，即“公共的财产”
- nonstatic field。这个nonstatic field，每一个instance有自己的一份；这个nonstatic field，在一个Tread中有多个。
- ThreadLocal Field。这正是介于两者之间的一种状态。

JVM是一个世界，堆内存就是大陆，static field就像一个King，他在所有Instance上和Thread上都是有效的，而ThreadLocal field就像一个Landlord，他是一个地方的领主，他只在自己的领地（Thread）上是有效的，而在别人的领地（Thread）上就没有权威了。而non-static field就是一个common people，一个普通的平民。

形象化理解：

- 中国周朝
  - 天子（static field），
  - 诸侯（ThreadLocal field）
  - 平民（nonstatic field）
- 西方王国
  - King
  - LandLord
  - HouseHold/common people

## 2. Code View

主要是三个方法

```java
public class ThreadLocal {
    // Public Constructors
    public ThreadLocal();

    // Public Instance Methods
    public Object get();
    public void set(Object value);

    // Protected Instance Methods
    protected Object initialValue();
}
```

- （1）`set()`方法。The `set()` method sets the value held by the `ThreadLocal` object for the currently running thread.
- （2）`get()`方法。`get()` returns the value held for the currently running thread.
- （3）`initialValue()`方法。If a thread calls `get()` for the first time without having first called `set()` to establish a thread-local value, `get()` calls the protected `initialValue()` method to obtain the initial value to return. The default implementation of `initialValue()` simply returns `null`, but subclasses can override this if they desire.

需要注意的事情

- Note that there is no way to obtain the value of the `ThreadLocal` object for any thread other than the one<sub>【注：只能获取自己线程中的数据】</sub> that calls `get()`.
- Objects running in the same thread<sub>【注：同一个线程】</sub> see the same value<sub>【注：相同的值】</sub> when they call the `get()` method of the `ThreadLocal` object.
- Objects running in different threads<sub>【注：不同的线程】</sub> obtain different values<sub>【注：不同的值】</sub> from `get()`, however.

从Map的角度来理解`ThreadLocal`

To understand the `ThreadLocal` class, you may find it helpful to think of a `ThreadLocal` object as a hashtable or `java.util.Map` that maps from Thread objects to arbitrary values. Calling `set()` creates an association between the current `Thread` (`Thread.currentThread()`) and the specified value. Calling `get()` first looks up the current thread, then uses the hashtable to look up the value associated with that current thread.



## 3. Use View

`ThreadLocal` instances are typically `private` `static` fields in classes that wish to associate state with a thread (e.g., a user ID or Transaction ID).

For example, the class below generates unique identifiers local to each thread. A thread's id is assigned the first time it invokes `ThreadId.get()` and remains unchanged on subsequent calls.

```java
import java.util.concurrent.atomic.AtomicInteger;

public class ThreadId {
    // Atomic integer containing the next thread ID to be assigned
    private static final AtomicInteger nextId = new AtomicInteger(0);

    // Thread local variable containing each thread's ID
    private static final ThreadLocal<Integer> threadId =
        new ThreadLocal<Integer>() {
            @Override protected Integer initialValue() {
                return nextId.getAndIncrement();
        }
    };

    // Returns the current thread's unique ID, assigning it if necessary
    public static int get() {
        return threadId.get();
    }
}
```

示例使用

```java
import java.io.Serializable;

public class HelloWorld implements Serializable, Cloneable {
    public static void main(String[] args) throws Exception {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + ":" + ThreadId.get());
            }
        });

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + ":" + ThreadId.get());
            }
        });

        t1.start();
        t2.start();

        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + ":" + ThreadId.get());
        }

        Thread.sleep(3000);
    }
}
```

## 4. More

See also `InheritableThreadLocal`, which allows thread-local values to be inherited from **parent threads** by **child threads**.
