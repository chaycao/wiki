# ThreadLocal

## 作用

是线程内的局部变量，只能被当前线程访问，其他线程无法访问和修改。可减少同一个线程内多个函数或组件之间一些公共变量的传递的复杂度。

## 实现原理

每个 Thread 维护一个 ThreadLocalMap 映射表，该表的 key 为 ThreadLocal 实例，value 为需要存储的 Object。

即，ThreadLocal本身不存储值，只作为 key 让线程从 ThreadLocalMap 获取value。

![](images/threadlocal.jpg)


图中的虚线表示 ThreadLocalMap 使用 ThreadLocal 的弱引用作为 key（ThreadLocal 若无强引用，会在GC时被回收）。

为什么不使用强引用

当 ThreadLocal Ref 的对象被回收时，ThreadLocalMap 仍持有 ThreadLocal 强引用，若没有手动删除，会导致 Entry 内存泄漏。

## 隐患——内存泄漏

当 ThreadLocal 无强引用时，会在GC时被回收。则 ThreadLocalMap 中会出现 key 为 null 的 Entry，则无法访问该 Entry 的 value。若线程不结束，则 value 存在一条强引用链：Thread Ref -> Thread -> ThreadLocalMap -> Entry -> value，将无法回收，造成内存泄露。

ThreadLocal 针对上面情况，已有防护措施：在 ThreadLocal 的 get()、set()、remove()的时候会清除线程 ThreadLocalMap 里所有 key 为 null 的 value。

但是！仍然可能内存泄漏：

使用 static 的 ThreadLocal，延长了 ThreadLocal 的生命周期。

分配了 ThreadLocal，但不调用get()、set()、remove。

内存泄漏的根本原因：ThreadLcoalMap 的生命周期与 Thread 一样长。

## 使用建议

每次使用完 ThreadLocal，调用remove()方法，清除数据。



## 参考

- [理解Java中的ThreadLocal](https://droidyue.com/blog/2016/03/13/learning-threadlocal-in-java/)
- [深入分析 ThreadLocal 内存泄漏问题](http://blog.xiaohansong.com/2016/08/06/ThreadLocal-memory-leak/)
