# Lambda表达式
在Java 8中，为了能够将行为参数化而引入了Lambda表达式。

可以把Lambda表达式理解为简洁地表示可传递的匿名函数的一种方式：它没有名称，但它有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表。

## 语法
Lambda表达式在Java语言中引入了`->`操作符，`->`操作符被称为Lambda表达式的操作符或者箭头操作符，它将Lambda表达式分为两部分：

- 左侧部分指定了Lambda表达式需要的所有参数，Lambda表达式本质上是对接口的实现，Lambda表达式的参数列表本质上对应着接口中方法的参数列表。

- 右侧部分指定了Lambda体，即Lambda表达式要执行的功能，Lambda体本质上就是接口方法具体实现的功能。

Lambda表达式的参数列表的数据类型可以省略不写，因为JVM编译器能够通过上下文推断出数据类型，这就是`类型推断`。
- 左侧部分只有一个参数时，小括号可以省略
- Lambda表达式的参数列表的数据类型可以省略不写，因为JVM编译器能够通过上下文推断出数据类型，这就是“类型推断”

```java
public class LambdaTest {
    public void completedLeft() {
        // 类型String可以省略，由上下文进行类型推断；因为只有一个参数，小括号可以省略
        Consumer<String> consumer = (String e) -> println(e.substring(3, 6));
    }
    
    public void singleLeft() {
        Consumer<String> consumer = e -> println(e.substring(3, 6));
    }
}
```

- 右侧部分只有一行语句时，大括号和`return`关键字可以省略
```java
public class LambdaTest {
    public String completeRight() {
        Supplier<String> supplier = () -> {
            String result = new Random().nextInt(100) + "";
            return result.substring(1);
        };
        return supplier.get();
    }

    public String simpleRight() {
        Supplier<String> supplier = () -> new Random().nextInt(100) + "";
        return supplier.get();
    }
}
```

- 无参无返回值
```java
public interface INoArgsAndNoReturn {
    void execute();
}
```

```java
public class LambdaTest {
    public void noArgsAndNoReturn() {
        INoArgsAndNoReturn lambda = () -> println("execute lambda");
        execute(lambda);
    }

    private void execute(INoArgsAndNoReturn lambda) {
        lambda.execute();
    }
}
```

- 无参有返回值
```java
public interface INoArgsAndHasReturn {
    String execute();
}
```

```java
public class LambdaTest {
    public void noArgsAndHasReturn() {
        INoArgsAndHasReturn lambda = () -> "return lambda result";
        execute(lambda);
    }

    private void execute(INoArgsAndHasReturn lambda) {
        println(lambda.execute());
    }
}
```

- 有参无返回值
```java
public interface IHasArgsAndNoReturn {
    void execute(Integer one, Integer two);
}
```

```java
public class LambdaTest {
    public void hasArgsAndNoReturn() {
        IHasArgsAndNoReturn lambda = (a, b) -> println(a + b);
        execute(lambda);
    }

    private void execute(IHasArgsAndNoReturn lambda) {
        lambda.execute(1, 2);
    }
}
```

- 有参有返回值
```java
public interface IHasArgsAndHasReturn {
    String execute(Integer one, Integer two);
}
```

```java
public class LambdaTest {
    public void hasArgsAndHasReturn() {
        execute((x, y) -> x + " " + y);
    }

    private void execute(IHasArgsAndHasReturn argsReturn) {
        println(argsReturn.execute(1, 2));
    }
}
```

## 使用局部变量
Lambda可以没有限制地捕获（也就是在其主体中引用）实例变量和静态变量。但局部变量必须显式声明为
final，或事实上是 final 。

### 对局部变量的限制
第一，实例变量和局部变量背后的实现有一个关键不同。实例变量都存储在堆中，而局部变量则保存在栈上。如果Lambda可以直接访问局部变量，而且Lambda是在一个线程中使用的，则使用Lambda的线程，可能会在分配该变量的线程将这个变量收回之后，去访问该变量。因此，Java在访问自由局部变量时，实际上是在访问它的副本，而不是访问原始变量。如果局部变量仅仅赋值一次那就没有什么区别了——因此就有了这个限制。

第二，这一限制不鼓励你使用改变外部变量的典型命令式编程模式，这种模式会阻碍很容易做到的并行处理。

## 经验总结
- lambda表达式通过传递代码，来使代码根据条件被延迟执行或不执行。
- 多参数lambda通过创建匹配的函数式接口来实现。

# 方法引用
需要使用方法引用时，目标引用放在分隔符 :: 前，方法的名称放在后面。

- 指向静态方法的方法引用
```java
@Test
public void methodReferences(){
    String str = "methodReferences";
    // String的length方法是一个静态方法，可以直接引用
    Optional.ofNullable(str).map(String::length);
}
```
- 指向任意类型实例方法的方法引用

指向任意类型实例方法的方法引用的思想就是在引用一个对象的方法，而这个对象本身就是Lambda的一个参数。

```java
@Test
public void methodReferences(){
    String str = "methodReferences";
    // 对象 str 本身就是Lambda的参数，可以指向任意类型实例方法的方法引用替换
    Optional.ofNullable(str).map(s -> s.toUpperCase());
    Optional.ofNullable(str).map(String::toUpperCase);
}
```

- 指向现有对象的实例方法的方法引用

## 构造方法引用
构造函数可以利用它的名称和关键字new来创建它的引用：ClassName::new 。功能与指向静态方法的引用类似。

```java
@Test
public void constructorMethodReferences(){
    // ArrayList就可以通过构造器来建立方法引用 ArrayList::new
    Supplier<List<String>> supplier = ArrayList::new;
    List<String> list = supplier.get();
}
```