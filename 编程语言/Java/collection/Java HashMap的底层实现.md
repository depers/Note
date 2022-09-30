# Java HashMap的底层实现

> 2022年9月29日 陕西西安

## 一、概述

1. `List` 和 `Set` 集合接口扩展了 `Collection`接口，但`Map` 没有。
2. 键值对存储在所谓的存储桶（bucket）中，这些存储桶共同构成了所谓的表，实际上是一个内部数组。
3. 在HashMap中存储和检索一个元素（键值对）的时间复杂度是**O(1)**。

## 二、The put API

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

## 三、The get API

在调用`get()`方法检索存储在map中的对象时，在map的内部，会使用相同的散列原理去调用key的`hashcode()`方法，最终得到存储桶的位置（或者说是内部数组的索引）。

若键对象与map中的任何值没有关联或者说value是`null`的情况下，`get()`方法会返回`null`。这里说的这两种情况我们可以调用`containsKey()`方法进行区别。

## 四、集合视图

HashMap提供了三个集合视图，分别是键视图，值视图和条目视图。

### 1. 键视图

* 移除键视图中的元素，会同步体现在原map中

### 2. 值视图

* 移除值视图中的元素，会同步体现在原map中

### 3. 条目视图

* `HashMap`中的元素是无序的，如果使用for each遍历条目（`Map.Entry`）中的键和值时，是没有顺序的

### 4. Fail fast

* 值得注意的是上述三种视图的迭代器都是Fail fast的。
* 如果对 map 进行任何结构修改，在创建迭代器后，将抛出并发修改异常：`java.util.ConcurrentModificationException`。
* 唯一允许的结构修改是通过迭代器本身执行的删除操作。

### 5. 性能表现

* `HashMap`与`LinkedHashMap`和`TreeMap`相比，上面集合视图的遍历性能比较差。

* `HashMap`的迭代发生在最坏情况 **O(n)** 中，其中 n 是其**容量**和**条目数**的总和。

## 五、HashMap的性能

### 1. 分析

1. HashMap的性能受到两个参数的影响：
   * 初始容量（capacity）：容量是桶的数量或底层数组长度，初始容量只是创建期间的容量。需要注意的是 capacity 必须保证为 2 的 n 次方。默认值是16。
   * 负载因子（loadFactor）：负载因子或 LF 是衡量哈希映射在调整大小之前，已添加映射条目（或者说是键值对的数量）的数量占总容量的量度。默认值是0.75。
2. 当哈希映射条目的数量超过 LF 和容量的乘积时，就会发生重新哈希，即创建另一个内部数组，其大小是初始数组的两倍，并且所有条目都被移动到新数组中的新存储桶位置。
3. 低初始容量会降低空间成本，但会增加重新散列的频率。高初始容量会减低重新散列的频率，但是会增加迭代遍历的时间。

### 2. 总结

* 高初始容量对于大量条目以及很少或没有迭代是有好处的。
* 较低的初始容量适用于具有大量迭代的少数条目。

## 六、HashMap的冲突

### 1. 为什么会出现哈希冲突

* 因为根据equals和hashCode契约，Java中两个不相等的对象可以具有相同的哈希代码。
* 因为容量（或者说底层数组）的大小有限。容量越小，碰撞的几率就越高。
* HashMap是通过拉链法解决哈希冲突的：
  1. HashMap通过调用键对象的hashCode()方法，然后计算出存储桶的位置。
  2. 如果两个键对象的桶位置一样，此时就会调用键对象的equals()方法和桶内的对象做比较，若有相等的，就会替换原有键对应的值。若没有就会将他添加到链表的头部。

## Reference

* [The Java HashMap Under the Hood](https://www.baeldung.com/java-hashmap-advanced)
