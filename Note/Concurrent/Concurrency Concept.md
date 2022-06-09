# 高并发基础概念

## 并发和并行

### 并发

并发（Concurrent）是指在操作系统中，是指一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理器上运行。

并发不是真正意义上的“同时进行”，只是CPU把一个时间段划分成几个时间片段(时间区间)，然后在这几个时间区间之间来回切换，由于CPU处理的速度非常快，只要时间间隔处理得当，即可让用户感觉是多个应用程序同时在进行。



### 并行

并行（Parallel）是指当系统有一个以上CPU时，当一个CPU执行一个进程时，另一个CPU可以执行另一个进程，两个进程互不抢占CPU资源。

其实决定并行的因素不是CPU的数量，而是CPU的核心数量，比如一个CPU多个核也可以并行。适合科学计算，后台处理等弱交互场景。



#### 并发和并行对比

- 并发，指的是多个事情，在同一时间段内同时发生了；并行，指的是多个事情，在同一时间点上同时发生了。
- 并发的多个任务之间是互相抢占资源的；并行的多个任务之间是不互相抢占资源的。
- 只有在多CPU或者一个CPU多核的情况中，才会发生并行；否则，看似同时发生的事情，其实都是并发执行的。



## 核心概念
- 分工

分工就是将一个比较大的任务，拆分成多个大小合适的任务，交给合适的线程去完成，强调的是性能。

Java中提供的Executor、Fork/Join和Future都是实现分工的一种方式。

- 同步

在并发编程中的同步，主要指的是一个线程执行完任务后，如何通知其他的线程继续执行，强调的是性能。当线程执行的条件不满足时，线程需要继续等待，一旦条件满足，就需要唤醒等待的线程继续执行。

Java中提供了一些实现线程之间同步的工具类，比如说：CountDownLatch、 CyclicBarrier 等。

- 互斥

同一时刻，只允许一个线程访问共享变量，强调的是线程执行任务的正确性。如果多个线程同时访问同一个共享变量，则可能会发生意想不到的后果，而这种意想不到的后果主要是由线程的可见性、原子性和有序性问题产生的。而解决可见性、原子性和有序性问题的核心，就是互斥。

Java中提供的synchronized、Lock、ThreadLocal、final关键字等都可以解决互斥的问题。

## 导致问题原因
缓存导致的可见性问题、线程切换导致的原子性问题、编译优化导致的有序性问题。

### 可见性
可见性问题，可以这样理解为一个线程修改了共享变量，另一个线程不能立刻看到，这是由于CPU添加了缓存导致的问题。

单核CPU不存在可见性问题，因为在单核CPU上，无论创建了多少个线程，同一时刻只会有一个线程能够获取到CPU的资源来执行任务，即使这个单核的CPU已经添加了缓存。

多核CPU上，每个CPU的内核都有自己的缓存。当多个不同的线程运行在不同的CPU内核上时，这些线程操作的是不同的CPU缓存。一个线程对其绑定的CPU的缓存的写操作，对于另外一个线程来说，不一定是可见的，这就造成了线程的可见性问题。

#### Java中的可见性问题
使用Java语言编写并发程序时，如果线程使用变量时，会把主内存中的数据复制到线程的私有内存，也就是工作内存中，每个线程读写数据时，都是操作自己的工作内存中的数据。

此时，Java中线程读写共享变量的模型与多核CPU类似，原因是Java并发程序运行在多核CPU上时，线程的私有内存，也就是工作内存就相当于多核CPU中每个CPU内核的缓存了。

```java
@Slf4j
public class ConcurrentTest {
    private int viewableCount;
    private void plusViewableCount() {
        viewableCount++;
    }

    @Test
    @SneakyThrows
    public void viewablePrincipleTest(){
        Thread threadA = new Thread(() -> {
            for(int i = 0; i < 1000; i++){
                plusViewableCount();
            }
        });

        Thread threadB = new Thread(() -> {
            for(int i = 0; i < 1000; i++){
                plusViewableCount();
            }
        });

        threadA.start();
        threadB.start();

        threadA.join();
        threadB.join();
        log.info("viewableCount result : " + viewableCount);
    }
}
```

> 执行结果
```
viewableCunt result : 1483
```

而在整个计算的过程中，线程A和线程B都是基于各自工作内存中的viewableCount值进行计算。这就导致了最终的count值小于2000。

### 原子性
原子性是指一个或者多个操作在CPU中执行的过程不被中断的特性。原子性操作一旦开始运行，就会一直到运行结束为止，中间不会有中断的情况发生。

#### 线程切换
在并发编程中，往往设置的线程数目会大于CPU数目，而每个CPU在同一时刻只能被一个线程使用。而CPU资源的分配采用了时间片轮转策略，也就是给每个线程分配一个时间片，线程在这个时间片内占用CPU的资源来执行任务。当占用CPU资源的线程执行完任务后，会让出CPU的资源供其他线程运行，这就是任务切换，也叫做线程切换或者线程的上下文切换。

线程在执行某项操作时，此时由于CPU发生了线程切换，CPU转而去执行其他的任务，中断了当前线程执行的操作，这就会造成原子性问题。

### 有序性


