# 函数式接口定义及应用
只包含一个抽象方法的接口，允许有默认实现方法和静态实现方法的接口称为函数式接口。

与普通接口只能通过类来实现不同，函数式接口的实现还可以是一个lambda表达式，甚至可以是一个方法引用（method reference）。

## @FunctionalInterface注解
可以在函数式接口上使用 `@FunctionalInterface` 注解，标注它是一个函数式接口，同时javadoc也会包
含一条声明，说明这个接口是一个函数式接口。

如果被 `@FunctionalInterface` 标注的接口不是一个函数式接口，编译器将返回一个提示原因的错误。

```java
@FunctionalInterface
public interface ICalculator<M, N, R> {
    /**
     * 函数式接口只有一个抽象方法
     */
    R calculate(M m, N n);

    /**
     * 函数式接口可以有实现的默认方法
     */
    default void defaultPrint(R result){
        PrintUtil.println("默认方法输出结果：" + result);
    }

    /**
     * 函数式接口可以有实现的静态方法
     */
    static void staticPrint(String result){
        PrintUtil.println("静态方法输出结果：" + result);
    }
}
```

```java
public class CalculatorDemo {
    public static void main(String[] args) {
        ICalculator<Integer, Integer, Integer> calculator = (x, y) -> x * 10 + y;
        // 执行函数式接口ICalculator的calculate方法
        Integer result = calculator.calculate(1, 2);
        // 执行函数式接口的默认方法
        calculator.defaultPrint(result);
        // 执行函数式接口的静态方法
        ICalculator.staticPrint(result.toString());
    }
}
```


# JDK函数式接口
## 核心函数式接口
| 函数式接口 | 参数类型 | 返回类型 | 描述                                                         |
| ---------- | -------- | -------- | ------------------------------------------------------------ |
| Consumer   | T        | void     | 对类型为T的对象应用操作。                                    |
| Supplier   | 无       | T        | 返回类型为T的对象。                                          |
| Function   | T        | R        | 对类型为T的对象应用操作，并R类型的返回结果。                 |
| Predicate  | T        | boolean  | 确定类型为T的对象是否满足约束条件，并返回boolean类型的数据。 |

## 原始类型特化函数式接口
Java 8在现有的函数式接口的基础上还提供了几种原始类型特化的函数式接口。这类函数式接口在输入输出时都是原始类型，避免了自动装箱的操作，从而提高了性能。

下表是一些常用函数式接口及其原始类型特化后的函数式接口对应表。

| 函数式接口         | 函数描述符     | 原始类型特化                                                 |
| ------------------ | -------------- | ------------------------------------------------------------ |
| Predicate<T\>      | T->boolean     | IntPredicate,<br/>LongPredicate, <br/>DoublePredicate        |
| Consumer<T\>       | T->void        | IntConsumer,<br/>LongConsumer,<br/>DoubleConsumer            |
| Function<T,R>      | T->R           | IntFunction<R\>,<br/>IntToDoubleFunction,<br/>IntToLongFunction,<br/>LongFunction<R\>,<br/>LongToDoubleFunction,<br/>LongToIntFunction,<br/>DoubleFunction<R\>,<br/>ToIntFunction<T\>,<br/>ToDoubleFunction<T\>,<br/>ToLongFunction<T\> |
| Supplier<T\>       | ()->T          | BooleanSupplier,<br/>IntSupplier, <br/>LongSupplier,<br/>DoubleSupplier |
| UnaryOperator<T\>  | T->T           | IntUnaryOperator,<br/>LongUnaryOperator,<br/>DoubleUnaryOperator |
| BinaryOperator<T\> | (T,T)->T       | IntBinaryOperator,<br/>LongBinaryOperator,<br/>DoubleBinaryOperator |
| BiPredicate<L,R>   | (L,R)->boolean |                                                              |
| BiConsumer<T,U>    | (T,U)->void    | ObjIntConsumer<T\>,<br/>ObjLongConsumer<T\>,<br/>ObjDoubleConsumer<T\> |
| BiFunction<T,U,R>  | (T,U)->R       | ToIntBiFunction<T,U>,<br/>ToLongBiFunction<T,U>,<br/>ToDoubleBiFunction<T,U> |

## 其他函数式接口

# 参考资料
- [Java中的函数式编程（二）函数式接口Functional Interface](https://www.cnblogs.com/anyuanwai/p/15429735.html)