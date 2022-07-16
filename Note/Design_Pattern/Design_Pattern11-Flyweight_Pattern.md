# 设计模式11-享元模式（Flyweight）
## 概述
> Use sharing to support large numbers of fine-grained objects efficiently.

享元模式（Flyweight Pattern）：使用共享对象可有效地支持大量的细粒度的对象，从而节省内存。

## 简单实现
### JDK实现
> com.demo.design.pattern.structural.flyweight.Circle
```java
@Data
public class Circle {
    private int radius;
    private Font font;

    public void draw() {
        System.out.println("Circle: [Font : " + font + ", radius : " + radius + "]");
    }
}
```

调用示例

```java
@Test
public void flyWeightPattern(){
    // 事先定义好需要的Circle对象的共享属性
    Map<Integer, Font> fontMap = new HashMap<>(10);
    fontMap.put(1, new Font("Arial Bold", Font.PLAIN, 1));
    fontMap.put(2, new Font("Arial Bold", Font.BOLD, 2));
    
    List<Circle> circles = new ArrayList<>(100000);
    Circle circle;
    for (int i = 0; i < 100000; ++i) {
        circle = new Circle();
        // 需要大量Circle对象时，共享属性属性可以引用同一个对象
        circle.setFont(fontMap.get(i & 1));
        
        // 非共享属性则每个Circle独有
        circle.setRadius(new Random().nextInt(100));
        circles.add(circle);
    }
    circles.forEach(Circle::draw);
}
```
## 源码解析
### JDK
> java.lang.Integer
```java
public final class Integer extends Number implements Comparable<Integer> {
    /**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
}
```

Integer缓存对象缓存了 -128 到 127 之间的整型值，这是最常用的一部分整型值，JDK也提供了方法来自定义缓存的最大值。
           
> java.lang.Integer.IntegerCache
```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```
## 优点
减少应用程序创建的对象， 降低程序内存的占用， 增强程序的性能。

## 缺点
提高了系统复杂性， 需要分离出外部状态和内部状态， 而且外部状态具有固化特性， 不应该随内部状态改变而改变， 否则导致系统的逻辑混乱。

## 使用场景
- 系统中存在大量的相似对象。

- 细粒度的对象都具备较接近的外部状态， 而且内部状态与环境无关， 也就是说对象没有特定身份。

- 需要缓冲池的场景。

## 参考资料
- [Java设计模式之（十一）——享元模式](https://www.cnblogs.com/ysocean/p/15621446.html)