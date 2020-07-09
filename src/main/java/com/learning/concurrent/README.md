### 解决共享资源竞争

#### synchronized:
- 在对象上调用其任意的synchronized方法的时候，此对象都会被加锁，此时该对象的其他synchronized方法只有等到前一个方法调用完毕，并释放了锁之后才能被调用。
- 如果一个方法在同一个对象上调用了第二个方法，后者又调用了同一对象上的另一个方法。JVM负责跟踪对象被加锁的次数，在任务第一次给对象加锁的时候，计数变为1，每当这个相同的任务在这个对象上获得锁时，技术都会增加，每当任务离开一个synchronized的方法，计数递减。当计数为0时，锁完全释放。（前提是首先获得了锁的任务才能允许继续获取多个锁）
- 针对每个类，也有一个锁（作为类的Class对象的一部分），所以synchronized static 方法可以在类的范围内防止对static数据的并发访问。
- 对比synchronized 同步方法和同步控制块的效率，同步控制块对象不加锁的时间更长，效率更高。

#### lock对象 
- ReentrantLock 允许尝试着获取但最终未获取锁。
- 相比于synchronized锁，lock对象可以做到更细粒度的控制力。


#### 原子性
- 原子操作是不能被线程调度机制中断的操作，一旦操作开始，那么它一定可以在可能发生的，线程上下文切换操作之前执行完毕。
- 原子性可以应用于除long和double之外的所有基本类型之上的基本操作。
- 因为JVM可以将64位（long和double变量）的读取和写入当做两个分离的32位操作来执行，这就产生了在一个读取和写入操作中间发生上下文切换，从而导致结果有误。当然加上volatile，就能获得原子性。
- java中i++操作不是原子性的。具体看如下JVM指令结果
```
public class Atomicity {
  int i;
  void f1() { i++; }
  void f2() { i += 3; }
} /* Output: (Sample)
...
void f1();
  Code:
   0:        aload_0
   1:        dup
   2:        getfield        #2; //Field i:I
   5:        iconst_1
   6:        iadd
   7:        putfield        #2; //Field i:I
   10:        return

void f2();
  Code:
   0:        aload_0
   1:        dup
   2:        getfield        #2; //Field i:I
   5:        iconst_3
   6:        iadd
   7:        putfield        #2; //Field i:I
   10:        return
*///:~
```

### 可见性
- 一个任务作出修改，对其他任务可能是不可视的。
- volatile关键字确保了应用中的可见性。如果一个域被声明为volatile，那么只要对这个域进行了写操作，其他所有的读操作均可以看到这个修改。即便使用了本地缓存，情况也一样，volatile域会立即被写入到主存中，而读取操作就发生在主存中。
- 将一个域标记为volatile，将告诉编译器，不需要执行任务移除和写入操作的优化。


- Collections.synchronizedCollection() 返回同步的集合类，即方法均用对象锁。

#### ThreadLocal
- 通常当做静态域存储，因为ThreadLocal为每个线程都分配了自己的空间。使用当前线程作为key，因此不会出现线程冲突。

#### 退出线程的方法
- 与synchronized相关
1. 线程中使用一个volatile的标志判断退出。
2. 调用Executors的submit方法，获取线程上下文对象Future，调用cancel方法。（注：无法终端正在试图获取synchronized锁或者试图执行I/O操作的线程）
3. 调用ExecutorService的shutdown的方法。
4. 通过检查中断状态Thread.interrupted()，主动调用Thread的interrupt方法实现。（注意处理的时候，如果线程已经调用了interrupt()，如果再调用sleep方法，将抛出interruptException的异常）
- ReentrantLock调用锁的lockInterruptibly()方法。