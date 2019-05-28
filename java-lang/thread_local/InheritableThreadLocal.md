# InheritableThreadLocal

This class holds a thread-local value that is inherited by child threads<sub>【注：可以被child threads继承】</sub>. See `ThreadLocal` for a discussion of thread-local values.

Note that the **inheritance**<sub>【注：继承这个概念】</sub> referred to in the name of this class is not `superclass-to-subclass` inheritance<sub>【注：并不是父类和子类之间的继承】</sub>; instead, it is `parent-thread-to-child-thread`<sub>【注：而是父线程和子线程之间的继承】</sub> inheritance.

This class is best understood by example<sub>【注：通过举例说明】</sub>. Suppose that an application has defined an `InheritableThreadLocal` object and that a certain thread (the parent thread) has a thread-local value stored in that object<sub>【注：父线程中有一个对象】</sub>. Whenever that thread creates a new thread (a child thread), the `InheritableThreadLocal` object is automatically updated so that the new child thread has the same value associated with it as the parent thread<sub>【注：子线程会自动“继承”父线程中的值】</sub>. Note that the value associated with the child thread is independent from the value associated with the parent thread<sub>【注：之后，子线程的值是独立于父线程的】</sub>. If the child thread subsequently alters its value by calling the `set()` method of the `InheritableThreadLocal`, the value associated with the parent thread does not change<sub>【注：子线程的值发生改变，而父线程中的值并不会受到影响】</sub>.

By default, a child thread inherits a parent’s values unmodified<sub>【注：默认情况下】</sub>. By overriding the `childValue()` method<sub>【注：可以自定义】</sub>, however, you can create a subclass of `InheritableThreadLocal` in which the child thread inherits some arbitrary function of the parent thread’s value.

```java
public class InheritableThreadLocal extends ThreadLocal {
    // Public Constructors
    public InheritableThreadLocal();

    // Protected Instance Methods
    protected Object childValue(Object parentValue);
}
```
