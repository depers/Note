---
ForkJointypora-root-url: 笔记图片
---

# Java并发编程

## 第1章  课程准备

### 1-1 课程导学

![](E:\markdown笔记\笔记图片\8\1.png)

![2](E:\markdown笔记\笔记图片\8\2.png)

![3](E:\markdown笔记\笔记图片\8\3.png)

![4](E:\markdown笔记\笔记图片\8\4.png)

### 1-2 并发编程初体验

实现一个计数功能

1. 计数count，同时并发的线程数为200，参见代码com.bravedawn.concurrency.example.count.CountExample

   ![](E:\markdown笔记\笔记图片\8\5.png)

2. 计数map，同时并发的线程数为200，参加代码com.bravedawn.concurrency.example.count.MapExample

   ![](E:\markdown笔记\笔记图片\8\6.png)

   可以看到上面两个实验的计数结果都不能达到5000。

3. 计数count，同时并发的线程数为1，参见代码com.bravedawn.concurrency.example.count.CountExample

   ![](E:\markdown笔记\笔记图片\8\7.png)

4. 计数map，同时并发的线程数为1，参加代码com.bravedawn.concurrency.example.count.MapExample

   ![](E:\markdown笔记\笔记图片\8\8.png)

   当同时并发的线程数修改为1时，计数器每次都能到达5000。

### 1-3 并发与高并发基本概念

![](E:\markdown笔记\笔记图片\8\9.png)

![10](E:\markdown笔记\笔记图片\8\10.png)

![11](E:\markdown笔记\笔记图片\8\11.png)

![12](E:\markdown笔记\笔记图片\8\12.png)

![](E:\markdown笔记\笔记图片\8\13.png)

## 第2章 并发基础

### 2-1 CPU多级缓存-缓存一致性

![](E:\markdown笔记\笔记图片\8\14.png)

![15](E:\markdown笔记\笔记图片\8\15.png)

![16](E:\markdown笔记\笔记图片\8\16.png)

![17](E:\markdown笔记\笔记图片\8\17.png)

**`MESI`**（`Modified Exclusive Shared Or Invalid`）(也称为伊利诺斯协议，是因为该协议由伊利诺斯州立大学提出）是一种广泛使用的支持写回策略的缓存一致性协议。

#### MESI协议的背景（他要解决的问题）

基于高速缓存的存储系统交互很好的解决了处理器与内存速度的矛盾，但是也为计算机系统带来更高的复杂度，因为引入了一个新问题：缓存一致性。

在多处理器的系统中(或者单处理器多核的系统)，每个处理器(每个核)都有自己的高速缓存，而它们有共享同一主内存(Main Memory)。

当多个处理器的运算任务都涉及同一块主内存区域时，将可能导致各自的缓存数据不一致。

为此，需要各个处理器访问缓存时都遵循一些协议，在读写时要根据协议进行操作，来维护缓存的一致性。

#### MESI协议中的状态

`CPU`中每个缓存行（`caceh line`)使用4种状态进行标记（使用额外的两位(`bit`)表示):

**M: 被修改（Modified)**

该缓存行只被缓存在该`CPU`的缓存中，并且是被修改过的（`dirty`)，即与主存中的数据不一致，该缓存行中的内存需要在未来的某个时间点（允许其它`CPU`读取请主存中相应内存之前）写回（`write back`）主存。

当被写回主存之后，该缓存行的状态会变成独享（`exclusive`)状态。

**E: 独享的（Exclusive)**

该缓存行只被缓存在该`CPU`的缓存中，它是未被修改过的（`clean`)，与主存中数据一致。该状态可以在任何时刻当有其它`CPU`读取该内存时变成共享状态（`shared`)。

同样地，当`CPU`修改该缓存行中内容时，该状态可以变成`Modified`状态。

**S: 共享的（Shared)**

该状态意味着该缓存行可能被多个`CPU`缓存，并且各个缓存中的数据与主存数据一致（`clean`)，当有一个`CPU`修改该缓存行中，其它`CPU`中该缓存行可以被作废（变成无效状态（`Invalid`））。

**I: 无效的（Invalid）**

该缓存是无效的（可能有其它`CPU`修改了该缓存行）。

状态之间的相互转换关系也可以使用下表进行表示：

![](E:\markdown笔记\笔记图片\8\18.png)

#### MESI协议的操作

* **Local Read**：读本地缓存中的数据
* **Local Write**：将数据写到本地的缓存中
* **Remote Read**：读取内存中的数据
* **Remote Write**：将数据写回到内存中去

![](E:\markdown笔记\笔记图片\8\19.png)

参考文章：https://www.cnblogs.com/z00377750/p/9180644.html

###2-2 CPU多级缓存-乱序执行优化

为了使得处理器内部的运算单元尽量被充分利用，提高运算效率，处理器可能会对输入的代码进行乱序执行。

![](E:\markdown笔记\笔记图片\8\20.png)

###2-3 JAVA内存模型

> 参考文章：https://zhuanlan.zhihu.com/p/29881777

#### 定义

我们常说的JVM内存模式指的是JVM的内存分区；而Java内存模式是一种虚拟机规范。

**Java虚拟机规范中定义了Java内存模型（Java Memory Model，JMM），用于屏蔽掉各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的并发效果，JMM规范了Java虚拟机与计算机内存是如何协同工作的：规定了一个线程如何和何时可以看到由其他线程修改过后的共享变量的值，以及在必须时如何同步的访问共享变量。**

原始的Java内存模型存在一些不足，因此Java内存模型在Java1.5时被重新修订。这个版本的Java内存模型在Java8中仍然在使用。

#### JVM对Java内存模型的实现

下图展示了**调用栈和本地变量都存储在栈区**，**对象都存储在堆区**：

![](E:\markdown笔记\笔记图片\8\21.png)

一个本地变量如果是**原始类型**，那么它会被完全存储到**栈区**。（原始类型一共有8种，它们分别是char,boolean,byte,short,int,long,float,double）

一个本地变量也有可能是一个**对象的引用**，这种情况下，这个**本地引用会被存储到栈**中，但是**对象本身仍然存储在堆区**。

对于一个对象的**成员方法**，这些方法中包含本地变量，仍需要存储在**栈区**，即使它们**所属的对象在堆区**。
对于一个对象的**成员变量**，不管它是原始类型还是包装类型，都会被存储到**堆区**。

**Static类型的变量**以及类本身相关信息都会随着类本身存储在**堆区**。

存放在堆上的对象可以被所有持有对这个对象引用的线程访问。当一个线程可以访问一个对象时，它也可以访问这个对象的成员变量。**如果两个线程同时调用同一个对象上的同一个方法，它们将会都访问这个对象的成员变量，但是每一个线程都拥有这个成员变量的私有拷贝。**

**堆（Heap）**：

是运行时的数据区，堆是由垃圾回收机制来负责的。

堆的优势是可以动态的分配内存大小，生存期也不必事先告诉编译器，他是在运行时动态分配内存的。Java的垃圾收集器会自动收走这些不再使用的数据。

堆的缺点是因为要在运行时动态分配内存，因此它的存取速度相对慢一些。

**栈（stack）**：

栈的优势是存取速度要比堆要快，速度仅次于计算机中的寄存器。

栈的数据是可以共享的。

栈的缺点是存在栈中的数据大小和生存期必须是确定的，缺乏灵活性。

#### 硬件内存架构

现代硬件内存模型与Java内存模型有一些不同，理解内存模型架构以及Java内存模型如何与它协同工作也是非常重要的。

现代计算机硬件架构的简单图示：

![](E:\markdown笔记\笔记图片\8\23.png)

- **多CPU**：一个现代计算机通常由两个或者多个CPU。其中一些CPU还有多核。从这一点可以看出，在一个有两个或者多个CPU的现代计算机上同时运行多个线程是可能的。每个CPU在某一时刻运行一个线程是没有问题的。这意味着，如果你的Java程序是多线程的，在你的Java程序中每个CPU上一个线程可能同时（并发）执行。
- **CPU寄存器**：每个CPU都包含一系列的寄存器，它们是CPU内存的基础。CPU在寄存器上执行操作的速度远大于在主存上执行的速度。这是因为CPU访问寄存器的速度远大于主存。
- **高速缓存cache**：由于计算机的存储设备与处理器的运算速度之间有着几个数量级的差距，所以现代计算机系统都不得不加入一层读写速度尽可能接近处理器运算速度的高速缓存（Cache）来作为内存与处理器之间的缓冲：将运算需要使用到的数据复制到缓存中，让运算能快速进行，当运算结束后再从缓存同步回内存之中，这样处理器就无须等待缓慢的内存读写了。CPU访问缓存层的速度快于访问主存的速度，但通常比访问内部寄存器的速度还要慢一点。每个CPU可能有一个CPU缓存层，一些CPU还有多层缓存。在某一时刻，一个或者多个缓存行（cache lines）可能被读到缓存，一个或者多个缓存行可能再被刷新回主存。
- **内存**：一个计算机还包含一个主存。所有的CPU都可以访问主存。主存通常比CPU中的缓存大得多。
- **运作原理**：通常情况下，当一个CPU需要读取主存时，它会将主存的部分读到CPU缓存中。它甚至可能将缓存中的部分内容读到它的内部寄存器中，然后在寄存器中执行操作。当CPU需要将结果写回到主存中去时，它会将内部寄存器的值刷新到缓存中，然后在某个时间点将值刷新回主存。

#### Java内存模型和硬件内存架构之间的桥接

Java内存模型与硬件内存架构之间存在差异。硬件内存架构没有区分线程栈和堆。对于硬件，所有的线程栈和堆都分布在主内存中。部分线程栈和堆可能有时候会出现在CPU缓存中和CPU内部的寄存器中。如下图所示：

![](E:\markdown笔记\笔记图片\8\24.jpg)

下图是**Java内存模型抽象结构示意图**，从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：

- 线程之间的**共享变量**存储在**主内存**（Main Memory）中
- **每个线程**都有一个私有的**本地内存**（Local Memory），本地内存是JMM的一个抽象概念，并不真实存在，它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。本地内存中存储了该线程以读/写共享变量的拷贝副本。
- 从更低的层次来说，主内存就是硬件的内存，而为了获取更好的运行速度，虚拟机及硬件系统可能会让工作内存优先存储于寄存器和高速缓存中。
- Java内存模型中的**线程的工作内存**（working memory）是cpu的寄存器和高速缓存的抽象描述。而**JVM的静态内存储模型（JVM内存模型）**只是一种对内存的物理划分而已，它只局限在内存，而且只局限在JVM的内存。

![](E:\markdown笔记\笔记图片\8\25.jpg)

#### JMM模型下的线程间通信

线程间通信必须要经过主内存。

如下，如果线程A与线程B之间要通信的话，必须要经历下面2个步骤：

1）线程A把本地内存A中更新过的共享变量刷新到主内存中去。

2）线程B到主内存中去读取线程A之前已更新过的共享变量。

![](E:\markdown笔记\笔记图片\8\26.jpg)

关于主内存与工作内存之间的具体交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存**同步**到主内存之间的实现细节，Java内存模型定义了以下**八种操作**来完成：

- **lock（锁定）**：作用于**主内存**的变量，把一个变量标识为一条线程独占状态。
- **unlock（解锁）**：作用于**主内存**变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- **read（读取）**：作用于**主内存**变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
- **load（载入）**：作用于**工作内存**的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
- **use（使用）**：作用于**工作内存**的变量，**把工作内存中的一个变量值传递给执行引擎**，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
- **assign（赋值）**：作用于**工作内存**的变量，它**把一个从执行引擎接收到的值赋值给工作内存的变量**，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- **store（存储）**：作用于**工作内存**的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。
- **write（写入）**：作用于**主内存**的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。

Java内存模型还规定了在执行上述八种基本操作时，必须满足如下**规则**：

- 如果要把一个变量从主内存中复制到工作内存，就需要按顺寻地执行read和load操作， 如果把变量从工作内存中同步回主内存中，就要按顺序地执行store和write操作。但Java内存模型只要求上述操作必须按顺序执行，而没有保证必须是连续执行。
- 不允许read和load、store和write操作之一单独出现（*<u>这个两组操作中的两步是按顺序进行的，但是并不是连续的，例如read和load之间是可以插入别的操作的</u>*）
- 不允许一个线程丢弃它的最近assign的操作，即变量在工作内存中改变了之后必须同步到主内存中。
- 不允许一个线程无原因地（没有发生过任何assign操作）把数据从工作内存同步回主内存中。
- 一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量。即就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。
- 一个变量在同一时刻只允许一条线程对其进行lock操作，但lock操作可以被同一条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。lock和unlock必须成对出现
- 如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值
- 如果一个变量事先没有被lock操作锁定，则不允许对它执行unlock操作；也不允许去unlock一个被其他线程锁定的变量。
- 对一个变量执行unlock操作之前，必须先把此变量同步到主内存中（执行store和write操作）。

![](E:\markdown笔记\笔记图片\8\27.png)



##第4章 线程安全性

### 1.线程安全性-原子性

* 线程安全性的定义

  ![](E:\markdown笔记\笔记图片\8\8-1.png)

* 线程安全性的组成

  ![8-2](E:\markdown笔记\笔记图片\8\8-2.png)

#### 原子性-Atomic包

* AtomicXXX：CAS、`Unsafe.compareAndSwapInt`（**atomic包下的方法具有原子性**）

  参见com.bravedawn.concurrency.example.count.CountExample2

  * CAS取的其实就是compareAndSwapInt的三个首字母，调用该方法的源码如下：

    ```java
    // public static AtomicInteger count = new AtomicInteger(0);
    count.incrementAndGet();
    ```

    这里的`incrementAndGet`方法是基于类unsafe的实现：

    ```java
    // java.util.concurrent.atomic.AtomicInteger
    public final int incrementAndGet() {
    	return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }
    ```

    `Unsafe`类下的`getAndAddInt`方法：

    ```java
    // sun.misc.Unsafe
    public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
        	var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    
        return var5;
    }
    ```

    上面三段代码是我们调用Java底层实现的`compareAndSwapInt`方法的步骤，在第三段代码段中，var1也就是count，var2是当前对象的值，var4是要增加的值，var5是底层的值。compareAndSwapInt方法在**当前的值var2（工作内存）**与**底层的值var5（主内存）**相等时，才会执行var5 + var4。（**这个也是CAS的原理**）

   * 为什么对象count的值var2会与底层的值var5不相同呢？

     对象count的值var2是工作内存的值，而var5是主内存中的值。他们的值不一定是相等的。需要我们做同步操作使得他们在某一时刻是相等的。

 * `AtomicLong`、`LongAdder`

   参见：com.bravedawn.concurrency.example.atomic.AtomicExample2；com.bravedawn.concurrency.example.atomic.AtomicExample3

   * AtomicLong是作用是对长整形进行原子操作，显而易见，在java1.8中新加入了一个新的原子类LongAdder，该类也可以保证Long类型操作的原子性，相对于AtomicLong，LongAdder有着更高的性能和更好的表现，可以完全替代AtomicLong的来进行原子操作。
   * 在32位操作系统中，64位的long 和 double 变量由于会被JVM当作两个分离的32位来进行操作，所以不具有原子性。而使用AtomicLong能让long的操作保持原子型。
   * 【**LongAdder优点**】LongAdder在AtomicLong的基础上将单点的更新压力分散到各个节点，在低并发的时候通过对base的直接更新可以很好的保障和AtomicLong的性能基本保持一致，而在高并发的时候通过分散提高了性能。 
   * 【**LongAdder缺点**】缺点是LongAdder在统计的时候如果有并发更新，可能导致统计的数据有误差。
   * 【博客】[AtomicLong和LongAdder的区别](https://blog.csdn.net/yao123long/article/details/63683991)
   * 【**LongAddr的使用场景**】
     * 高并发计数使用LongAddr。在线程竞争很低的情况下，使用AtomicLong效率会很高

 * `AtomicBoolean#compareAndSet`

   参见：com.bravedawn.concurrency.example.atomic.AtomicExample6

    * 在实际项目中，我只**希望一件事情只执行一次**，在执行之前该事件的标记可能为false，一旦执行之后标记就会变成true。此时我们调用AtomicBoolean#compareAndSet(false, ture)方法，就可保证对于我们需要执行的那一段代码以原子方式只执行一次。可以理解为同一时间只有一个线程会执行这段代码。

* `AtomicReference`

  参见：com.bravedawn.concurrency.example.atomic.AtomicExample4

  * AtomicReference类提供了一个**可以原子读写的对象引用变量**

  * AtomicReference.compareAndSet的示例代码中我们调用了unsafe.compareAndSwapObject方法，与先前介绍的CAS方法原理一致

* `AtomicReferenceFieldUpdater`

  参见：com.bravedawn.concurrency.example.atomic.AtomicExample5

  * AtomicReferenceFieldUpdater是基于反射的原子更新字段的值，是**用来更新某一个类指定的字段的值**
  * 要求该字段：
    * 字段必须是**volatile类型**的
    * 只能是**实例变量**，不能是类变量，也就是说不能加static关键字

* `AtomicStampReference`：CAS的ABA问题
  
* ABA问题
  
  1、可以发现，CAS实现的过程是先取出内存中某时刻的数据，在下一时刻比较并替换，那么在这个时间差会导致数据的变化，此时就会导致出现“ABA”问题。 
  
    2、什么是”ABA”问题？ 
  比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且two进行了一些操作变成了B，然后two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后one操作成功。
  
 * AtomicStampReference中的核心方法`compareAndSet`。除了对象值，AtomicStampedReference内部还维护了一个“状态戳” **stamp**。状态戳可类比为时间戳，是一个整数值，每一次修改对象值的同时，也要修改状态戳，从而区分相同对象值的不同状态。当AtomicStampedReference设置对象值时，对象值以及状态戳都必须满足期望值，写入才会成功
  
 * `AtomicLongArray`：根据索引原子更新长整型数组里的元素

### 2.原子性——锁（JDK中提供原子性的除了atomic之外还有锁——下面的两个锁）

![8-3](E:\markdown笔记\笔记图片\8\8-3.png)

* **synchronized**它是一种同步锁

  ![8-4](E:\markdown笔记\笔记图片\8\8-4.png)

  

  * 为什么要是用线程池？

    使用线程池的目的是为了查看同一对象的两个进程同时调用函数时的情况。

  * synchronized关键字不能继承

    * 子类继承父类时，如果没有重写父类中的同步方法，子类的同一对象，在不同线程并发调用该方法时，具有同步效果。
    * 子类继承父类，并且重写父类中的同步方法，但没有添加关键字synchronized，子类的同一对象，在不同线程并发调用该方法时，不再具有同步效果，这种情形即是"关键字synchronized不能被继承"的转述。具体的举例见：com.bravedawn.concurrency.example.sync.SynchronizedExample4

   * synchronized修饰代码块

     **不同对象调用同步代码块是互不影响的**

     参见：com.bravedawn.concurrency.example.sync.SynchronizedExample1；com.bravedawn.concurrency.example.sync.SynchronizedExample2

     ```java
     // 修饰一个代码块：同步语句块
     public void test1(){
         synchronized (this){
         for (int i = 0; i < 10; i++){
             	log.info("test1 - {}", i);
             }
         }
     }
     ```

  * synchronized修饰方法

    **如果一个方法中放着一个完整的同步代码块（synchronized修饰代码块），他其实和同步方法（synchronized修饰方法）是一样的。**

    **不同对象调用同步代码块是互不影响的**

    参见：com.bravedawn.concurrency.example.sync.SynchronizedExample1；com.bravedawn.concurrency.example.sync.SynchronizedExample2

    ```java
    // 修饰一个方法：同步方法
    public synchronized void test2(){
        for (int i = 0; i < 10; i++){
        	log.info("test2 - {}", i);
        }
    }
    ```

  * synchronized修饰静态方法

    参见：com.bravedawn.concurrency.example.sync.SynchronizedExample3

    ```java
    public synchronized static void test2(int j){
        for (int i = 0; i < 10; i++){
        	log.info("test2 {} - {}", j, i);
        }
    }
    ```

  * synchronized修饰类

    参见：com.bravedawn.concurrency.example.sync.SynchronizedExample3

    ```java
    public void test1(int j){
        synchronized (SynchronizedExample3.class){
            // 以下就是括号括起来的问题
            for (int i = 0; i < 10; i++){
            	log.info("test1 {} - {}", j, i);
            }
        }
    }
    ```

  * 在上面的四个synchronized修饰的代码片中，如果**synchronized修饰代码块**和**synchronized修饰方法**的作用域代码相同时，他们两的功能就是相同的，可以相互替换。如果**synchronized修饰静态方法**和**synchronized修饰类**的作用域代码相同时，他们两的功能就是相同的，可以相互替换。

  * 在计数中的使用

    参见：com.bravedawn.concurrency.example.count.CountExample3

    **synchronized实现原理**：synchronized可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时它还可以保证共享变量的内存可见性

* 原子性对比

  ![8-5](E:\markdown笔记\笔记图片\8\8-5.png)

### 3.可见性

**可见性**则更为微妙，它必须确保释放锁之前对共享数据做出的更改对于随后获得该锁的另一个线程是可见的。 —— 如果没有同步机制提供的这种可见性保证，线程看到的共享变量可能是修改前的值或不一致的值，这将引发许多严重问题。

![8-6](E:\markdown笔记\笔记图片\8\8-6.png)

* 可见性——synchronized

  ![8-7](E:\markdown笔记\笔记图片\8\8-7.png)

* 可见性——volatile

  ![8-8](E:\markdown笔记\笔记图片\8\8-8.png)

* volatile读写操作加入内存屏障和禁止重排序的CPU指令示意图
    ![8-10](E:\markdown笔记\笔记图片\8\8-10.png)

    ![8-9](E:\markdown笔记\笔记图片\8\8-9.png)

  参见：com.bravedawn.concurrency.example.count.CountExample4，在这个计数的程序中：

  ```java
  public static volatile int count = 0;
  
  private static void add(){
      count++;
  }
  ```

  我们将count变量加了**volatile**关键字，但时还是**不能保证线程的原子性**。再看下面的add方法，这次这里的执行顺序主要有三步，若此时同时有两个进程：

  * 首先他们会从主内存中将count变量的值取出来
  * 然后再同时加1
  * 最后他们再同时将工作内存的值写回主内存

  完成上面的三步后，其实真正计数的值只增加了一次，也就是只有一个进程做了add操作。**所以直接使用volatile执行加操作不是线程安全的，volatile不具有原子性**。

* volatile的使用条件

  * 对变量的写操作不依赖于当前值。
  * 该变量没有包含在具有其他变量的不变式中。

  所以volatile特别适合作为状态标识量。

* 使用场景一：volatile作为状态标识量的使用方法

  ```java
  volatile boolean inited = false;
  
  // 线程1
  context = loadContext();
  inited = true;
  
  // 线程2
  while(!inited){
      sleep();
  }
  doSomethingWitConfig(context);
  ```

  上述代码必须先执行完loadContext()，才能执行doSomethingWitConfig(context)。

* 使用场景二：Double-Check。这一块参考**第五章第2节**

### 4.有序性

![8-11](E:\markdown笔记\笔记图片\8\8-11.png)

* 可以通过volatile、synchronized、Lock保证程序的有序性

* 其中synchronized、Lock保证同一时刻只有一个线程执行同步代码，相当于让线程顺序徐执行同步代码，自然就保证了有序性。

* 在java内存模型中说过，为了性能优化，编译器和处理器会进行指令重排序；也就是说java程序天然的有序性可以总结为：**如果在本线程内观察，所有的操作都是有序的；如果在一个线程观察另一个线程，所有的操作都是无序的**。

* [happens-before原则](https://www.cnblogs.com/chenssy/p/6393321.html)

  * **在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系。**

  * **如果两个操作不存在下述（下面8条 ）任一一个happens-before规则，那么这两个操作就没有顺序的保障，JVM可以对这两个操作进行重排序。如果操作A happens-before操作B，那么操作A在内存上所做的操作对操作B都是可见的。**

  * 下面是happens-before原则规则：

    * **1、**程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作

      一段代码在**单线程中执行的结果是有序的**。注意是执行结果，因为虚拟机、处理器会对指令进行重排序。虽然重排序了，但是并不会影响程序的执行结果，所以程序最终执行的结果与顺序执行的结果是一致的。**故而这个规则只对单线程有效，在多线程环境下无法保证正确性**。

    * **2、**锁定规则：一个unLock操作先行发生于后面对同一个锁额lock操作

      这个规则比较好理解，无论是在单线程环境还是多线程环境，一个锁处于被锁定状态，那么必须先执行unlock操作后面才能进行lock操作。

    * **3、**volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作

      这是一条比较重要的规则，它标志着volatile保证了线程可见性。通俗点讲就是如果一个线程先去写一个volatile变量，然后一个线程去读这个变量，那么这个写操作一定是happens-before读操作的。

    * **4、**传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C

      提现了happens-before原则具有传递性，即A happens-before B , B happens-before C，那么A happens-before C

    * 5、线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作

    * 6、线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生

    * 7、线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行

    * 8、对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始

线程安全性总结：

![](E:\markdown笔记\笔记图片\8\28.png)

## 第5章 安全发布对象

### 1.发布与逸出

> com.bravedawn.concurrency.example.publish

![](E:\markdown笔记\笔记图片\8\29.png)

参考文章：https://www.jianshu.com/p/a3fc770d11b9

* 发布对象：是一个对象能够被当前范围之外的代码所使用

  ```java
  @Slf4j
  @NotThreadSafe
  public class UnSafePublish {
  
      private String[] state = {"a", "b", "c"};
  
      public String[] getState(){
          return state;
      }
  
      public static void main(String[] args) {
          UnSafePublish unSafePublish = new UnSafePublish();
          log.info(Arrays.toString(unSafePublish.getState()));
  
          // 当前对象的私有属性可以被其它线程任意修改
          unSafePublish.getState()[0] = "d";
          log.info(Arrays.toString(unSafePublish.getState()));
      }
  }
  ```

* 对象逸出：一种错误的发布。当一个对象还没有构造完成时，就使它被其它线程所见

  ``` java
  @Slf4j
  @NotThreadSafe
  @NotRecommend
  public class Escape {
  
      private int thisCanBeEscape;
  
  
      public Escape(){
          new InnerClass();
          // 该变量还未完成初始化，this对象就已经逸出
          thisCanBeEscape = 1;
      }
  
      private class InnerClass{
  
          public InnerClass(){
              log.info("{}", Escape.this.thisCanBeEscape);
          }
      }
  
      public static void main(String[] args) {
          new Escape();
      }
  }
  ```
  
  this对象逸出，有可能在构造方法初始化的时候一些初始工作未完成而提前发布了对象从而导致对象逸出的问题。在一个可变对象未构造完成之前，不应该将其发布。
  
  应该利用**工厂方法**和**私有构造函数**来完成对象创建和监听器的创建，从而避免对象逸出。
  
* 不正确的发布可变对象可能会导致两种错误：

  * 发布线程以外的其他线程都可以看到发布对象过期的值
  * 线程看到的对象的引用是最新的，但是对象状态的值却是失效

参考：

* [JAVA - 可变对象与不可变对](https://blog.csdn.net/bupa900318/article/details/80696785)

* [Java 可变对象和不可变对象](https://my.oschina.net/jackieyeah/blog/205198?fromerr=yJ78hGIV)

### 2.安全发布对象的四种方法

> com.bravedawn.concurrency.example.singleton

* 在静态初始化函数中初始化一个对象引用

* 将对象的引用保存到volatile类型或者AtomicReference对象中

* 将对象的引用保存到某个正确构造对象的final类型域中

* 将对象的引用保存到一个有锁保护的域中

--------

* 下面介绍一下singleton单例包下的代码：

  * singleton.SingletonExample1

    **线程不安全：**懒汉模式，单例实例在第一次使用时进行创建。

  * singleton.SingletonExample2

    **线程安全：**饿汉模式，单例实例在类装载时进行创建。

    * 缺点：
    * 饿汉模式如果构造函数内的进行过多的操作，在类加载时就会很慢从而造成性能问题；
      * 如果使用饿汉模式只进行类的加载而不使用会造成资源浪费。
    * 使用：
      * 构造函数内的不应该有过多的操作
      * 这个类在实际的场景中肯定会被使用
  
  * singleton.SingletonExample3
  
    **线程安全：**懒汉模式，单例实例在第一次使用时进行创建。在静态的工厂方法上加了synchronized关键字后，虽然保证了线程安全性，但是却带了性能上面的开销。
  
  * singleton.SingletonExample4
  
    **线程不安全：**懒汉模式 --> 双重同步锁单例模式，单例实例在第一次使用时进行创建。
  
    ```java
    // 静态的工厂方法
    public static SingletonExample4 getInstance(){
        if (instance == null){ // 双重检测机制
            synchronized (SingletonExample4.class){ // 同步锁，将对象的引用保存到一个有锁保护的域中
                if (instance == null){
                	instance = new SingletonExample4();
              }
            }
      }
        return instance;
    }
    ```
  
    在单线程情况下，执行instance = new SingletonExample4()，指令重排对单线程是没有影响的;
  
    * 1.memory = allocate() 分配对象的内存空间
  
    * 2.ctorInstance() 初始化对象
    * 3.instance = memory 设置instance指向刚分配的内存空间
  
    由于JVM和cpu优化，发生了指令重排，导致上述三步执行发生变化：
  
    * 1.memory = allocate() 分配对象的内存空间
  
    * 2.instance = memory 设置instance指向刚分配的
    * 3.ctorInstance() 初始化对象
  
    线程不安全的情况，假设现在有两个线程A，B：
  
    1. 线程A在执行`instance = new SingletonExample4();`时，发生指令重排，执行到`2.instance = memory 设置instance指向刚分配的`，线程B此时执行到第一个`if (instance == null){`时，认为instance已经有值了，所以直接执行了`return instance;`
    2. 但是A进程还没有执行`3.ctorInstance() 初始化对象`。
    3. 线程B一旦拿到还没有初始化的instance对象就会出错，所以不是线程安全的
  
  * singleton.SingletonExample5
  
    **线程安全：**懒汉模式 --> 双重同步锁单例模式，单例实例在第一次使用时进行创建。
  
    ```java
    // 单例对象
  // volatile + 双重检测机制 --> 禁止指令重排
    // 在实例对象前面加上volatile关键字就可以避免SingletonExample4中出现的问题了
  private volatile static SingletonExample5 instance = null;
    ```
  
  * singleton.SingletonExample6
  
    **线程安全：**饿汉模式的第二种写法
  
    ```java
    @ThreadSafe
    public class SingletonExample6 {
        // 构造函数
        private SingletonExample6(){
    
        }
    
        // 这里需要注意的就是一定要把静态域放在静态代码块前面，先执行静态域在执行静态代码块
    
        // 单例对象
        // 静态域
        private static SingletonExample6 instance = null;
    
        // 静态代码块
        static {
            instance = new SingletonExample6();
        }
    
        // 静态的工厂方法
        public static SingletonExample6 getInstance(){
            return instance;
        }
    
      public static void main(String[] args) {
            System.out.println(getInstance().hashCode());
          System.out.println(getInstance().hashCode());
        }
}
    ```
  
  * singleton.SingletonExample7
  
    **线程安全、推荐：**枚举模式：最安全
  
    这个代码长见识niuniuniu！
  
    ```java
    @ThreadSafe
    @Recommend
    public class SingletonExample7 {
    
        private SingletonExample7(){
    
        }
    
        public static SingletonExample7 getInstance(){
            return Singleton.INSTANCE.getSingleton();
        }
    
        private enum Singleton{
            INSTANCE;
            private SingletonExample7 singleton;
    
            // JVM保证这个方法绝对只调用一次
            Singleton(){
                singleton = new SingletonExample7();
            }
    
            public SingletonExample7 getSingleton(){
                return singleton;
            }
        }
    }
    ```


## 第6章 线程安全策略

### 1.不可变对象

![8-12](E:\markdown笔记\笔记图片\8\8-12.png)

* 创建不可变对象

  这里可以采用的方式有：

  * 将类声明为final，不允许继承
  * 将类所有的成员变量声明为私有的，这样就不能直接访问类成员
  * 对变量不提供set方法，将所有可变的成员声明为final，这样只能对他们赋值一次
  * 在构造方法中所有变量进行赋值，进行深度拷贝
  * 在get方法中不直接返回变量本身而是克隆对象，并返回对象的拷贝。

  对于要创建不可变对象，我们应多参考String和final类

* final关键字

  ![](E:\markdown笔记\笔记图片\8\30.png)

  **1. 数据**

  声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

  - 对于基本类型，final 使数值不变；
  - 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。
  - 代码示例可以参见：com.bravedawn.concurrency.example.immutable.ImmutableExample1

  ```
  final int x = 1;
  // x = 2;  // cannot assign value to final variable 'x'
  final A y = new A();
  y.a = 1;
  ```

  **2. 方法** 

  声明方法不能被子类覆盖。

  private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是覆盖基类方法，而是在子类中定义了一个新的方法。

  **3. 类**

  声明类不允许被继承。

  **4.方法的参数**

  使用final修饰的参数也不能被修改

  ```java
  private void test(final int c){
      // 使用final修饰的参数也不能被修改
      //c = 1;
  }
  ```

* 将对象修改为不可变对象

  ![8-13](E:\markdown笔记\笔记图片\8\8-13.png)

  * Java提供的Collections类，值得注意的是Collections修改的对象不能是final的。使用参见com.bravedawn.concurrency.example.immutable.ImmutableExample2
  * Guava提供的ImmutableXXX，值得注意的是Collections修改的对象可以是final的。使用参见com.bravedawn.concurrency.example.immutable.ImmutableExample3

### 2.线程封闭

避免并发除了将对象声明为不可变对象之外，还有一个简单的方法就是线程封闭。

为了确保线程的安全，通常需要保证可变的共享数据的同步访问，具体采用的方式有很多；但还有一种方法可以保证线程的安全，即是使可变数据不共享，或者是使数据不可变。**所谓“线程封闭”即是仅在单线程中访问数据，也就是通过让可变数据不被多个线程共享以确保数据的正确性。**

![8-14](E:\markdown笔记\笔记图片\8\8-14.png)

* 堆栈封闭：

  多个线程访问一个方法的时候，方法的局部变量都会拷贝一份到线程的栈中。所以局部变量不会被多个线程所共享的。因此就不会出现并发问题，所以能局部变量的时候就不用全局变量，全局变量会引发并发问题。

  这也就是我们写的程序基本上都在一个方法里面定义局部变量来完成的，所以不会有并发问题。

* ThreadLocal（详情参见：com.bravedawn.concurrency.example.threadlocal）

  * 使用TreadLocal的相关场景：

    在我们进行**web开发的时候**，用户的每一个请求就是一个线程。一般的话我们获取用户的信息，需要在controller层通过request进行获取，然后传到service层或者是util里，这样传递数据是很费劲麻烦的。而我们使用TreadLocal之后，我们在请求已经到了后端服务器，在进行处理之前调用RequestHolder.add()将相关信息保存到ThreadLocal中，之后我们随用随取，当请求结束后，我们在调用RequestHolder.remove()。

  * 具体实现

    * ThreadLocal处理类：com.bravedawn.concurrency.example.threadlocal.RequestHolder
    
    * Filter类：com.bravedawn.concurrency.HttpFilter
    
    * Interceptor类：com.bravedawn.concurrency.HttpInterceptor
    
    * Controller类：com.bravedawn.concurrency.example.threadlocal.ThreadlocalController
    
    * 配置类，用于配置Filter和Interceptor：com.bravedawn.concurrency.ConcurrencyApplication
    
      * Spring Boot Filter的写法
    
        1. 编写com.bravedawn.concurrency.HttpFilter
    
        2. 在Application中进行配置
    
           ```java
           @Bean
           public FilterRegistrationBean httpFilter(){
               FilterRegistrationBean registrationBean = new FilterRegistrationBean();
               registrationBean.setFilter(new HttpFilter());
               registrationBean.addUrlPatterns("/threadlocal/*");
               return registrationBean;
           }
           ```
    
      * Spring Boot Interceptor的写法
    
        1. 编写com.bravedawn.concurrency.HttpInterceptor
    
        2. 首先让Application类继承`WebMvcConfigurationSupport`
    
        3. 在Application中进行配置
    
           ```java
           @Override
           protected void addInterceptors(InterceptorRegistry registry) {
           	registry.addInterceptor(new HttpInterceptor()).addPathPatterns("/**");
           }
           ```
    
  * **ThreadLocal对象通常用于防止可变的单实例变量或全局变量进行共享**。例如：在单线程应用程序中可能会维持一个全局的数据库连接，并在程序启动时初始化这个连接对象，从而避免在调用每个方法时都要传递一个Connection对象。由于JDBC的连接对象不一定是线程安全的，因此，当多线程程序在没有协同的情况下使用全局变量时，就不是线程安全的。通过将JDBC的连接保存到ThreadLocal对象中，每个线程都会拥有属于自己的连接，如下为代码示例。
  
    ```java
    private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<Connection>(){
        //初始化ThreadLocal对象connectionHolder的共享变量为Connection类型的对象
        public Connection initialValue(){
            return DriverManager.getConnection(DB_URL);  
        }
    };
     
    public static Connection getConnection(){
        //使用ThreadLocal对象的get()方法获取其共享变量
        return connectionHolder.get();
    }
    ```
    
  * ThreadLocal的作用
  
    ThreadLocal是解决线程安全问题一个很好的思路，它**通过为每个线程提供一个独立的变量副本解决了变量并发访问的冲突问题**。在很多情况下，ThreadLocal比直接使用synchronized同步机制解决线程安全问题更简单，更方便，且结果程序拥有更高的并发性。
  
  * ThreadLocal的使用场景
  
    在Java的多线程编程中，为保证多个线程对共享变量的安全访问，通常会使用synchronized来保证同一时刻只有一个线程对共享变量进行操作。这种情况下可以将[类变量](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E7%B1%BB%E5%8F%98%E9%87%8F)放到ThreadLocal类型的对象中，使变量在每个线程中都有独立拷贝，不会出现一个线程读取变量时而被另一个线程修改的现象。最常见的ThreadLocal使用场景为用来解决数据库连接、Session管理等。

### 3.线程不安全的类与写法

* 什么是线程不安全的类

  如果一个类的对象同时被多个线程访问，如果不做特殊的同步或并发处理，很容易表现出线程不安全的现象，比如抛出异常、逻辑处理错误等，这种类我们就称为线程不安全的类。

* StringBuilder和StringBuffer

  * StringBuilder，线程不安全的类

    参见：com.bravedawn.concurrency.example.commonUnsafe.StringExample1

  * StringBuffer，线程安全的类

  * 参见：com.bravedawn.concurrency.example.commonUnsafe.StringExample2

  * 为什么要提供线程不安全的类StringBuilder?

    StringBuffer是线程安全的类。其中他的方法中添加了synchroinzed关键字，保证同一时间只有一个线程才能访问，是有性能损耗的。所以我们可以**在方法中使用StringBuilder**，因为方法中的**局部变量**是堆栈封闭的。

* SimpleDataFormat和JodaTime

  * SimpleDataFormat线程不安全的写法，参见：com.bravedawn.concurrency.example.commonUnsafe.DataFormatExample1
  * SimpleDataFormat线程安全的写法，在方法中每次声明一个SimpleDataFormat对象，防止线程不安全。参见：com.bravedawn.concurrency.example.commonUnsafe.DataFormatExample2
  * JodaTime线程安全的写法，参见：com.bravedawn.concurrency.example.commonUnsafe.DateFormatExample3

* ArrayList，HashSet，HashMap等Collections

  * ArrayList是线程不安全的，具体写法参见：com.bravedawn.concurrency.example.commonUnsafe.ArrayListExample
  * HashSet在多线程并发条件下是线程不安全的，具体写法参加：com.bravedawn.concurrency.example.commonUnsafe.HashSetExample
  * HashMap是线程不安全的，参见：com.bravedawn.concurrency.example.commonUnsafe.HashMapExample

  后面会有专门的章节来讲解他们对应线程安全的类。

* 先检查再执行的写法是线程不安全的：`if(condition(a)){handle(a);}`

   `if（condition（a））{handle（a）；} `就算a是一个线程安全的类所对应的对象，对a的处理handle（a）也是原子性的，但由于**这两步之间的不是原子性的**也会引发线程安全问题，如A、B两个线程都通过了a的判断条件，A线程执行handle（a）之后，a已经不符合condition（a）的判断条件了，可是此时B线程仍然要执行handle（a），这就引发了线程安全问题。
   
   之前的Atomic自增在底层实现时是通过CAS原理来保证原子性的更新；
   
   在实际项目中，如果要判断某个对象是否满足某个条件，满足某个条件再做某个处理的时候。要考虑：
   
   1. 这个对象是不是多线程共享的
   2. 如果这个对象是多线程共享的，一定要在这个对象上加一个锁，保证检查和执行两个操作时原子性的。否则会触发线程不去安全的问题

### 4.同步容器

![](E:\markdown笔记\笔记图片\8\31.png)

* ArrayList --> Vector，Stack

  * Vector 是线程安全的，具体参见：com.bravedawn.concurrency.example.syncContainer.VectorExample

  * Vector也有线程不安全的时候，具体参见：com.bravedawn.concurrency.example.syncContainer.VectorExample2。在这个方法中，我们启用了两个线程，一个线程做remove操作，一个线程做get操作。当我们运行代码的时候，就会发现get操作老是报错，显示get越界。假设这样的场景：

    ```java
    Thread thread1 = new Thread(){
        @Override
        public void run() {
            for (int i = 0; i < vector.size(); i++){  // <--i=9
            	vector.remove(i);
            }
        }
    };
    ```

    ```java
    Thread thread2 = new Thread(){
        @Override
        public void run() {
            for (int i = 0; i < vector.size(); i++){  // <--i=
                vector.get(i);
            }
        }
    };
    ```

    上面两段代码当i=9时，线程一先执行，导致第九个元素被移除。然后线程二进行get操作，发生了数据越界。

* HashMap --> HashTable(key, value不能为null)

  HashTable是线程安全的，具体参见：com.bravedawn.concurrency.example.syncContainer.HashTableExample

* Collections.synchronizedXXX(List, Set, Map)

  * Collections.synchronizedList()是线程安全的，参见：com.bravedawn.concurrency.example.syncContainer.CollectionsExample1
  * Collections.synchronizedSet()是线程安全的，参见：com.bravedawn.concurrency.example.syncContainer.CollectionsExample2
  * Collections.synchronizedMap()是线程安全的，参见：com.bravedawn.concurrency.example.syncContainer.CollectionsExample3

* vector类在单线程遍历情况下进行增删操作会造成线程不安全的情况，多线程并发情况下更是如此。详情参见：com.bravedawn.concurrency.example.syncContainer.VectorExample3。

  所以建议是：

  1. 在对集合进行`foreach`循环和`迭代器`循环时，尽量不要再遍历的过程中进行`remove`操作；
  2. 如果要对集合进行删除操作，建议是在集合遍历时对相关的值进行标记，等集合遍历结束之后再进行删除操作；

### 5.并发容器及安全共享策略总结

![](E:\markdown笔记\笔记图片\8\32.png)

1. ArrayList --> CopyOnWriteArrayList

   * 介绍

     CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

   * 缺点

     1. **内存占用问题**。因为CopyOnWrite的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象（注意:在复制的时候只是复制容器里的引用，只是在写的时候会创建新对象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。

        针对内存占用问题，可以通过压缩容器中的元素的方法来减少大对象的内存消耗，比如，如果元素全是10进制的数字，可以考虑把它压缩成36进制或64进制。或者不使用CopyOnWrite容器，而使用其他的并发容器，如ConcurrentHashMap。

     2. **数据一致性问题**。不能用于实时读的场景，CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。**适合读多写少的操作。**

   * 优点

     1. 线程安全
     2. 适合读多写少的场景，适合size较小，只读操作远大于可变操作

   * 设计思想

     1. 读写分离
     2. 最终一致性
     3. 使用时另外开辟空间，从而解决并发冲突

   * 具体实现

     * add操作：java.util.concurrent.CopyOnWriteArrayList#add(E)
     * get操作：java.util.concurrent.CopyOnWriteArrayList#get(int)
     
   * 代码调用：com.bravedawn.concurrency.example.concurent.CopyOnWriteArrayListExample
   
2. HashSet、TreeSet -> CopyOnWriteArraySet、ConcurrentSkipListSet

   1. CopyOnWriteArraySet

      * 介绍

        它是线程安全的无序的集合，可以将它理解成线程安全的[HashSet](http://www.cnblogs.com/skywang12345/p/3311252.html)。有意思的是，CopyOnWriteArraySet和HashSet虽然都继承于共同的父类AbstractSet；但是，HashSet是通过[“散列表](http://www.cnblogs.com/skywang12345/p/3310835.html)(HashMap)”实现的，而CopyOnWriteArraySet则是通过“[动态数组(CopyOnWriteArrayList)](http://www.cnblogs.com/skywang12345/p/3498483.html)”实现的，并不是散列表。
        和CopyOnWriteArrayList类似，**其实CopyOnWriteSet底层包含一个CopyOnWriteList**，几乎所有操作都是借助**CopyOnWriteList，就像HashSet包含HashMap**

      * 特性：
      
        1. 它最适合于具有以下特征的应用程序：Set 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。
        2. 它是线程安全的。
        3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
        4. 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等 操作。
        5. 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。
      
      * 代码调用：com.bravedawn.concurrency.example.concurent.CopyOnWriteArraySetExample
      
   2. ConcurrentSkipListSet

      * 介绍

        ConcurrentSkipListSet是线程安全的有序的集合，适用于高并发的场景。
        ConcurrentSkipListSet和[TreeSet](http://www.cnblogs.com/skywang12345/p/3311268.html)，它们虽然都是有序的集合。但是，第一，它们的线程安全机制不同，TreeSet是非线程安全的，而ConcurrentSkipListSet是线程安全的。第二，ConcurrentSkipListSet是通过[ConcurrentSkipListMap](http://www.cnblogs.com/skywang12345/p/3498556.html)实现的，而TreeSet是通过TreeMap实现的。

      * 特点

        1. 线程安全
        2. 单独的add、remove等操作是线程安全的。addAll、removeAll等操作需要自己加锁

      * 代码调用：com.bravedawn.concurrency.example.concurent.ConcurrentSkipListSetExample
   
3. HashMap、TreeMap -> ConcurrentHashMap、ConcurrentSkipListMap

   1. ConcurrentHashMap
      * 代码调用：com.bravedawn.concurrency.example.concurent.ConcurrentHashMapExample
   2. ConcurrentSkipListMap
      * 代码调用：com.bravedawn.concurrency.example.concurent.ConcurrentSkipListMapExample

![](E:\markdown笔记\笔记图片\8\33.png)

![34](E:\markdown笔记\笔记图片\8\34.png)

![35](E:\markdown笔记\笔记图片\8\35.png)

## 第7章 J.U.C之AQS

### 7-1 J.U.C之AQS-介绍

![](E:\markdown笔记\笔记图片\8\36.png)

* AQS的设计
  * 使用Node实现FIFO队列，可以用于构建锁或者其他的同步装置的基础框架
  * 利用一个int类型表示状态
  * 使用方法是继承
  * 子类通过继承并通过实现它的方法管理其状态（acquire和release）的方法操作状态
  * 可以同时实现排他锁和共享锁模式（独占、共享）
* AQS的组件
  * CountDownLatch
  * Semaphore
  * CyclicBarrier
  * ReentrantLock
  * Condition
  * FutureTask

### 7-2 J.U.C之AQS-CountDownLatch

1. 概念

   这个图应该从上往下看。

   ![](E:\markdown笔记\笔记图片\8\37.png)

   countDownLatch这个类使一个线程等待其他线程各自执行完毕后再执行。

   是通过一个计数器来实现的，计数器的初始值是线程的数量。每当一个线程执行完毕后，计数器的值就-1，当计数器的值为0时，表示所有线程都执行完毕，然后在闭锁上等待的线程就可以恢复工作了。

2. 实践

   1. 使用CountDownLatch计数：com.bravedawn.concurrency.example.aqs.CountDownLatchExample1
   2. 使用CountDownLatch计数，若在有限的时间内未完成任务，则不再等待，直接结束；或者是有得时候，忘记调用`countDown()`方法时可以设置该值：com.bravedawn.concurrency.example.aqs.CountDownLatchExample2

### 7-3 J.U.C之AQS-Semaphore（信号量）

![](E:\markdown笔记\笔记图片\8\38.png)

1. 概念

   Semaphore也是一个线程同步的辅助类，可以维护当前访问自身的线程个数，并提供了同步机制。使用Semaphore可以控制同时访问资源的线程个数。例如，实现一个文件允许的并发访问数。

2. 方法摘要

  * void acquire()：从此信号量获取一个许可，在提供一个许可前一直将线程阻塞，否则线程被中断。
  * void release()：释放一个许可，将其返回给信号量。
  * int availablePermits()：返回此信号量中当前可用的许可数。
  * boolean hasQueuedThreads()：查询是否有线程正在等待获取。
3. 实践

   1. 一次获取一个许可：com.bravedawn.concurrency.example.aqs.SemaphoreExample1

   2. 一次获取多个许可，执行一次需要获得多个许可才能接着进行：com.bravedawn.concurrency.example.aqs.SemaphoreExample2

   3. 尝试获取一个许可，其他的丢弃：com.bravedawn.concurrency.example.aqs.SemaphoreExample3

   4. 在一定时间内，尝试获取一个许可，其他的丢弃：com.bravedawn.concurrency.example.aqs.SemaphoreExample4

### 7-4 J.U.C之AQS-CyclicBarrier

下面这个图可以应该从下往上看。

![](E:\markdown笔记\笔记图片\8\39.png)

1. 概念

   假设有一个场景，每个线程代表一个跑步的运动员，当运动员都准备好之后，才一起出发，只要有一个运动员还没有准备好，所有线程就一起等待。

   CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。

2. CyclicBarrier 使用场景

   CyclicBarrier可以用于多线程计算数据，最后合并计算结果的应用场景。比如我们用一个Excel保存了用户所有银行流水，每个Sheet保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，都执行完之后，得到每个sheet的日均银行流水，最后，再用barrierAction用这些线程的计算结果，计算出整个Excel的日均银行流水。

3. CountDownLatch和CyclicBarrier区别

   * CountDownLatch简单的说就是一个线程等待，直到他所等待的其他线程都执行完成并且调用countDown()方法发出通知后，当前线程才可以继续执行。
   * cyclicBarrier是所有线程都进行等待，直到所有线程都准备好进入await()方法之后，所有线程同时开始执行。
   * CountDownLatch的计数器只能使用一次。而CyclicBarrier的计数器可以使用reset() 方法重置。所以CyclicBarrier能处理更为复杂的业务场景，比如如果计算发生错误，可以重置计数器，并让线程们重新执行一次。
   * CyclicBarrier还提供其他有用的方法，比如getNumberWaiting方法可以获得CyclicBarrier阻塞的线程数量。isBroken方法用来知道阻塞的线程是否被中断。如果被中断返回true，否则返回false。

4. 实践

   * CyclicBarrier线程等待策略：com.bravedawn.concurrency.example.aqs.CyclicBarrierExample1
   * await()超时报错：com.bravedawn.concurrency.example.aqs.CyclicBarrierExample2
   * 第二个构造方法有一个 Runnable 参数，这个参数的意思是最后一个到达线程要做的任务，他会抢先执行：com.bravedawn.concurrency.example.aqs.CyclicBarrierExample3

5. 博文
  
   * https://www.jianshu.com/p/333fd8faa56e


### 7-5 J.U.C之AQS-ReentrantLock与锁-1

Java中提供的两种锁，**synchronized**和**ReentrantLock**。

1. **ReentrantLock（可重入锁）和synchronized区别**

   * **可重入性**

     从名字上理解，ReenTrantLock的字面意思就是再进入的锁，其实synchronized关键字所使用的锁也是可重入的，两者关于这个的区别不大。两者都是同一个线程每进入一次，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。

   * **锁的实现**

     Synchronized是依赖于JVM实现的，而ReenTrantLock是JDK实现的，有什么区别，说白了就类似于[操作系统](http://lib.csdn.net/base/operatingsystem)来控制实现和用户自己敲代码实现的区别。前者的实现是比较难见到的，后者有直接的源码可供阅读。

   * **性能的区别**

     在Synchronized优化以前，synchronized的性能是比ReenTrantLock差很多的，但是自从Synchronized引入了偏向锁，轻量级锁（自旋锁）后，两者的性能就差不多了，在两种方法都可用的情况下，官方甚至建议使用synchronized，其实synchronized的优化我感觉就借鉴了ReenTrantLock中的CAS技术。都是试图在用户态就把加锁问题解决，避免进入内核态的线程阻塞。

   * **功能区别**

     1. **便利性**：很明显Synchronized的使用比较方便简洁，并且由编译器去保证锁的加锁和释放，而ReenTrantLock需要手工声明来加锁和释放锁，为了避免忘记手工释放锁造成死锁，所以最好在finally中声明释放锁。
     2. **锁的细粒度和灵活度**：很明显ReenTrantLock优于Synchronized

2. **ReenTrantLock独有的能力**

   1. ReenTrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。

   2. ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。

   3. ReenTrantLock提供了一种能够中断等待锁的线程的机制，通过**lock.lockInterruptibly()**来实现这个机制。

3. **ReenTrantLock实现的原理**

   在网上看到相关的源码分析，本来这块应该是本文的核心，但是感觉比较复杂就不一一详解了，简单来说，ReenTrantLock的实现是一种自旋锁，通过循环调用CAS操作来实现加锁。它的性能比较好也是因为避免了使线程进入内核态的阻塞状态。**想尽办法避免线程进入内核的阻塞状态是我们去分析和理解锁设计的关键钥匙。**

4. **什么情况下使用ReenTrantLock**

   答案是，如果你需要实现ReenTrantLock的三个独有功能时。

5. **是否要放弃synchronized**

   synchronized能做的，ReentrantLock都能做；而ReentrantLock能做的，而synchronized却不一定做得了。性能方面，ReentrantLock不比synchronized差，那么要放弃使用synchronized？

   * J.U.C包中的锁定类是用于高级情况和高级用户的工具，除非说你对Lock的高级特性有特别清楚的了解以及有明确的需要，或这有明确的证据表明同步已经成为可伸缩性的瓶颈的时候，否则我们还是继续使用synchronized
   * 相比较这些高级的锁定类，synchronized还是有一些优势的，比如synchronized不可能忘记释放锁。 在退出synchronized块时，JVM会自动释放锁，会很容易忘记要使用finally释放锁，这对程序非常有害。
   * 还有当JVM使用synchronized管理锁定请求和释放时，JVM在生成线程转储时能够包括锁定信息，这些信息对调试非常有价值，它们可以标识死锁以及其他异常行为的来源。 而Lock类只是普通的类，JVM不知道哪个线程具有Lock对象，而且几乎每个开发人员都是比较熟悉synchronized。

6. 实践

   1. ReenTrantLock的简单使用：com.bravedawn.concurrency.example.lock.LockExample2

### 7-6 J.U.C之AQS-ReentrantLock与锁-2

1. 读写锁-ReentrantReadWriteLock

   1. 简介

      对共享资源有读和写的操作，且写操作没有读操作那么频繁。在没有写操作的时候，多个线程同时读一个资源没有任何问题，所以应该允许多个线程同时读取共享资源；但是如果一个线程想去写这些共享资源，就不应该允许其他线程对该资源进行读和写的操作了。

      **JAVA的并发包提供了读写锁ReentrantReadWriteLock，它表示两个锁，一个是读操作相关的锁，称为共享锁；一个是写相关的锁，称为排他锁**

      **ReentrantReadWriteLock是悲观读取。**

   2. 读写锁条件

      * 线程进入读锁的前提条件：

        没有其他线程的写锁，

        没有写请求或者**有写请求，但调用线程和持有锁的线程是同一个。**

      * 线程进入写锁的前提条件：

        没有其他线程的读锁

        没有其他线程的写锁

   3. 读写锁有以下三个重要的特性：

      （1）公平选择性：支持非公平（默认）和公平的锁获取方式，吞吐量还是非公平优于公平。

      （2）重进入：读锁和写锁都支持线程重进入。

      （3）锁降级：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级成为读锁。

   4. 实践

      1. 实现一个简单的并发Map：com.bravedawn.concurrency.example.lock.LockExample3

2. StampedLock

   1. 参考博客：

      https://www.cnblogs.com/fanhaiping/p/9577486.html

      https://segmentfault.com/a/1190000015808032?utm_source=tag-newest

   2. 实践

      1. JDk官网的一个例子：com.bravedawn.concurrency.example.lock.LockExample4
      2. 线程安全的计数：com.bravedawn.concurrency.example.lock.LockExample

3. 两条锁的使用建议

   * 当只有少量竞争者的时候，**synchronized**是一个很好的通用的锁实现，JVM会控制死锁的情况。
   * 竞争者不少，但是线程增长的趋势可以预估。**ReentrantLock**是一个很好的通用的锁实现，死锁情况需自己控制。

4. Condition的使用

  1. Condition接口常用方法

        Condition可以通俗的理解为条件队列。当一个线程在调用了await方法以后，直到线程等待的某个条件为真的时候才会被唤醒。这种方式为线程提供了更加简单的等待/通知模式。Condition必须要配合锁一起使用，因为对共享状态变量的访问发生在多线程环境下。一个Condition的实例必须与一个Lock绑定，因此Condition一般都是作为Lock的内部实现。
  
     Condition是通过AQS等待队列和Condition的等待队列实现多线程之间交互的工具类。
     
     * **await()** ：造成当前线程在接到信号或被中断之前一直处于等待状态。
     * **signalAll()** ：唤醒所有等待线程。能够从等待方法返回的线程必须获得与Condition相关的锁。
     
  2. 实践
  
        * Condition类的使用：com.bravedawn.concurrency.example.lock.LockExample6
  
## 第8章 J.U.C组件拓展

   ### 8-1 J.U.C-FutureTask-1

#### 1.Java中三种创建线程类的方式

1. **通过继承Thread类创建线程类**

    通过继承Thread类来创建并启动多线程的步骤如下：

   1、定义一个类继承Thread类，并重写Thread类的run()方法，run()方法的方法体就是线程要完成的任务，因此把run()称为线程的执行体；

   2、创建该类的实例对象，即创建了线程对象；

   3、调用线程对象的start()方法来启动线程；、

2. **通过实现Runnable接口创建线程类**

   这种方式创建并启动多线程的步骤如下：

   1、定义一个类实现Runnable接口；

   2、创建该类的实例对象obj；

   3、将obj作为构造器参数传入Thread类实例对象，这个对象才是真正的线程对象；

   4、调用线程对象的start()方法启动该线程；
   
3. 上面这两种方式的缺陷：**在执行完任务之后，无法获取执行结果**。可以**通过Callable和Future实现或是通过FutureTask类实现**。

#### 2.Callable与Runnable接口对比

1. Runnable需要在内部使用try-catch处理异常，callable在方法上抛出异常给上一级调用者处理。
2. Runnable没有返回值，Callable有返回值，泛型指定返回值类型，call方法类型由泛型指定。
3. 需要使用线程池来启动Callable实现类线程。

#### 3.Future接口

ExecutorService接口的submit()方法和invokeAll()方法返回一个Future对象或者Future对象的集合，**从Future中可以获取到任务执行的结果或者获取到任务执行的状态(任务是运行中还是执行完成)**。

Future接口提供了一个可能阻塞的get()方法,返回Callable任务的返回值,如果是Runnable任务,将返回null。当任务还没有返回结果之前，调用get()方法将会导致方法被阻塞，直到任务返回结果。

#### 4.FutureTask类

FutureTask实现了Runnable和Future的接口，它既可以作为Runnable被线程执行，又可以作为Future得到Callable的返回值。**推荐使用。**

### 8-2 J.U.C-FutureTask-2

1. Future接口的实践：com.bravedawn.concurrency.example.aqs.FutureExample
2. FutureTask类的实践：com.bravedawn.concurrency.example.aqs.FutureTaskExample

### 8-3 J.U.C-ForkJoin

1. 简介

   ForkJoin框架，它可以将一个大的任务拆分成多个子任务进行并行处理，最后将子任务结果合并成最后的计算结果。类似于MapReduce的思想。

2. 工作窃取算法

   ![](E:\markdown笔记\笔记图片\8\40.png)

   针对多个线程分配相同多子任务的时候，会出现不同线程完成所有任务的时间有快有慢的情况，分支/合并框架工程使用了工作窃取的技术来解决这个问题。在实际应用中，这些子任务被差不多的分配到ForkJoinSumCalculator中的所有线程上，每个线程都为分配给他的任务保存一个双向的链式队列，每完成一个任务，就会队列头上取出下一个任务开始执行。因为上面所述的原因，有些线程可能早早地完成了分配给他的任务，也就是他的队列已经空了，但其他的线程还是很忙。这个时候队列已经空了的线程并不会闲置下来，而是随机选择一个其他的线程从队列的尾巴上“偷走”一个任务。这个过程会一直继续下去，知道所有的任务都执行完毕，所有的队列都清空。这就是为什么要划成许多小任务而不是少数几个大任务的原因，他能有助于工作线程之间的平衡负载。

3. **工作窃取算法的优点：**

   * 充分利用线程进行并行计算，并减少了线程间的竞争。

4. **工作窃取算法的缺点：**

   * 在某些情况下还是存在竞争，比如双端队列里只有一个任务时。
   * 并且该算法会消耗更多的系统资源，比如创建多个线程和多个双端队列。

5. Fork/Join框架局限性：

   对于Fork/Join框架而言，当一个任务正在等待它使用Join操作创建的子任务结束时，执行这个任务的工作线程查找其他未被执行的任务，并开始执行这些未被执行的任务，通过这种方式，线程充分利用它们的运行时间来提高应用程序的性能。为了实现这个目标，Fork/Join框架执行的任务有一些局限性。

   * 任务只能使用Fork和Join操作来进行同步机制，如果使用了其他同步机制，则在同步操作时，工作线程就不能执行其他任务了。比如，在Fork/Join框架中，使任务进行了睡眠，那么，在睡眠期间内，正在执行这个任务的工作线程将不会执行其他任务了。
   * 在Fork/Join框架中，所拆分的任务不应该去执行IO操作，比如：读写数据文件。
   * 任务不能抛出检查异常，必须通过必要的代码来处理这些异常。
   
6. Fork/Join框架的核心类

  ForkJoinPool和ForkJoinTask是支持Fork/Join机制的核心类。

  ForkjoinPool是实现了ExecutorService和work-stealing(工作窃取)算法。管理工作线程和提供关于任务的状态和执行信息。

  ForkJoinTask主要是任务中执行Fork和join的机制。
  
7. Fork/Join框架的实践，计算1-100的加法：com.bravedawn.concurrency.example.aqs.ForkJoinTaskExample
  
### 8-4 J.U.C-BlockingQueue

1. 简介

   阻塞队列（BlockingQueue）被广泛使用在“**生产者-消费者**”问题中，其原因是 BlockingQueue 提供了可阻塞的插入和移除的方法。当队列容器已满，生产者线程会被阻塞，直到队列未满；当队列容器为空时，消费者线程会被阻塞，直至队列非空时为止。

2. 阻塞队列一共提供了四套方法，如下图所示：

   ![](E:\markdown笔记\笔记图片\8\41.png)

   * 竖轴
     * Insert：插入
     * Remove：移除
     * Examine：检查
   * 横轴
     * Throws Exception：如果不能马上进行就抛出异常
     * Special Value：如果不能马上进行，就返回一个特殊的值，一般是true和false
     * Blocks：如果操作不能马上进行，操作就会被阻塞
     * Times Out：如果不能马上进行，则操作就会被阻塞一段时间，如果超时后，一般会返回true和false

3. 实现类

   在Java中，BlockingQueue是一个接口，它的实现类有ArrayBlockingQueue、DelayQueue、 LinkedBlockingDeque、LinkedBlockingQueue、PriorityBlockingQueue、SynchronousQueue等，它们的区别主要体现在存储结构上或对元素操作上的不同，但是对于take与put操作的原理，却是类似的。

   * **ArrayBlockingQueue**：有界的阻塞队列，内部实现是数组，以先进先出的形式存储数据

   * **DelayQueue**：阻塞的是内部元素，元素需要实现Delayed接口，队列元素需要实现getDelay(TimeUnit unit)方法和compareTo(Delayed o)方法, getDelay定义了剩余到期时间，compareTo方法定义了元素排序规则

   * **LinkedBlockingQueue**：是一个基于链表实现的可选容量的阻塞队列。队头的元素是插入时间最长的，队尾的元素是最新插入的。新的元素将会被插入到队列的尾部。 

     LinkedBlockingQueue的容量限制是可选的，如果在初始化时没有指定容量，那么默认使用int的最大值作为队列容量。

   * **PriorityBlockingQueue**：是一个支持优先级的无界阻塞队列，直到系统资源耗尽。默认情况下元素采用自然顺序升序排列。也可以自定义类实现compareTo()方法来指定元素排序规则，或者初始化PriorityBlockingQueue时，指定构造参数Comparator来对元素进行排序。但需要注意的是不能保证同优先级元素的顺序。PriorityBlockingQueue也是基于最小二叉堆实现，使用基于CAS实现的自旋锁来控制队列的动态扩容，保证了扩容操作不会阻塞take操作的执行。

   * **SynchronousQueue**：是一个内部只能包含一个元素的队列。插入元素到队列的线程被阻塞，直到另一个线程从队列中获取了队列中存储的元素。同样，如果线程尝试获取元素并且当前不存在任何元素，则该线程将被阻塞，直到线程将元素插入队列。

4. 参考博文：https://www.jianshu.com/p/7b2f1fa616c6

## 第9章 线程调度-线程池

### 9-1 线程池-1

参考博文：https://www.jianshu.com/p/c41e942bcd64

#### 1.new Thread弊端

* 每次new Thread新建对象，性能差
* 线程缺乏统一管理，可能无限制的新建线程，相互竞争，有坑内占用过多系统资源导致司机或OOM
* 缺少更多功能，比如更多执行、定期执行、线程中断

#### 2.线程池的好处

* 重用存在的线程，减少对象创建、消亡的开销，性能佳
* 可有效控制最大并发线程数，提高系统资源利用率，同时可以避免过多资源竞争，避免阻塞
* 提供定时执行、定期执行、单线程、并发数控制等功能

#### 3.线程池-ThreadPoolExecutor

* ThreadPoolExecutor是线程池中最核心的一个类

* ThreadPoolExecutor构造方法核心参数

  * **corePoolSize**： 线程池核心线程数
  * **maximumPoolSize**：线程池最大线程数
  * **workQueue**： 线程池所使用的缓冲队列。阻塞队列，存储等待执行的任务，很重要。会对线程池运行过程产生重大影响
  * **keepAliveTime**：线程没有任务执行时最多保持多久时间终止
  * **unit**：keepAliveTime的时间单位
  * **threadFactory**：线程工厂，用来创建线程
  * **rejectHandler**：当拒绝处理任务是的策略

* 核心参数**corePoolSize**和**maximumPoolSize**的作用关系

  1. **poolSize < corePoolSize时：**当池中正在运行的线程数（包括空闲线程）小于corePoolSize时，直接新建线程执行任务，而不管是否有空闲的线程

  2. **corePoolSize < poolSize < maximumPoolSize时：**

     1. **poolSize < maximumPoolSize：**如果有多于corePoolSize但小于maximumPoolSize线程正在运行，则仅当队列已满时才会创建新线程。
     2. **poolSize = maximumPoolSize：**那么意味着线程池的处理能力已经达到了极限，此时需要拒绝新增加的任务。至于如何拒绝处理新增的任务，取决于线程池的饱和策略RejectedExecutionHandler。

  3. **corePoolSize = maximumPoolSize**通过设置corePoolSize和maximumPoolSize相同，您可以创建一个固定大小的线程池。 

  4. 通过将maximumPoolSize设置为基本上无界的值，例如Integer.MAX_VALUE，您可以允许池容纳任意数量的并发任务。

  5. 通常，核心和最大池大小仅在构建时设置，但也可以使用`setCorePoolSize`和`setMaximumPoolSize`进行动态更改。

     **这段话详细了描述了线程池对任务的处理流程，这里用个图总结一下**

     ![img](https://upload-images.jianshu.io/upload_images/11183270-a01aea078d7f4178.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  6. 这两个参数设置项一般经验：**待补充**

  7. 拒绝策略

     拒绝任务有两种情况：1. 线程池已经被关闭；2. 任务队列已满且maximumPoolSizes已满；无论哪种情况，都会调用RejectedExecutionHandler的rejectedExecution方法。预定义了四种处理策略：

     1. **AbortPolicy**：默认测策略，抛出RejectedExecutionException运行时异常；
     2. **CallerRunsPolicy**：这提供了一个简单的反馈控制机制，可以减慢提交新任务的速度；
     3. **DiscardPolicy**：直接丢弃新提交的任务；
     4. **DiscardOldestPolicy**：如果执行器没有关闭，队列头的任务将会被丢弃，然后执行器重新尝试执行任务（如果失败，则重复这一过程）；


### 9-2 线程池-2

#### 1.线程池的状态

参考博文：https://www.cnblogs.com/-wyl/p/9760670.html

线程池的5种状态：Running、ShutDown、Stop、Tidying、Terminated。

![](E:\markdown笔记\笔记图片\8\42.jpg)

* **RUNNING**：

  1. 状态说明：线程池处在RUNNING状态时，能够接收新任务，以及对已添加的任务（阻塞队列中的任务）进行处理。
  2. 状态切换：线程池的初始化状态是RUNNING。换句话说，线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0

*  **SHUTDOWN**：

  1. 状态说明：线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务。

  2. 状态切换：调用线程池的shutdown()接口时，线程池由RUNNING -> SHUTDOWN。

* **STOP**：

  1. 状态说明：线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。
  2. 状态切换：调用线程池的shutdownNow()接口时，线程池由(RUNNING or SHUTDOWN ) -> STOP。

* **TIDYING**：

  1. 状态说明：当所有的任务已终止，线程池中记录的”任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。
  2. 状态切换：当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。
     当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。

* **TERMINATED**：

  1. 状态说明：线程池彻底终止，就变成TERMINATED状态。
  2. 状态切换：线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。

#### 2.ThreadPoolExecutor基础方法

* execute()：提交任务，交给线程池执行
* submit()：提交任务，能够返回执行结果（相当于execute + Future）
* shutdown()：关闭线程池，等待任务执行完
* shutdownNow()：关闭线程池，不等待任务执行完

#### 3.ThreadPoolExecutor监控的方法

* getTaskCount()：线程池已执行和未执行的任务总数
* getCompletedTaskCount()：已完成的任务数量
* getPoolSize()：线程池当前的线程数量
* getActiveCount()：当前线程池中正在执行任务的线程数量

#### 4.线程池类图

![](E:\markdown笔记\笔记图片\8\42.png)

#### 5.Executor框架接口

* Executors.newCachedThreadPool
* Executors.newFixedThreadPool
* Executors.newScheduledThreadPool
* Executors.newSingleThreadExceutor