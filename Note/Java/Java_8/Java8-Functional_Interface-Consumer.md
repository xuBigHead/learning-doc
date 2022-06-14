## Consumer接口
### Consumer接口源码
> java.util.function.Consumer
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

### Consumer接口用例
```java
public class ConsumerDemo {
    public static void main(String[] args) {
        String product = "产品";
        Consumer<String> consumer1 = str -> PrintUtil.println(str.concat("被消费了"));
        consumer1.accept(product);

        Consumer<String> consumer2 = str -> PrintUtil.print(str.concat("已售空，无法消费"));
        // 先执行对象consumer1的accept方法，在执行consumer2的accept方法
        consumer1.andThen(consumer2).accept(product);
    }
}
```
