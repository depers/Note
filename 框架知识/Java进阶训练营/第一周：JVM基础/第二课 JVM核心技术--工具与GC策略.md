## 第二课 JVM核心技术--工具与GC策略

### 1. JDK内置命令行工具

1. JVM命令行工具（P7）
   * `jps -lmv`
   * `jinfo`
   * `jstat -gc pid 1000 100`/`jstat -gcutil pid 1000 1000`
   * `jstat`
   * `jmap`
   * `jstack`
   * `jcmd`
   * `jrunscript`/`jjs`
   * 问题：
     1. 直接用java -jar 启动一个jar包默认的堆大小是多少，可以使用jmap查看

### 2. JDK内置图形化工具

* jconsole
* jvisualvm
* IDEA visualGC
* jmc

### 3. GC 的背景与一般原理

问题：

1. 标记在阶段Eden 区存活的对象就会复制到存活区，为什么是复制不是移动？

2. 对象从存活区进入老年代，为什么使用的是移动而不是复制？

3. 清算算法、复制算法和整理算法三者的优缺点？（p43）

4. TLAB？

   按照群里面的回答是：线程分配缓存区，当你多个线程去操作内存的时候，如果操作同一块内存，需要保证线程之间分配内存互不影响，这样会带来额外的开销。但是使用TLAB就会给每个线程分配一部分内存。

5. STW

6. `System.gc()`

### 4. 串行GC/并行GC（Serial GC/Parallel GC）
1. 串行GC
2. ParNewGC
3. 并行GC（Parallel GC）
4. JDK各版本的默认GC是什么？
   * JDK 6,7,8 都是并行GC
   * JDK 9以上 是G1

### 5. CMS GC/G1 GC

1. CMS GC

   * 问题：并行Parallel与并发Concurrent的区别

     在JVM的语言环境里面，所谓的并发是指有一部分线程在做GC，有一部分线程在处理业务，业务并没有停。并行GC指的是有多个线程在处理GC，业务线程可能全部停了。

   * 主要用于处理老年代， 新生代用ParNewGC

2. G1 GC

3. GC对比

4. GC组合

5. GC选择

### 6. ZGC/Shenandoah GC

1. ZGC
2. ShennandoahGC
3. GC总结



