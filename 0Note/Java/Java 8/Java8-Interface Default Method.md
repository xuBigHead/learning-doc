# 接口默认方法
Java 8中接口添加了一个默认方法的新特性，为的是提供一个扩充接口而不会破坏现有代码的方式。

接口中的默认方法用 `default` 来表示，如下所示：

> java.util.Collection
```java
public interface Collection<E> extends Iterable<E> {
    // Collection 接口新增了获取并行流的默认方法，使得所有实现了 Collection 接口的类不在需要实现该方法即可调用
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
}
```

## JDK源码解析
