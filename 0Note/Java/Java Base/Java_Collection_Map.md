# Map
## HashMap
**Java7与Java8中的HashMap**

JDK7 HashMap结构为数组+链表（发生元素碰撞时，会将新元素添加到链表开头）

JDK8 HashMap结构为数组+链表+红黑树（发生元素碰撞时，会将新元素添加到链表末尾，当HashMap总容量大于等于64，并且某个链表的大小大于等于8，会将链表转化为红黑树（注意：红黑树是二叉树的一种））

**JDK8 HashMap重排序**

如果删除了HashMap中红黑树的某个元素导致元素重排序时，不需要计算待重排序的元素的HashCode码，只需要将当前元素放到（HashMap总长度+当前元素在HashMap中的位置）的位置即可。

### 源码解析

#### HashMap读方法
##### V get(Object key)
> java.util.HashMap.get
```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

> java.util.HashMap.getNode
```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 通过 hash & (table.length - 1)获取该key对应的数据节点的hash槽;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // 首节点是树形节点, 则进入红黑树数的取值流程, 并返回结果;
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 进入链表的取值流程, 并返回结果;
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

> java.util.HashMap.TreeNode.getTreeNode
```java
final TreeNode<K,V> getTreeNode(int h, Object k) {
    return ((parent != null) ? root() : this).find(h, k, null);
}
```

> java.util.HashMap.TreeNode.root
```java
final TreeNode<K,V> root() {
    for (TreeNode<K,V> r = this, p;;) {
        if ((p = r.parent) == null)
            return r;
        r = p;
    }
}
```

> java.util.HashMap.TreeNode.find
```java
final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
    TreeNode<K,V> p = this;
    do {
        int ph, dir; K pk;
        TreeNode<K,V> pl = p.left, pr = p.right, q;
        if ((ph = p.hash) > h)
            p = pl;
        else if (ph < h)
            p = pr;
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        else if (pl == null)
            p = pr;
        else if (pr == null)
            p = pl;
        else if ((kc != null ||
                  (kc = comparableClassFor(k)) != null) &&
                 (dir = compareComparables(kc, k, pk)) != 0)
            p = (dir < 0) ? pl : pr;
        else if ((q = pr.find(h, k, kc)) != null)
            return q;
        else
            p = pl;
    } while (p != null);
    return null;
}
``` 
##### 参考资料
- [HashMap之get方法详解](https://blog.csdn.net/weixin_39667787/article/details/86687414)

#### HashMap写方法
##### V put(K key, V value)
设置指定key的value值并返回旧值，如果已存在则替换旧值

> java.util.HashMap.put
```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

##### V putIfAbsent(K key, V value)
设置指定key的value值并返回旧值，如果已存在则不替换

> java.util.HashMap.putIfAbsent
```java
@Override
public V putIfAbsent(K key, V value) {
    return putVal(hash(key), key, value, true, true);
}
```

> java.util.HashMap.putVal
```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        // 如果当前key没有value，则设置value
        tab[i] = newNode(hash, key, value, null);
    else {
        // 当前key有value，则替换当前value并返回旧的value
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) {
            // 存在key的映射
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                // 判断是否将新value替换旧value
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    
    // 新增key时，计算是否要扩容并返回null
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

##### V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction)
设置指定key的value值并返回新值，如果已存在则替换旧值

> java.util.HashMap.compute
```java
@Override
public V compute(K key,
                 BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    if (remappingFunction == null)
        throw new NullPointerException();
    int hash = hash(key);
    Node<K,V>[] tab; Node<K,V> first; int n, i;
    int binCount = 0;
    TreeNode<K,V> t = null;
    Node<K,V> old = null;
    if (size > threshold || (tab = table) == null ||
        (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((first = tab[i = (n - 1) & hash]) != null) {
        if (first instanceof TreeNode)
            old = (t = (TreeNode<K,V>)first).getTreeNode(hash, key);
        else {
            Node<K,V> e = first; K k;
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k)))) {
                    old = e;
                    break;
                }
                ++binCount;
            } while ((e = e.next) != null);
        }
    }
    V oldValue = (old == null) ? null : old.value;
    V v = remappingFunction.apply(key, oldValue);
    if (old != null) {
        if (v != null) {
            old.value = v;
            afterNodeAccess(old);
        }
        else
            removeNode(hash, key, null, false, true);
    }
    else if (v != null) {
        if (t != null)
            t.putTreeVal(this, tab, hash, key, v);
        else {
            tab[i] = newNode(hash, key, v, first);
            if (binCount >= TREEIFY_THRESHOLD - 1)
                treeifyBin(tab, hash);
        }
        ++modCount;
        ++size;
        afterNodeInsertion(true);
    }
    return v;
}
```

##### V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction)
设置指定key的value值并返回新值，如果已存在则不替换

> java.util.HashMap.computeIfAbsent
```java
@Override
public V computeIfAbsent(K key,
                         Function<? super K, ? extends V> mappingFunction) {
    if (mappingFunction == null)
        throw new NullPointerException();
    int hash = hash(key);
    Node<K,V>[] tab; Node<K,V> first; int n, i;
    int binCount = 0;
    TreeNode<K,V> t = null;
    Node<K,V> old = null;
    if (size > threshold || (tab = table) == null ||
        (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((first = tab[i = (n - 1) & hash]) != null) {
        if (first instanceof TreeNode)
            old = (t = (TreeNode<K,V>)first).getTreeNode(hash, key);
        else {
            Node<K,V> e = first; K k;
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k)))) {
                    old = e;
                    break;
                }
                ++binCount;
            } while ((e = e.next) != null);
        }
        V oldValue;
        if (old != null && (oldValue = old.value) != null) {
            afterNodeAccess(old);
            return oldValue;
        }
    }
    V v = mappingFunction.apply(key);
    if (v == null) {
        return null;
    } else if (old != null) {
        old.value = v;
        afterNodeAccess(old);
        return v;
    }
    else if (t != null)
        t.putTreeVal(this, tab, hash, key, v);
    else {
        tab[i] = newNode(hash, key, v, first);
        if (binCount >= TREEIFY_THRESHOLD - 1)
            treeifyBin(tab, hash);
    }
    ++modCount;
    ++size;
    afterNodeInsertion(true);
    return v;
}
```

##### V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction)
> java.util.HashMap.computeIfPresent
 
```java
public V computeIfPresent(K key,
                          BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    if (remappingFunction == null)
        throw new NullPointerException();
    Node<K,V> e; V oldValue;
    int hash = hash(key);
    if ((e = getNode(hash, key)) != null &&
        (oldValue = e.value) != null) {
        V v = remappingFunction.apply(key, oldValue);
        if (v != null) {
            e.value = v;
            afterNodeAccess(e);
            return v;
        }
        else
            removeNode(hash, key, null, false, true);
    }
    return null;
}
```

##### 参考资料
- [JAVA8 Map新方法：compute，computeIfAbsent，putIfAbsent与put的区别](https://blog.csdn.net/wang_8101/article/details/82191146)

#### 参考资料
