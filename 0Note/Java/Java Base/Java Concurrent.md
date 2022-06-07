# Table Of Contents
[TOC]

# ThreadLocal相关功能
## ThreadLocal
### 概述
`ThreadLocal`和`Synchronized`都是为了解决多线程中相同变量的访问冲突问题，不同的点是

- `Synchronized`是通过线程等待，牺牲时间来解决访问冲突。
- `ThreadLocal`是通过每个线程单独一份存储空间，牺牲空间来解决冲突，并且相比于`Synchronized`，`ThreadLocal`具有线程隔离的效果，只有在线程内才能获取到对应的值，线程外则不能访问到想要的值。

正因为`ThreadLocal`的线程隔离特性，使他的应用场景相对来说更为特殊一些。在android中Looper、ActivityThread以及AMS中都用到了`ThreadLocal`。当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用`ThreadLocal`。

### 原理
#### threadLocalHashCode属性
通过属性threadLocalHashCode和容量长度len的进行位运算来获取在table中的下标位置。

#### nextHashCode属性
因为nextHashCode属性是static的原因，在每次new ThreadLocal时因为threadLocalHashCode的初始化，会使threadLocalHashCode值自增一次，增量为0x61c88647。

#### HASH_INCREMENT常量
HASH_INCREMENT常量值为0x61c88647。

0x61c88647是斐波那契散列乘数,它的优点是通过它散列(hash)出来的结果分布会比较均匀，可以很大程度上避免hash冲突，已初始容量16为例，hash并与15位运算计算数组下标结果如下：

ThreadLocalMap使用的是线性探测法，均匀分布的好处在于很快就能探测到下一个临近的可用slot，从而保证效率。。

|hashCode	|数组下标    |
| ----      | ----      |
|0x61c88647	|7          |
|0xc3910c8e	|14         |
|0x255992d5	|5          |
|0x8722191c	|12         |
|0xe8ea9f63	|3          |
|0x4ab325aa	|10         |
|0xac7babf1	|1          |
|0xe443238	|8          |
|0x700cb87f	|15         |

总结如下：
- 对于某一ThreadLocal来讲，他的索引值i是确定的，在不同线程之间访问时访问的是不同的table数组的同一位置即都为table[i]，只不过这个不同线程之间的table是独立的。
- 对于同一线程的不同ThreadLocal来讲，这些ThreadLocal实例共享一个table数组，然后每个ThreadLocal实例在table中的索引i是不同的。

### ThreadLocal与内存泄漏
- 因为在有线程复用如线程池的场景中，一个线程的寿命很长，大对象长期不被回收影响系统运行效率与安全，导致内存泄露。

- 使用ThreadLocal的过程中，显式地进行调用remove方法，将WeakReference与值一起删除，避免引起内存泄漏。

   
### 源码解析
#### 基础结构
> java.lang.ThreadLocal
```java
public class ThreadLocal<T> {
   
    private final int threadLocalHashCode = nextHashCode();

    private static AtomicInteger nextHashCode =
        new AtomicInteger();
    
    private static final int HASH_INCREMENT = 0x61c88647;
    
    private static int nextHashCode() {
        // HASH_INCREMENT可以让生成出来的值或者说ThreadLocal的ID较为均匀地分布在2的幂大小的数组中
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }

    public ThreadLocal() {
    }
```

#### set方法
> java.lang.ThreadLocal
```java
public class ThreadLocal<T> {
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    
    ThreadLocalMap getMap(Thread t) {
        // 获取当前Thread类维护的ThreadLocalMap类型属性threadLocals
        return t.threadLocals;
    }

    void createMap(Thread t, T firstValue) {
        // 创建ThreadLocalMap对象并复制给当前Thread对象的属性threadLocals
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

#### get方法
> java.lang.ThreadLocal
```java
public class ThreadLocal<T> {
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        // 设置初始化值并返回
        return setInitialValue();
    }

    private T setInitialValue() {
        // 获取初始化值，默认为null
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }    

    protected T initialValue() {
        return null;
    }
}
```

#### remove方法
> java.lang.ThreadLocal
```java
public class ThreadLocal<T> {
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }        
}
```

## SuppliedThreadLocal
### 概述
`SuppliedThreadLocal`继承了`ThreadLocal`类，对其功能提供了扩展，可以通过函数式接口`Supplier`来设置默认的初始化值。

### 源码解析
> java.lang.ThreadLocal.SuppliedThreadLocal
```java
static final class SuppliedThreadLocal<T> extends ThreadLocal<T> {

    private final Supplier<? extends T> supplier;

    SuppliedThreadLocal(Supplier<? extends T> supplier) {
        this.supplier = Objects.requireNonNull(supplier);
    }

    @Override
    protected T initialValue() {
        return supplier.get();
    }
}
```

## ThreadLocalMap
### 概述
ThreadLocalMap是定义于ThreadLocal类中的默认静太类，没有对外暴露任何public方法，因此只能有ThreadLocal内进行调用。

### 源码解析
#### 基础结构
> java.lang.ThreadLocal.ThreadLocalMap
```java
static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;
        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    /**
     * 初始化容器容量
     */
    private static final int INITIAL_CAPACITY = 16;

    private Entry[] table;

    /**
     * table中元素的数量
     */
    private int size = 0;

    /**
     * 当size >= threshold时，遍历table并删除key为null的元素，如果删除后size >= threshold*3/4时，需要对table进行扩容。
     */
    private int threshold; // Default to 0

    private void setThreshold(int len) {
        // threshold是table大小的2/3
        threshold = len * 2 / 3;
    }

    private static int nextIndex(int i, int len) {
        return ((i + 1 < len) ? i + 1 : 0);
    }

    private static int prevIndex(int i, int len) {
        return ((i - 1 >= 0) ? i - 1 : len - 1);
    }

    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        // 定义一个长度为16的Entry数组table
        table = new Entry[INITIAL_CAPACITY];
        // 位运算，获取存放下标位置
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }

    private ThreadLocalMap(ThreadLocalMap parentMap) {
        Entry[] parentTable = parentMap.table;
        int len = parentTable.length;
        setThreshold(len);
        table = new Entry[len];

        for (int j = 0; j < len; j++) {
            Entry e = parentTable[j];
            if (e != null) {
                @SuppressWarnings("unchecked")
                ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
                if (key != null) {
                    Object value = key.childValue(e.value);
                    Entry c = new Entry(key, value);
                    int h = key.threadLocalHashCode & (len - 1);
                    while (table[h] != null)
                        h = nextIndex(h, len);
                    table[h] = c;
                    size++;
                }
            }
        }
    }
}
```

#### set方法
> java.lang.ThreadLocal.ThreadLocalMap
```java
static class ThreadLocalMap {
    private void set(ThreadLocal<?> key, Object value) {

        // We don't use a fast path as with get() because it is at
        // least as common to use set() to create new entries as
        // it is to replace existing ones, in which case, a fast
        // path would fail more often than not.

        Entry[] tab = table;
        int len = tab.length;
        int i = key.threadLocalHashCode & (len-1);

        for (Entry e = tab[i];
             e != null;
             e = tab[i = nextIndex(i, len)]) {
            ThreadLocal<?> k = e.get();

            if (k == key) {
                // 当前线程key存在，设置value值并返回
                e.value = value;
                return;
            }

            if (k == null) {
                // 如果k为null，表示这个位置的value已经是陈旧的元素，将该旧元素替换
                replaceStaleEntry(key, value, i);
                return;
            }
        }
        
        // 当前下标位置没有Entry对象，则创建Entry对象并设置到当前下标位置    
        tab[i] = new Entry(key, value);
        int sz = ++size;
        if (!cleanSomeSlots(i, sz) && sz >= threshold)
            // 满足没有清除元素且元素数量大于设置的threshold条件
            rehash();
    }

    private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                                   int staleSlot) {
        // 删除所有陈旧元素并设置新元素
        Entry[] tab = table;
        int len = tab.length;
        Entry e;

        int slotToExpunge = staleSlot;
        for (int i = prevIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = prevIndex(i, len))
            if (e.get() == null)
                slotToExpunge = i;

        for (int i = nextIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = nextIndex(i, len)) {
            ThreadLocal<?> k = e.get();

            if (k == key) {
                e.value = value;

                tab[i] = tab[staleSlot];
                tab[staleSlot] = e;

                if (slotToExpunge == staleSlot)
                    slotToExpunge = i;
                cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
                return;
            }

            if (k == null && slotToExpunge == staleSlot)
                slotToExpunge = i;
        }

        tab[staleSlot].value = null;
        tab[staleSlot] = new Entry(key, value);

        if (slotToExpunge != staleSlot)
            cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
    }
   
    private boolean cleanSomeSlots(int i, int n) {
        boolean removed = false;
        Entry[] tab = table;
        int len = tab.length;
        do {
            i = nextIndex(i, len);
            Entry e = tab[i];
            if (e != null && e.get() == null) {
                n = len;
                removed = true;
                i = expungeStaleEntry(i);
            }
        } while ( (n >>>= 1) != 0);
        return removed;
    }

    private void expungeStaleEntries() {
        Entry[] tab = table;
        int len = tab.length;
        for (int j = 0; j < len; j++) {
            Entry e = tab[j];
            if (e != null && e.get() == null)
                expungeStaleEntry(j);
        }
    }
}
```

#### getEntry方法
> java.lang.ThreadLocal.ThreadLocalMap
```java
static class ThreadLocalMap {
    private Entry getEntry(ThreadLocal<?> key) {
        int i = key.threadLocalHashCode & (table.length - 1);
        Entry e = table[i];
        if (e != null && e.get() == key)
            // 指定下标的entry存在且key与当前参数TreadLocal相等，则返回对应的值
            return e;
        else
            return getEntryAfterMiss(key, i, e);
    }

    private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
        Entry[] tab = table;
        int len = tab.length;

        while (e != null) {
            ThreadLocal<?> k = e.get();
            if (k == key)
                return e;
            if (k == null)
                expungeStaleEntry(i);
            else
                i = nextIndex(i, len);
            e = tab[i];
        }
        return null;
    }    

    private int expungeStaleEntry(int staleSlot) {
        Entry[] tab = table;
        int len = tab.length;

        // expunge entry at staleSlot
        tab[staleSlot].value = null;
        tab[staleSlot] = null;
        size--;

        // Rehash until we encounter null
        Entry e;
        int i;
        for (i = nextIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = nextIndex(i, len)) {
            ThreadLocal<?> k = e.get();
            if (k == null) {
                e.value = null;
                tab[i] = null;
                size--;
            } else {
                int h = k.threadLocalHashCode & (len - 1);
                if (h != i) {
                    tab[i] = null;

                    // Unlike Knuth 6.4 Algorithm R, we must scan until
                    // null because multiple entries could have been stale.
                    while (tab[h] != null)
                        h = nextIndex(h, len);
                    tab[h] = e;
                }
            }
        }
        return i;
    }
}
```

#### remove方法
> java.lang.ThreadLocal.ThreadLocalMap
```java
static class ThreadLocalMap {
    private void remove(ThreadLocal<?> key) {
        Entry[] tab = table;
        int len = tab.length;
        int i = key.threadLocalHashCode & (len-1);
        for (Entry e = tab[i];
             e != null;
             e = tab[i = nextIndex(i, len)]) {
            if (e.get() == key) {
                e.clear();
                expungeStaleEntry(i);
                return;
            }
        }
    }    
}
```

#### rehash方法
> java.lang.ThreadLocal.ThreadLocalMap
```java
static class ThreadLocalMap {
    private void rehash() {
        // 删除所有旧的元素
        expungeStaleEntries();

        if (size >= threshold - threshold / 4)
            // 超出设置的threshold*3/4时，进行扩容操作
            resize();
    }

    private void resize() {
        Entry[] oldTab = table;
        int oldLen = oldTab.length;
        // 扩容后的大小为原先的2倍
        int newLen = oldLen * 2;
        Entry[] newTab = new Entry[newLen];
        int count = 0;

        for (int j = 0; j < oldLen; ++j) {
            Entry e = oldTab[j];
            if (e != null) {
                ThreadLocal<?> k = e.get();
                if (k == null) {
                    e.value = null;
                } else {
                    int h = k.threadLocalHashCode & (newLen - 1);
                    while (newTab[h] != null)
                        h = nextIndex(h, newLen);
                    newTab[h] = e;
                    count++;
                }
            }
        }

        setThreshold(newLen);
        size = count;
        table = newTab;
    }
}
```