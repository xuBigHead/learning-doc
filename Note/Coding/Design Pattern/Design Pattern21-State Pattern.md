# 设计模式21-状态模式（State）
## 概述
如果一个实体具备状态，且在不同状态下会在同一业务场景执行不同的业务逻辑时，就可以考虑使用状态模式。

### 优点
- 容易新加状态，封装了状态转移规则，每个状态可以被复用和共享。
- 避免大量的if else结构。

### 缺点
- 状态类膨胀。
- 新加入状态时，可能需要修改现有的状态实现。

## 简单实现
### JDK实现
```java
public interface State {
    /**
     * open mobile
     */
    void open();
    
    /**
     * close mobile
     */
    void close();
}
```

```java
@Slf4j
public class WaitingState implements State{
    private final MobileModel model;

    public WaitingState(MobileModel model) {
        this.model = model;
    }

    @Override
    public void open() {
        log.info("开启手机中。。。");
        this.model.setState(new OpenState(this.model));
    }

    @Override
    public void close() {
        log.info("关闭手机中。。。");
        this.model.setState(new CloseState(this.model));
    }
}
```

```java
@Slf4j
public class OpenState implements State {
    private final MobileModel model;

    public OpenState(MobileModel model) {
        this.model = model;
    }

    @Override
    public void open() {
        log.info("手机已开启");
    }

    @Override
    public void close() {
       log.info("关闭手机中。。。");
        model.setState(new CloseState(this.model));
    }
}
```

```java
@Slf4j
public class CloseState implements State {
    private final MobileModel model;

    public CloseState(MobileModel model) {
        this.model = model;
    }

    @Override
    public void open() {
        log.info("开启手机中。。。");
        this.model.setState(new OpenState(this.model));
    }

    @Override
    public void close() {
        log.info("手机已关闭");
    }
}
```

状态模式中具体实现功能的代码被封装到了 `State` 的实现类中，上下文通过设置不同的 `State` 实现类。
```java
public class MobileModel {
    private State state;

    public MobileModel() { }

    public MobileModel(State state) {
        this.state = state;
    }

    public void setState(State state) {
        this.state = state;
    }

    public void open() {
        state.open();
    }

    public void close() {
        state.close();
    }
}
```

在同一场景中根据不同的状态来调用不同的业务逻辑。
```java
public class DesignPattenTest {
    @Test
    public void stateDesignPattern(){
        MobileModel model = new MobileModel();
        model.setState(new WaitingState(model));
        model.open();
        model.open();
        model.close();
        model.close();
        model.open();
    }
}
```

## 参考资料