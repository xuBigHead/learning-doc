# 设计模式22-策略模式（Strategy）
## 概述
策略模式的重点在于其实现可以去感知随意替换，根据不同的场景调用不同的实现。

## 简单实现
### JDK实现
策略模式需要设计一个接口，为一系列策略模式实现类提供统一的方法对外提供调用。
```java
interface Strategy {
    /**
     * print message
     *
     * @param message message 
     */
    void print(String message);
}
```

通过不同的实现提供不同的业务逻辑。
```java
public class NormalStrategy implements Strategy {
    @Override
    public void print(String message) {
        System.out.println(message);
    }
}
```

```java
public class ErrorStrategy implements Strategy {
    @Override
    public void print(String message) {
        System.err.println(message);
    }
}
```

在不同的场景中，通过调用不同的实现类来完成不同的业务逻辑。
```java
public class DesignPattenTest {
    @Test
    public void strategyDesignPattern() {
        String message = "this is a message";
        Strategy printStrategy = new NormalStrategy();
        printStrategy.print(message);
        printStrategy = new ErrorStrategy();
        printStrategy.print(message);
    }
}
```
## 源码解析

### 优点
 - 算法可以自由切换。 
 - 免使用多重条件判断。 
 - 扩展性良好。

### 缺点
- 策略类膨胀。 
- 所有策略类都需要对外暴露。

## 和状态模式的比较
- 策略模式封装了业务逻辑，业务逻辑之间是解耦的；状态模式将业务逻辑封装到了状态中，状态之间存在切换的关系，切换操作可以在上下文或状态自身内部进行。
- 策略模式根据不同的业务场景切换不同的业务逻辑；状态模式通过在同一场景中设置不同的状态来调用不同的业务逻辑。
- 策略模式的业务逻辑增删不影响其他已有的业务逻辑；状态模式的状态增删需要修改其他的状态实现中的切换状态逻辑。
- 策略模式可以直接被 `Client` 调用，状态模式必须通过一个有状态的 `Model` 来被 `Client` 调用。

## 参考资料