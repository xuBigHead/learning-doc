# Stream流
## 构建流
### 由值创建流
### 由数组创建流
### 由文件生成流
### 无限流

## 处理数据
### 筛选和切片
### 映射
### 查找和匹配
### 归约


## 收集数据
### 归约和汇总

### 分组
### 分区

# Stream API
> @since Java8

Stream具备如下特点：

- 非集合，无法存储元素；

- 不改变源对象，而是返回一个持有结果的新Stream；

- 操作是延迟执行的，必须知道有哪些数据才会继续往下执行，即会等到需要结果时才执行；

- 只执行一次，如果要继续操作则需要重新获取Stream对象；

- 类似高级的iterator，单向不可重复，数据只能遍历一次，但是可以并行化数据。



## 串行流和并行流



## 创建操作

常见的创建流的方式有如下几种：

- java.util.Collection#stream

```java
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
```

- java.util.Arrays#stream(T[])

java.util.stream类中的of(T...)方法的实现就是调用的Arrays类的stream(T[])方法。

```java
    public static <T> Stream<T> stream(T[] array) {
        return stream(array, 0, array.length);
    }
```

- java.util.stream.Stream#of(T)

```java
    public static<T> Stream<T> of(T t) {
        return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
    }
```

所有上述创建流的方法都是最终调用了java.util.stream.StreamSupport#stream(java.util.Spliterator<T>, boolean)来创建流

```java
    /*
     * @param spliterator 表示流中元素的描述
     * @param parallel 为true表示创建并行流，否则创建串行流
     */
    public static <T> Stream<T> stream(Spliterator<T> spliterator, boolean parallel) {
        Objects.requireNonNull(spliterator);
        return new ReferencePipeline.Head<>(spliterator,
                                            StreamOpFlag.fromCharacteristics(spliterator),
                                            parallel);
    }
```

### 代码示例

```java
    public void build() {
        Stream<Integer> listStream = Lists.newArrayList(1, 2, 3).stream();

        Integer[] arr = {1, 2, 3};
        Stream<Integer> arrayStream = Arrays.stream(arr);

        Stream<Integer> singleStream = Stream.of(1);
    }
```

## 聚合操作


## 返回结果操作


## 常见应用

### 递归构建树形结构

```java
    public void buildTree() {
        List<StreamNode> nodes = getTreeElementList();
        // 创建根节点
        StreamNode rootNode = new StreamNode(1L, 0L);
        println(JSON.toJSONString(covert(rootNode, nodes), true));
    }

    private StreamNode covert(StreamNode rootNode, List<StreamNode> nodes) {
        // 递归获取子节点
        List<StreamNode> children = nodes.stream()
            .filter(node -> node.getParentId().equals(rootNode.getId()))
            .map(node -> covert(node, nodes)).collect(Collectors.toList());
        rootNode.setChildren(children);
        return rootNode;
    }
```

## 参考资料
- [Java8 Stream groupingBy对List进行分组](https://blog.csdn.net/weixin_41835612/article/details/83687088)