## 1. HashMap什么情况下发生死链 

JDK1.8之前，并发resize()的时候可能会发生死链。

[参考](https://coolshell.cn/articles/9606.html)

## 2. HashMap

**HashMap的特性？**

HashMap 存储键值对，实现快速存取数据；允许null键\值；非同步；不保证有序

**HashMap的原理，内部数据结构？**

基于hashing的原理，jdk1.8之前底层使用哈希表（数组+链表）实现。jdk1.8之后，当链表长度大于8时，将链表转换成红黑树。

HashMap中的最重要两个方法put、get

put方法的原理：传入k，v，通过取hashcode，高位参与运算、取模运算三步，获得bucket位置，进行存储，如果没有碰撞直接放bucket里，如果碰撞，以链表形式存在buctets后，如果节点已经存在就替换oldvalue。当hashmap的bucket占用情况超过capacity*load_factor则，通过resize方法扩容为2倍。

get方法的原理：传入k，通过取hashcode，高位参与运算、取模运算三步，获得bucket位置，并进一步调用equals方法确定键值对。

**当两个对象的hashcode相同会发生什么？**

若两个对象的hashcode相同，则他们的bucket位置相同，会发生碰撞。

**有哪些hash的实现方式？**

- 直接定址法：取k的某个线性函数值为散列地址。
- 数字分析法：提取关键字k中取值比较均匀的数字作为哈希地址。例如，生日使用月份和日期构成散列地址。
- 除留余数法：用关键字k除以某个不大于哈希表长度m的数p，将所得余数作为哈希表地址。
- 分段叠加法：按照哈希表地址位数将关键字分成位数相等的几部分，其中最后一部分可以比较短。然后将这几部分相加，舍弃最高进位后的结果就是该关键字的哈希地址。
- 平方取中法：如果关键字各个部分分布都不均匀的话，可以先求出它的平方值，然后按照需求取中间的几位作为哈希地址。
- 伪随机数法：采用一个伪随机数当作哈希函数。

**有哪些hash冲突解决办法？**

- 开放地址法：一旦发生了冲突，就去寻找下一个空的散列地址，只要散列表足够大，空的散列地址总能找到，并将记录存入。
- 链地址法：将哈希表的每个单元作为链表的头结点，所有哈希地址为i的元素构成一个链表。即发生冲突时就把该关键字链在以该单元为头结点的链表的尾部。
- 再哈希法：当哈希地址发生冲突用其他的函数计算另一个哈希函数地址，直到冲突不再产生为止。
- 建立公共溢出表：将哈希表分为基本表和溢出表两部分，发生冲突的元素都放入溢出表中。

**为什么String、Integer适合作为键？**

- 已重写了equals()和hashCode()。（让不相等的对象返回不同的hashcode值）
- 具有不可变性，可防止键值的改变，线程安全 

**HashMap与HashTable区别？**

1. 线程安全性。HashTable中几乎所有函数是同步的，所以它是线程安全的。HashMap是线程不安全的。
2. NULL值。HashMap的key、value都可为null，Hashtable都不可为null
3. 容器的初始值和扩容方式。HashMap默认16，扩容 （原始容量*2）,；HashTable默认11，扩容 （原始容量 * 2+1）
4. HashMap继承于AbstractMap，HashTable继承于Dictionary
5. hash算法。HashMap自定义的哈希算法；HashTable直接采用key的hashCode()；
6. 速度。单线程环境下HashMap更快

**让HashMap同步**

Collections.synchronizeMap()



## 3. 多线程顺序执行 

（1） 有A、B、C、D四个线程，A线程输出A, B线程输出B, C线程输出C，D线程输出D，要求, 同时启动四个线程, 按顺序输出ABCD 

答：通过Thread.join()方法，B、C、D分别持有A、B、C的引用，并且在输出前调用持有线程的join方法，等待线程执行完毕，再输出。

```java
package thread;

public class TestThread1 {

    public static void main(String[] args) {

        // 线程A
        final Thread a = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("A");
            }
        });

        // 线程B
        final Thread b = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    // 执行b线程之前，加入a线程,让a线程执行
                    a.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("B");
            }
        });

        // 线程C
        final Thread c = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    // 执行c线程之前，加入b线程,让b线程执行
                    b.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("C");
            }
        });

        // 线程D
        Thread d = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    // 执行d线程之前，加入c线程,让c线程执行
                    c.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("D");
            }
        });

        // 启动四个线程
        a.start();
        b.start();
        c.start();
        d.start();

    }

}
```

（2） 有A,B,C三个线程, A线程输出A, B线程输出B, C线程输出C。要求, 同时启动三个线程, 按顺序输出ABC, 循环10次 

使用ReentrantLock控制并发,并使用一个state整数来判断应哪个线程执行。 

```java
public class Test {
    private static ReentrantLock lock = new ReentrantLock();
    private static int state = 0;
    
    static class A extend Thread() {
        @Override
        public void run() {
            lock.lock();
            for (int i = 0; i < 10;) {
                if (state%3 == 0) {
                    System.out.println("A");
                }
                i++;
                state++;
            }
            lock.unlock();    
        }        
    }
    
    static class B extend Thread() {
        @Override
        public void run() {
            lock.lock();
            for (int i = 0; i < 10;) {
                if (state%3 == 1) {
                    System.out.println("B");
                }
                i++;
                state++;
            }
            lock.unlock();    
        }        
    }    
    
    static class C extend Thread() {
        @Override
        public void run() {
            lock.lock();
            for (int i = 0; i < 10;) {
                if (state%3 == 2) {
                    System.out.println("C");
                }
                i++;
                state++;
            }
            lock.unlock();    
        }        
    }
    
    public static void main(String[] args) {
        new A().start();
        new B().start();
        new C().start();
    }
}

```



## 4. hashCode()与equals()

equals()方法用来判断两个对象是否相等，Object默认比较对象地址

hashCode()方法用来获取哈希码，Object默认根据对象地址转换成一个整数

**为什么重写equals()一定要重写hashCode()方法**

这是因为HashMap等哈希表是由hashcode()定位要存放的位置，而equals()判断是否相等，这意味着逻辑上equals()相等，认为这两个对象相等则应该放在同一个桶中，所以hashcode()也需要相等。 