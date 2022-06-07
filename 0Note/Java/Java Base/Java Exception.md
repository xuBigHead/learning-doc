# Table Of Contents
[TOC]

# Throwable异常类
Throwable是所有异常的父类。

### 源码解析
#### 构造器相关方法
> 构造器
```java
    public Throwable() {
        fillInStackTrace();
    }
    public Throwable(String message) {
        fillInStackTrace();
        detailMessage = message;
    }
    public Throwable(String message, Throwable cause) {
        fillInStackTrace();
        detailMessage = message;
        this.cause = cause;
    }
    public Throwable(Throwable cause) {
        fillInStackTrace();
        detailMessage = (cause==null ? null : cause.toString());
        this.cause = cause;
    }
    protected Throwable(String message, Throwable cause,
                        boolean enableSuppression,
                        boolean writableStackTrace) {
        if (writableStackTrace) {
            fillInStackTrace();
        } else {
            stackTrace = null;
        }
        detailMessage = message;
        this.cause = cause;
        if (!enableSuppression)
            suppressedExceptions = null;
    }
```

> java.lang.Throwable.fillInStackTrace
```java
public synchronized Throwable fillInStackTrace() {
    if (stackTrace != null ||
        backtrace != null /* Out of protocol state */ ) {
        fillInStackTrace(0);
        stackTrace = UNASSIGNED_STACK;
    }
    return this;
}

private native Throwable fillInStackTrace(int dummy);
```


## Error
错误：JVM内部的严重问题。无法恢复。程序人员不用处理。 

### 源码解析
> 构造器
```java

```

## Exception
普通的问题。通过合理的处理，程序还可以回到正常执行流程。要求编程人员要进行处理。

### RuntimeException
RuntimeException及其子类异常也叫非受检异常(unchecked exception)，这类异常是编程人员的逻辑问题，Java编译器不进行强制要求处理。

例如：类型错误转换，数组下标访问越界，空指针异常、找不到指定类等等。

#### 源码解析
### 非RuntimeException
非RuntimeException也叫受检异常(checked exception)，这类异常是由一些外部的偶然因素所引起的。Java编译器强制要求处理。

例如：IOException, FileNotFoundException等等。

#### 源码解析
### 源码解析  
  
# 异常使用
## try-catch
- 多个catch块时候，最多只会匹配其中一个异常类且只会执行该catch块代码，而不会再执行其它的catch块;
- 匹配catch语句的顺序为从上到下，也可能所有的catch都没执行。因此要先catch子类异常再catch父类异常。

## finally
- try、catch、finally三个语句块均不能单独使用，三者可以组成 try-catch-finally、try-catch、try-finally三种结构;
- catch语句可以有一个或多个，finally语句最多一个;
- try、catch、finally三个代码块中变量的作用域为代码块内部，分别独立而不能相互访问。

```java
public void exception() {
    try {
        String errorJsonString = "{";
        JSON.parse(errorJsonString);
    } catch (JSONException e) {
        // 捕获解析json字符串的异常
        log.error("捕获json解析异常: {}", e.getMessage());
    } catch (Exception e) {
        // catch可以有多个，finally最多只能有一个
        log.error("捕获未知异常: {}", e.getMessage());
    } finally {
        log.info("一定会执行的功能");
    }
}
```

## throw
- throw关键字是用于方法体内部，用来抛出一个Throwable类型的异常。
- 如果抛出了检查异常，则还应该在方法头部声明方法可能抛出的异常类型。该方法的调用者也必须检查处理抛出的异常。如果所有方法都层层上抛获取的异常，最终JVM会进行处理，处理也很简单，就是打印异常消息和堆栈信息。
 
## throws
- 方法中存在受检异常，如果不对其捕获，则必须在方法头中显式声明该异常，告知方法调用者此方法有异常，需要进行处理。 
- 方法中存在异常，方法头中使用关键字throws，后面接上要声明的异常。若声明多个异常，则使用逗号分割。
- 父类的方法没有声明异常，则子类继承方法后，也不能声明异常。

```java
public void checkException() throws Exception {
    int randomNumber = new Random().nextInt(10);
    if ((randomNumber & 1) == 1) {
        // 受检异常要么用try-catch进行处理，要么用throws抛出，交给调用者处理
        throw new Exception("受检异常");
    }
    try {
        throw new IOException("IOException也是受检异常");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
public void unCheckException() {
    // 非受检异常可以不用处理，直接抛出
    throw new RuntimeException("非受检异常");
}
```