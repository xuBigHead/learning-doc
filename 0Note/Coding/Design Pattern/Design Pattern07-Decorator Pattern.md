# 设计模式07-装饰模式（Decorator）
## 概述
通过实现与被装饰类实现的相同接口或父类，并将被装饰类作为属性注入到装饰器对象中来完成对装饰器模式的应用。

装饰器模式重点在于调用方对整个过程无感知，仍然调用原先实现的接口或父类方法即可。

### 和适配器模式的比较
装饰器与适配器都有一个别名叫做 包装模式(Wrapper)，它们看似都是起到包装一个类或对象的作用，但是使用它们的目的很不一一样。

适配器模式的意义是要将一个接口转变成另一个接口，它的目的是通过改变接口来达到重复使用的目的。

而装饰器模式不是要改变被装饰对象的接口，而是恰恰要保持原有的接口，但是增强原有对象的功能，或者改变原有对象的处理方式而提升性能。

所以这两个模式设计的目的是不同的。

## 简单实践
### JDK实现
1、现在有接口 `DecoratorSource` 及其实现类 `DecoratorSourceImpl`，如果想在不修改已有类且继续通过 `Source` 类型的声明来调用的情况下对execute功能做修改，即可使用装饰器模式。

> com.demo.design.pattern.structural.decorator.DecoratorSource
```java
public interface DecoratorSource {
    /**
     * enhanced target method
     */
    void execute();
}
```

> com.demo.design.pattern.structural.decorator.DecoratorSourceImpl
```java
@Data
@Slf4j
public class DecoratorSourceImpl implements DecoratorSource {
    @Override
    public void execute() {
        log.info("default decorator source");
    }
}
```

2、创建 `SourceDecorator` 的装饰器类，该类要实现 `DecoratorSource` 接口并将要被装饰的 `DecoratorSource` 类型的实现类作为属性。
> com.demo.design.pattern.structural.decorator.SourceDecorator
```java
@Slf4j
public class SourceDecorator implements DecoratorSource {
    private final DecoratorSource source;

    public SourceDecorator(DecoratorSource source) {
        super();
        this.source = source;
    }

    @Override
    public void execute() {
        log.info("before decorator!");
        source.execute();
        log.info("after decorator!");
    }
}
```

3、调用装饰器类 `SourceDecorator` 方式示例如下：
> com.demo.design.pattern.StructuralPatternTest
```java
public class StructuralPatternTest {
    @Test
    public void decoratorPattern(){
        DecoratorSourceImpl decoratorSource = new DecoratorSourceImpl();
        SourceDecorator sourceDecorator = new SourceDecorator(decoratorSource);
        sourceDecorator.execute();
    }
}
```

## 源码解析
### JDK InputStream流
> java.io.FilterInputStream
```java
public class FilterInputStream extends InputStream {
    protected volatile InputStream in;

    protected FilterInputStream(InputStream in) {
        this.in = in;
    }
```

`FilterInputStream` 通过持有实现相同父类的对象，来对该对象功能作增强。

## 参考资料
