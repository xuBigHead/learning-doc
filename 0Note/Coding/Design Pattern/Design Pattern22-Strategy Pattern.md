# 设计模式22-策略模式（Strategy）
## 概述
策略模式的重点在于其实现可以去感知随意替换，根据不同的场景调用不同的实现。

### 优点
 - 算法可以自由切换。 
 - 免使用多重条件判断。 
 - 扩展性良好。

### 缺点
- 策略类膨胀。 
- 所有策略类都需要对外暴露。

### 和状态模式的比较
- 策略模式封装了业务逻辑，业务逻辑之间是解耦的；状态模式将业务逻辑封装到了状态中，状态之间存在切换的关系，切换操作可以在上下文或状态自身内部进行。
- 策略模式根据不同的业务场景切换不同的业务逻辑；状态模式通过在同一场景中设置不同的状态来调用不同的业务逻辑。
- 策略模式的业务逻辑增删不影响其他已有的业务逻辑；状态模式的状态增删需要修改其他的状态实现中的切换状态逻辑。
- 策略模式可以直接被 `Client` 调用，状态模式必须通过一个有状态的 `Model` 来被 `Client` 调用。


## JDK实现
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

## SpringBoot实现
通过SpringBoot实现时可以结合工厂模式，将不同的策略实现类注入到容器中。

1. 定义策略类工厂，并提供注册和获取策略实现类方法。

```java
public class StrategyFactory {
    private final static Map<String, IStrategy> STRATEGY_MAP = new ConcurrentHashMap<>();
    
    public static IStrategy getStrategy(String strategyMark) {
        return STRATEGY_MAP.get(strategyMark);
    }
    
    public static void register(String strategyMark, IStrategy strategy) {
        STRATEGY_MAP.put(strategyMark, strategy);
    }
}
```

2. 定义策略接口，如果需要可以定义策略抽象类实现该接口，增加中间层。

```java
public interface IStrategy {
    /**
     * execute strategy method
     */
    void execute();
}
```

```java
public abstract class BaseStrategy implements IStrategy {

}
```

3. 定义具体的策略实现类继承策略抽象类或策略接口
```java
@Slf4j
@Service
@AllArgsConstructor
public class FirstStrategy extends BaseStrategy implements InitializingBean {
    @Override
    public void execute() {
        log.info("execute first strategy method");
    }

    @Override
    public void afterPropertiesSet() {
        log.info("register first strategy implement class when init context");
        StrategyFactory.register("first", this);
    }
}
```

```java
@Slf4j
@Service
@AllArgsConstructor
public class SecondStrategy extends BaseStrategy implements InitializingBean {
    @Override
    public void execute() {
        log.info("execute second strategy method");
    }

    @Override
    public void afterPropertiesSet() {
        log.info("register second strategy implement class when init context");
        StrategyFactory.register("first", this);
    }
}
```

## 源码解析


## 参考资料