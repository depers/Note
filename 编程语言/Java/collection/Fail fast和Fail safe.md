# Fail fast和Fail safe

## 什么是并发修改

并发修改，Concurrent Modification

当一个或多个线程对集合进行迭代时，在此期间，一个线程会更改集合的结构（将元素添加到集合中或删除集合中的元素或更新集合中特定位置的值） 被称为并发修改。

## Fail Fast 迭代器和 Fail Safe 迭代器的区别

### Fail fast Iterator

若集合结构发生修改。会立即抛出并发修改异常。面对并发修改，迭代器快速而干净的失败，而不是在未来不确定的时间发生任意的、不确定的行为。

Fail fast迭代器会抛出`ConcurrentModificationException` ，在下面两种情况下：

1. 单线程环境

   在迭代器创建之后，结构被除迭代器自己的remove()方法之外的任何方法修改时。

2. 多线程环境

   如果一个线程正在修改集合的结构，而另一个线程正在对其进行迭代。

根据 Oracle 文档，不能保证迭代器的快速失败行为，因为一般来说，在存在不同步的并发修改的情况下，不可能做出任何硬保证。 快速失败的迭代器会尽最大努力抛出 ConcurrentModificationException。 **因此，编写一个依赖于这个异常的正确性的程序是错误的：迭代器的快速失败行为应该只用于检测错误。**

### 面试官：Fail Fast Iterator 是怎么知道内部结构被修改的？



## Reference

* [Fail Fast Vs Fail Safe Iterator In Java : Java Developer Interview Questions](https://javahungry.blogspot.com/2014/04/fail-fast-iterator-vs-fail-safe-iterator-difference-with-example-in-java.html)