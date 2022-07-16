# 单例模式
## 概述

单例模式（Singleton）要私有化构造器，在类中创建私有实例并提供公有方法给外部类获取该实例对象，确保只会存在唯一的一个实例。

单例模式优点在于在内存里只有一个实例，减少了内存的开销，尤其是在频繁的创建和销毁实例场景下，避免对资源的多重占用；缺点在于没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。

单例模式分为**懒汉式**（线程不安全，调用效率高，但是不能延时加载）、**饿汉式**（线程安全，调用效率不高，可以延时加载）、**双重检测锁式**（由于JVM底层内部模型原因，偶尔会出问题，不建议使用）**、静态内部类式**（线程安全，调用效率高，可以延时加载）**和枚举单例**（线程安全，调用效率高，不能延时加载）。



## 简单实现

### 基于JDK的实现

#### 懒汉式

线程不安全的懒汉式：

```java
public class Singleton {
    private static Singleton instance = null;
    private Singleton() {}
    public static Singleton getInstance() {
        if(instance == null) { 
            instance = new Singleton();
        }
        return instance;
    }
}
```

这种方式是最基本的实现方式，这种实现最大的问题就是不支持多线程。因为没有加锁 synchronized，所以严格意义上它并不算单例模式。



线程安全的懒汉式：

```java
public class Singleton {
    private static Singleton instance = null;
    private Singleton() {}
    public static synchronized Singleton getInstance() {
        if(instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```



这种方式具备很好的 lazy loading，能够在多线程中很好的工作，但是，效率很低，99% 情况下不需要同步。

优点：第一次调用才初始化，避免内存浪费。

缺点：必须加锁 synchronized 才能保证单例，但加锁会影响效率。



#### 饿汉式

```java
public class Singleton {
    private static Singleton instance = new Singleton();
    private Singleton() { }
    public static Singleton getInstance() { return instance; }
}
```



这种方式比较常用，但容易产生垃圾对象。

优点：没有加锁，执行效率会提高。

缺点：类加载时就初始化，浪费内存。

它基于 classloader 机制避免了多线程的同步问题，不过，instance 在类装载时就实例化，虽然导致类装载的原因有很多种，在单例模式中大多数都是调用 getInstance 方法， 但是也不能确定有其他的方式（或者其他的静态方法）导致类装载，这时候初始化 instance 显然没有达到 lazy loading 的效果。



#### 双检锁式

```java
public class Singleton {
    private volatile static Singleton instance = null;
    private Singleton() {}
    /**
     * 静态工程方法，创建实例，此时毫无线程安全保护，因此要添加synchronized
     * @return      单例对象
     */
    public static Singleton getInstance() {
        if(instance == null) {
            synchronized(Singleton.class) {
                if(instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

这种方式采用双检锁机制（DCL，即 double-checked locking），安全且在多线程情况下能保持高性能。

`uniqueInstance = new Singleton();` 这段代码其实是分为三步执行：

1. 为 `uniqueInstance` 分配内存空间
2. 初始化 `uniqueInstance`
3. 将 `uniqueInstance` 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 `getUniqueInstance`() 后发现 `uniqueInstance` 不为空，因此返回 `uniqueInstance`，但此时 `uniqueInstance` 还未被初始化。

使用 `volatile` 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。



#### 静态内部类式

实际情况是，单例模式使用内部类来维护单例的实现，JVM 内部的机制能够保证当一个类被加载的时候，这个类的加载过程是线程互斥的。这样当我们第一次调用getInstance的时候，JVM 能够帮我们保证instance只被创建一次，并且会保证把赋值给instance的内存初始化完毕， 这样我们就不用担心上面的问题。 同时该方法也只会在第一次调用的时候使用互斥机制，这样就解决了低性能问题。

```java
/**
 * 通过静态内部类的方式来实现单例模式
 * 线程安全，调用效率高，可以延时加载
 */
public class Singleton {
    private Singleton () {
    }

    /**
     * 使用一个内部类来维护单例
     */
    private static class SingletonFactory{
        private static Singleton instance = new Singleton();
    }
    public static Singleton getInstance() {
        return SingletonFactory.instance;
    }
}
```



这种方式能达到双检锁方式一样的功效，但实现更简单。对静态域使用延迟初始化，应使用这种方式而不是双检锁方式。这种方式只适用于静态域的情况，双检锁方式可在实例域需要延迟初始化时使用。

这种方式同样利用了 classloader 机制来保证初始化 instance 时只有一个线程，它跟饿汉式不同的是：饿汉式只要 Singleton 类被装载了，那么 instance 就会被实例化（没有达到 lazy loading 效果），而这种方式是 Singleton 类被装载了，instance 不一定被初始化。因为 SingletonHolder 类没有被主动使用，只有通过显式调用 getInstance 方法时，才会显式装载 SingletonHolder 类，从而实例化 instance。想象一下，如果实例化 instance 很消耗资源，所以想让它延迟加载，另外一方面，又不希望在 Singleton 类加载时就实例化，因为不能确保 Singleton 类还可能在其他的地方被主动使用从而被加载，那么这个时候实例化 instance 显然是不合适的。这个时候，这种方式相比饿汉式就显得很合理。

 

 如果在构造函数中抛出异常， 实例将永远得不到创建， 也会出错。所以说，十分完美的东西是没有的，我们只能根据实际情况，选择最适合自己应用场景的实现方法。也有人这样实现：因为我们只需要在创建类的时候进行同步，所以只要将创建和getInstance()分开，单独为创建加synchronized关键字，也是可以的。考虑性能的话，整个程序只需创建一次实例，所以性能也不会有什么影响。

```java
public class Singleton {
    private static Singleton instance = null;
    private Singleton() {
    }
    public static synchronized void syncInit() {
        if(instance == null) {
            instance = new Singleton();
        }
    }
    public static Singleton getInstance() {
        if(instance == null) {
            syncInit();
        }
        return instance;
    }
}
```



类的静态方法也能实现单例模式，两者的不同处：

静态类不能实现接口，因为接口中不允许有static修饰的方法

单例可以被延迟（比较庞大的类延迟加载有助于提升性能）初始化，静态类一般在第一次加载时初始化。

单例类比较灵活，可以实现一些其它功能，静态类不行。

前面的单例模式的实现，内部就是一个静态类实现的，两者思想的结合，才能造就出完美的解决方案。



#### 枚举式

```java
public enum Singleton {
    INSTANCE;
}
```

这种方式是实现单例模式的最佳方法。它更简洁，不仅能避免多线程同步问题，而且还自动支持序列化机制，防止反序列化重新创建新的对象。



### 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料