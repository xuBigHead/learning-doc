# Arrays

## copyOf

> java.util.Arrays

```java
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```



`Arrays.copyOf()`内部实际调用了 `System.arraycopy()` 方法

`System.arraycopy()` 需要目标数组，将原数组拷贝到自定义的数组里或者原数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置 `Arrays.copyOf()` 是系统自动在内部新建一个数组，并返回该数组。

