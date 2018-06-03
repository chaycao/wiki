# BlockingQueue

BlockingQueue 继承 Queue 接口。

提供三个添加元素方法：

| 方法    | 效果                                       |
| ----- | ---------------------------------------- |
| add   | 添加成功返回True，添加失败返回抛出 IllegalStateException 异常 |
| offer | 添加成功返回True，添加失败返回 False                  |
| put   | 若容量满了会阻塞直到                               |

提供三个删除元素方法：

| 方法     | 效果                              |
| ------ | ------------------------------- |
| poll   | 删除队列头部元素，并返回元素，若队列为空，返回null     |
| remove | 基于对象找到元素，并删除。成功返回True，失败返回False |
| take   | 删除队列头部元素，若队列为空，则阻塞直到队列有元素       |

常用阻塞队列：

- ArrayBlockingQueue：基于循环数组，规定大小，FIFO顺序

- LinkedBlockingQueue：基于单向链表，大小不定，FIFO顺序

- PriorityBlockingQueue：基于单向链表，大小不定，按对象的自然排序或其构造函数的Comparator决定的顺序

- LinkedBlockDeque：基于双向链表的双向队列，大小不定


## ArrayBlockingQueue的原理

使用一个可重入锁和该锁生成的两个条件对象进行并发控制（classic two-condition algorithm）

```java
final ReentrantLock lock;
private final Condition notEmpty;
private final Condition notFull;
```

## LinkedBlockingQueue的原理

使用放锁、拿锁，两个锁实现阻塞（"two lock queue" algorithm）

由于有两个锁，所以添加数据和删除数据可并行进行。

不仅在消费数据的时候进行唤醒插入阻塞的线程，同时在插入的同时如果容量还没满，也会唤醒阻塞的线程。

```java
private final ReentrantLock takeLock = new ReentrantLock();
private final Condition notEmpty = takeLock.newCondition();
private final ReentrantLock putLock = new ReentrantLock();
private final Condition notFull = putLock.newCondition();
```