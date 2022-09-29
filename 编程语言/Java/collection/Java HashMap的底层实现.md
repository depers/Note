# Java HashMap的底层实现

> 2022年9月29日 陕西西安

## 概述

1. `List` 和 `Set` 集合接口扩展了 `Collection`接口，但`Map` 没有。
2. 键值对存储在所谓的存储桶（bucket）中，这些存储桶共同构成了所谓的表，实际上是一个内部数组。
3. 在HashMap中存储和检索一个元素（键值对）的时间复杂度是**O(1)**。

## The put API

### 1. put方法的底层介绍

当我们使用一个类对象作为Key时，调用`put(k, v)`方法，它会默认调用作为Key的类对象的`hashcode()`方法。这段底层代码如下：

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

接着我们看一下这个put()方法：

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

上面的这段代码你可能注意到`putVal()`方法的第二个参数key，明明我们已经计算了一个key的hash值，为啥还要再次使用这个参数呢？**原因是哈希映射在存储桶位置中将键和值都存储为 `Map.Entry`对象**。

### 2. Map接口为什么不继承Collection接口

原因是Map不像其他集合那样完全存储单个元素，而是键值对的集合。所以Collection接口的泛型方法如`add`、`toArray`对Map来说是没有意义的。

### 3. Map可以接受null键和null值

哈希映射的一个特殊属性是它接受空值和空键。

在 put 操作期间遇到空键时，会自动为其分配最终哈希值 0，这意味着它成为底层数组的第一个元素，也就是会存储在第一个桶中。

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

在 `put()` 操作期间，当我们使用之前已经使用过的键来存储值时，它会返回与该键关联的先前值。若我们保存的是一个新的键值对，`put()`操作会返回`null`。

## The get API

在调用`get()`方法检索存储在map中的对象时，在map的内部，会使用相同的散列原理去调用key的`hashcode()`方法，最终得到存储桶的位置（或者说是内部数组的索引）。

若键对象与map中的任何值没有关联或者说value是`null`的情况下，`get()`方法会返回`null`。这里说的这两种情况我们可以调用`containsKey()`方法进行区别。