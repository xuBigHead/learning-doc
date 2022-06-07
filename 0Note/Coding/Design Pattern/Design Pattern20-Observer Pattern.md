# 设计模式20-观察者模式（Observer）
## 概述
> Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

在对象之间定义一个一对多的依赖，当一个对象状态改变的时候，所有依赖的对象都会得到通知并自动更新。也叫发布订阅模式，能够很好的解耦一个对象改变，自动改变另一个对象这种情况。

## 简单实现
### JDK实现
- 定义被观察者
> com.demo.design.pattern.behavioral.observer.Subject
```java
public interface Subject {
    /**
     * 添加观察者（如果有必要，可以添加remove方法移除观察者接口并在抽象类中实现）
     *
     * @param observer 观察者
     */
    void add(Observer observer);

    /**
     * 被观察者执行操作发生变化
     */
    void operation();
}
```

> com.demo.design.pattern.behavioral.observer.AbstractSubject
```java
public abstract class AbstractSubject implements Subject {
    /**
     * 观察者对象，可以有多个不同的观察者执行不同的行为
     */
    private final List<Observer> observers = new ArrayList<>();

    @Override
    public void add(Observer observer) {
        observers.add(observer);
    }

    void notifyObservers() {
        // 唤醒所有的观察者并同步更新
        for (Observer observer : observers) {
            observer.update();
        }
    }
}
```

> com.demo.design.pattern.behavioral.observer.SubjectImpl
```java
public class SubjectImpl extends AbstractSubject {
    @Override
    public void operation() {
        System.out.println("update self!");
        this.notifyObservers();
    }
}
```

- 定义被观察者
> com.demo.design.pattern.behavioral.observer.Observer
```java
interface Observer {
    /**
     * 更新操作
     */
    void update();
}
```

> com.demo.design.pattern.behavioral.observer.ObserverImpl
```java
public class ObserverImpl implements Observer {
    @Override
    public void update() {
        System.out.println("observer has received and update");
    }
}
```

- 使用示例
```java
@Test
public void observerPattern(){
    Subject sub = new SubjectImpl();
    sub.add(new ObserverImpl());
    sub.operation();
}
```

## 源码解析
### JDK
> java.util.Observer
```java
/**
 * 通过实现 Observer 接口来接受被观察对象发生改变而发出的通知
 */
public interface Observer {
    /**
     * 当被观察对象发生改变时会调用此方法。当改变发生时，调用被观察对象的唤醒方法来通知所有观察者。
     *
     * @param   o     被观察对象.
     * @param   arg   被观察对象唤醒方法参数
     */
    void update(Observable o, Object arg);
}
```

> java.util.Observable
```java
/**
 * This class represents an observable object, or "data"
 * in the model-view paradigm. It can be subclassed to represent an
 * object that the application wants to have observed.
 * <p>
 * An observable object can have one or more observers. An observer
 * may be any object that implements interface <tt>Observer</tt>. After an
 * observable instance changes, an application calling the
 * <code>Observable</code>'s <code>notifyObservers</code> method
 * causes all of its observers to be notified of the change by a call
 * to their <code>update</code> method.
 * <p>
 * The order in which notifications will be delivered is unspecified.
 * The default implementation provided in the Observable class will
 * notify Observers in the order in which they registered interest, but
 * subclasses may change this order, use no guaranteed order, deliver
 * notifications on separate threads, or may guarantee that their
 * subclass follows this order, as they choose.
 * <p>
 * Note that this notification mechanism has nothing to do with threads
 * and is completely separate from the <tt>wait</tt> and <tt>notify</tt>
 * mechanism of class <tt>Object</tt>.
 * <p>
 * When an observable object is newly created, its set of observers is
 * empty. Two observers are considered the same if and only if the
 * <tt>equals</tt> method returns true for them.
 *
 * @author  Chris Warth
 * @see     java.util.Observable#notifyObservers()
 * @see     java.util.Observable#notifyObservers(java.lang.Object)
 * @see     java.util.Observer
 * @see     java.util.Observer#update(java.util.Observable, java.lang.Object)
 * @since   JDK1.0
 */
public class Observable {
    private boolean changed = false;
    private Vector<Observer> obs;

    /** Construct an Observable with zero Observers. */

    public Observable() {
        obs = new Vector<>();
    }

    /**
     * Adds an observer to the set of observers for this object, provided
     * that it is not the same as some observer already in the set.
     * The order in which notifications will be delivered to multiple
     * observers is not specified. See the class comment.
     *
     * @param   o   an observer to be added.
     * @throws NullPointerException   if the parameter o is null.
     */
    public synchronized void addObserver(Observer o) {
        if (o == null)
            throw new NullPointerException();
        if (!obs.contains(o)) {
            obs.addElement(o);
        }
    }

    /**
     * Deletes an observer from the set of observers of this object.
     * Passing <CODE>null</CODE> to this method will have no effect.
     * @param   o   the observer to be deleted.
     */
    public synchronized void deleteObserver(Observer o) {
        obs.removeElement(o);
    }

    /**
     * If this object has changed, as indicated by the
     * <code>hasChanged</code> method, then notify all of its observers
     * and then call the <code>clearChanged</code> method to
     * indicate that this object has no longer changed.
     * <p>
     * Each observer has its <code>update</code> method called with two
     * arguments: this observable object and <code>null</code>. In other
     * words, this method is equivalent to:
     * <blockquote><tt>
     * notifyObservers(null)</tt></blockquote>
     *
     * @see     java.util.Observable#clearChanged()
     * @see     java.util.Observable#hasChanged()
     * @see     java.util.Observer#update(java.util.Observable, java.lang.Object)
     */
    public void notifyObservers() {
        notifyObservers(null);
    }

    /**
     * If this object has changed, as indicated by the
     * <code>hasChanged</code> method, then notify all of its observers
     * and then call the <code>clearChanged</code> method to indicate
     * that this object has no longer changed.
     * <p>
     * Each observer has its <code>update</code> method called with two
     * arguments: this observable object and the <code>arg</code> argument.
     *
     * @param   arg   any object.
     * @see     java.util.Observable#clearChanged()
     * @see     java.util.Observable#hasChanged()
     * @see     java.util.Observer#update(java.util.Observable, java.lang.Object)
     */
    public void notifyObservers(Object arg) {
        /*
         * a temporary array buffer, used as a snapshot of the state of
         * current Observers.
         */
        Object[] arrLocal;

        synchronized (this) {
            /* We don't want the Observer doing callbacks into
             * arbitrary code while holding its own Monitor.
             * The code where we extract each Observable from
             * the Vector and store the state of the Observer
             * needs synchronization, but notifying observers
             * does not (should not).  The worst result of any
             * potential race-condition here is that:
             * 1) a newly-added Observer will miss a
             *   notification in progress
             * 2) a recently unregistered Observer will be
             *   wrongly notified when it doesn't care
             */
            if (!changed)
                return;
            arrLocal = obs.toArray();
            clearChanged();
        }

        for (int i = arrLocal.length-1; i>=0; i--)
            ((Observer)arrLocal[i]).update(this, arg);
    }

    /**
     * Clears the observer list so that this object no longer has any observers.
     */
    public synchronized void deleteObservers() {
        obs.removeAllElements();
    }

    /**
     * Marks this <tt>Observable</tt> object as having been changed; the
     * <tt>hasChanged</tt> method will now return <tt>true</tt>.
     */
    protected synchronized void setChanged() {
        changed = true;
    }

    /**
     * Indicates that this object has no longer changed, or that it has
     * already notified all of its observers of its most recent change,
     * so that the <tt>hasChanged</tt> method will now return <tt>false</tt>.
     * This method is called automatically by the
     * <code>notifyObservers</code> methods.
     *
     * @see     java.util.Observable#notifyObservers()
     * @see     java.util.Observable#notifyObservers(java.lang.Object)
     */
    protected synchronized void clearChanged() {
        changed = false;
    }

    /**
     * Tests if this object has changed.
     *
     * @return  <code>true</code> if and only if the <code>setChanged</code>
     *          method has been called more recently than the
     *          <code>clearChanged</code> method on this object;
     *          <code>false</code> otherwise.
     * @see     java.util.Observable#clearChanged()
     * @see     java.util.Observable#setChanged()
     */
    public synchronized boolean hasChanged() {
        return changed;
    }

    /**
     * Returns the number of observers of this <tt>Observable</tt> object.
     *
     * @return  the number of observers of this object.
     */
    public synchronized int countObservers() {
        return obs.size();
    }
}
```

### EventBus
事件总线，提供了实现观察者模式的骨架代码。Google Guava EventBus 就是一个比较著名的 EventBus 框架，它不仅仅支持异步非阻塞模式，同时也支持同步阻塞模式。

> com.google.common.eventbus.EventBus
```java
@Beta
public class EventBus {
    private static final Logger logger = Logger.getLogger(EventBus.class.getName());
    private final String identifier;
    private final Executor executor;
    private final SubscriberExceptionHandler exceptionHandler; 
    private final SubscriberRegistry subscribers = new SubscriberRegistry(this);
    private final Dispatcher dispatcher;
    // ...
}
```

利用 `EventBus` 框架实现的观察者模式，不需要定义 `Observer` 接口，任意类型的对象都可以注册到 `EventBus` 中，通过 `@Subscribe` 注解来标明类中哪个函数可以接收被观察者发送的消息。

## 优点
- 观察者和被观察者之间抽象耦合

不管是增加观察者还是被观察者都非常容易扩展，在系统扩展方面会得心应手。

- 建立一套触发机制

被观察者变化引起观察者自动变化。但是需要注意的是，一个被观察者，多个观察者，Java的消息通知默认是顺序执行的，如果一个观察者卡住，会导致整个流程卡住，这就是同步阻塞。

所以实际开发中没有先后顺序的考虑使用异步，异步非阻塞除了能够实现代码解耦，还能充分利用硬件资源，提高代码的执行效率。

另外还有不同进程间的观察者模式，通常基于消息队列来实现，用于实现不同进程间的观察者和被观察者之间的交互。

## 使用场景

- 关联行为场景。如用户注册或登陆成功后发送短信提醒，就可以通过观察注册或登陆是否成功来发送短信。

- 事件多级触发场景。

- 跨系统的消息交换场景， 如消息队列的处理机制。

## 参考资料
- [Java设计模式之（十二）——观察者模式](https://www.cnblogs.com/ysocean/p/15627036.html)