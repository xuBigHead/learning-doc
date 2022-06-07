## Function接口
### Function接口源码
> java.util.function.Function
```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

### Function接口用例
```java
public class FunctionDemo {
    public static void main(String[] args) {
        Function<Integer, String> function = x -> "生成指定范围内的随机数：" + new Random().nextInt(x);
        PrintUtil.println(function.apply(10));  
    }
}
```
### Function特性化接口
#### IntFunction<R>
##### 源码
```java
@FunctionalInterface
public interface IntFunction<R> {
   R apply(int value);
}
```

#### IntToDoubleFunction

#### IntToLongFunction

#### LongFunction<R>

#### LongToDoubleFunction

#### LongToIntFunction

#### DoubleFunction<R>

#### ToIntFunction<T>

#### ToDoubleFunction<T>

#### ToLongFunction<T>
