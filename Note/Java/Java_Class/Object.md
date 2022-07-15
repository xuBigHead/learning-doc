# Object类

## equals方法

### 源码解析

> java.lang.Object

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```



通常会重写equals()方法：若两个对象的内容相等，则equals()方法返回true；否则，返回fasle。



### equals方法特性要求

1. 对称性：如果x.equals(y)返回是"true"，那么y.equals(x)也应该返回是"true"。
2. 反射性：x.equals(x)必须返回是"true"。
3. 类推性：如果x.equals(y)返回是"true"，而且y.equals(z)返回是"true"，那么z.equals(x)也应该返回是"true"。
4. 一致性：如果x.equals(y)返回是"true"，只要x和y内容一直不变，不管你重复x.equals(y)多少次，返回都是"true"。
5. 非空性，x.equals(null)，永远返回是"false"；x.equals(和x不同类型的对象)永远返回是"false"。



### == 与 equals

**==** 的作用是判断两个对象的地址是不是相等。即判断两个对象是不是同一个对象(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)。

**equals()** : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

- 情况 1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
- 情况 2：类覆盖了 equals() 方法。一般，覆盖 equals() 方法来比较两个对象的内容是否相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。



```java
public void equals(){
    String a = new String("ab"); // a 为一个引用
    String b = new String("ab"); // b为另一个引用,对象的内容一样
    String aa = "ab"; // 放在常量池中
    String bb = "ab"; // 从常量池中查找
    if (aa == bb) // true
        System.out.println("aa==bb");
    if (a == b) // false，非同一对象
        System.out.println("a==b");
    if (a.equals(b)) // true
        System.out.println("a equals b");
    if (42 == 42.0) { // true
        System.out.println("true");
    }
}
```



## hashCode方法

> java.lang.Object

```java
public native int hashCode();
```

hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个 int 整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode()定义在 JDK 的 Object 类中，这就意味着 Java 中的任何类都包含有 hashCode() 函数。另外需要注意的是： Object 的 hashcode 方法是本地方法，也就是用 c 语言或 c++ 实现的，该方法通常用来将对象的 内存地址 转换为整数之后返回。

哈希表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了哈希码！

**hashCode() 在哈希表中才有用，在其它情况下没用。**在哈希表中hashCode() 的作用是获取对象的哈希码，进而确定该对象在哈希表中的位置。

哈希表根据元素的哈希码计算出元素在哈希表中的位置，然后将元素插入该位置即可。对于相同的元素只保存了一个。

Java集合中本质是哈希表的类，如HashMap，Hashtable，HashSet。这些类通过hashCode()方法保证数据的唯一性。



**为什么要有 hashCode？**

当把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与其他已经加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 equals() 方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。这样大大减少了 equals 的次数，相应就大大提高了执行速度。



**为什么重写 equals 时必须重写 hashCode 方法**

如果两个对象相等，则 hashcode 一定也是相同的。两个对象相等,对两个对象分别调用 equals 方法都返回 true。

但是，两个对象有相同的 hashcode 值，它们也不一定是相等的 。

**因此，equals 方法被覆盖过，则 `hashCode` 方法也必须被覆盖。**

hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。



**为什么两个对象有相同的 hashcode 值，它们也不一定是相等的**

因为 hashCode() 所使用的杂凑算法也许刚好会让多个对象传回相同的杂凑值。越糟糕的杂凑算法越容易碰撞，但这也与数据值域分布的特性有关（所谓碰撞也就是指的是不同的对象得到相同的 hashCode。

如果 HashSet 在对比的时候，同样的 hashcode 有多个对象，它会使用 equals() 来判断是否真的相同。也就是说 hashcode 只是用来缩小查找成本。





