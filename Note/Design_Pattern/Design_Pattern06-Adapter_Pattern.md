# 设计模式06-适配器模式（Adapter）
## 概述 
对适配器模式的功能很好理解，就是把一个类的接口变换成客户端所能接受的另一种接口，从而使两个接口不匹配而无法在一起工作的两个类能够在一起工作。

适配器的重点在于将一个接口的功能转换为另一个接口的功能。

### 优点
- 更好的复用性

系统需要使用现有的类，而此类的接口不符合系统的需要。那么通过适配器模式就可以让这些功能得到更好的复用。

- 更好的扩展性

在实现适配器功能的时候，可以调用自己开发的功能，从而自然地扩展系统的功能。

### 缺点
过多的使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是A接口，其实内部被适配成了B接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。

## 简单实现
### JDK实现
1、已有 `AdapterSource` 接口及其实现类 `AdapterSourceImpl`
> com.demo.design.pattern.structural.adapter.AdapterSource
```java
public interface AdapterSource {
    /**
     * adapter method  
     */
    void execute();
}
```

> com.demo.design.pattern.structural.adapter.AdapterSourceImpl
```java
@Slf4j
public class AdapterSourceImpl implements AdapterSource {
    @Override
    public void execute() {
        log.info("default adapter function");
    }
}
```

2、现在要在使用 `AdapterTarget` 接口处调用 `AdapterSource` 接口方法而不修改 `AdapterSource` 相关类。
> com.demo.design.pattern.structural.adapter.AdapterTarget
```java
public interface AdapterTarget {
    /**
     * call adapter method in this method
     */
    void executeTarget();
}
```

3、可以通过两种适配方式来在 `AdapterTarget` 接口的实现类中调用 `AdapterSource` 实现类的功能。
第一种是通过持有 `AdapterSource` 实现类对象的方式来调用 `AdapterSource` 方法，相比于下述的继承方式，更推荐通过组装的方式完成 `AdapterSource` 接口的适配。
> com.demo.design.pattern.structural.adapter.HoldObjectAdapter
```java
public class HoldObjectAdapter implements AdapterTarget {
    private final AdapterSource source;

    HoldObjectAdapter(AdapterSource source) {
        super();
        this.source = source;
    }

    @Override
    public void executeTarget() {
        source.execute();
    }
}
```

第二种是通过继承 `AdapterSource` 实现类的方式来调用 `AdapterSource` 方法。
> com.demo.design.pattern.structural.adapter.ExtendClassAdapter
```java
public class ExtendClassAdapter extends AdapterSourceImpl implements AdapterTarget {
    @Override
    public void executeTarget() {
        execute();
    }
}
```

示例
```java
public class StructuralPatternTest {
    @Test
    public void adapterPattern(){
        AdapterSource adapterSource = new AdapterSourceImpl();
        AdapterTarget holdObjAdapter = new HoldObjectAdapter(adapterSource);
        holdObjAdapter.executeTarget();
        
        AdapterTarget extendAdapter = new ExtendClassAdapter();
        extendAdapter.executeTarget();           
    }
}
```


## 源码解析
### JDK
> java.io.InputStreamReader
```java
public class InputStreamReader extends Reader {
    private final StreamDecoder sd;
    public InputStreamReader(InputStream in) {
        // ...
    }
}
```

`InputStreamReader` 通过持有不同接口或父类 `InputStream` 类型的实现来将 `InputStream` 适配适配到 `Reader`

### Mybatis Log
> org.apache.ibatis.logging.Log
```java
public interface Log {
    // ...
}
```

`Mybatis` 为了适配多种日志实现类，就使用了适配器模式，将多种日志的实现适配为同一个接口 `Log` 的实现。

> org.apache.ibatis.logging.log4j2.Log4j2LoggerImpl
```java
// ...
import org.apache.logging.log4j.Logger;

public class Log4j2LoggerImpl implements Log {
    private static final Marker MARKER = MarkerManager.getMarker(LogFactory.MARKER);
    // 这里就适配了 apache 的 log4j2 日志实现类
    private final Logger log;
    public Log4j2LoggerImpl(Logger logger) {
        log = logger;
    }
}
```

> org.apache.ibatis.logging.slf4j.Slf4jLoggerImpl
```java
class Slf4jLoggerImpl implements Log {
    private final Logger log;
    // 这里就适配了 slf4j 的日志实现类
    public Slf4jLoggerImpl(Logger logger) {
        log = logger;
    }
```

## 参考资料