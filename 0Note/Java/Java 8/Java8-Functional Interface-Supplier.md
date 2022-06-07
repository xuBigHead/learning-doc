## Supplier接口
### Supplier接口源码
> java.util.function.Supplier
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

### Supplier接口用例
```java
public class SupplierDemo {
    public static void main(String[] args) {
        Supplier<Integer> supplier = () -> new Random().nextInt(100);
        PrintUtil.println("生成100以内随机数：" + supplier.get());
    }
}
```
