# Optional
> @since Java8

从Java8开始提供的Optional容器类，用于尽量避免空指针异常。

> java.util.Optional
```java
public final class Optional<T> {
    /**
     * 默认的空实例
     */
    private static final Optional<?> EMPTY = new Optional<>();

    /**
     * Optional容器封装的对象
     */
    private final T value;
}
```

## Optional创建
> java.util.Optional.Optional()
```java
    private Optional() {
        this.value = null;
    }
```

> java.util.Optional.Optional(T)
```java
    // 私有有参构造器，参数不能为null
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }
```
由于Optional私有化了空构造器，因此只能调用其提供的方法来实例化Optional类。
    
```java
    // 获取空数据的Optional容器类实例
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

    // of方法生成Optional实例，参数不能为null
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }
    
    // ofNullable方法生成Optional实例，参数允许为null
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
```

## Optional方法
## get
获取封装对象，如果为null则抛出异常

> java.util.Optional.get

```java
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
```

## isPresent
判断封装对象是否为null

> java.util.Optional.isPresent

```java
    public boolean isPresent() {
        return value != null;
    }
```

```java
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }
```

```java
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }
```

```java
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```

```java
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }
```

```java
    public T orElse(T other) {
        return value != null ? value : other;
    }
```

```java
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }
```

```java
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }
```