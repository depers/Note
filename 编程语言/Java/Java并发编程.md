---
ForkJointypora-root-url: 笔记图片
---

# Java并发编程

## 第1章  课程准备

### 1-1 课程导学

![](..\..\笔记图片\8\1.png)

![2](..\..\笔记图片\8\2.png)   

![3](..\..\笔记图片\8\3.png)

![4](..\..\笔记图片\8\4.png)

### 1-2 并发编程初体验

实现一个计数功能

1. 计数count，同时并发的线程数为200，参见代码com.bravedawn.concurrency.example.count.CountExample

   ![](..\..\笔记图片\8\5.png)

2. 计数map，同时并发的线程数为200，参加代码com.bravedawn.concurrency.example.count.MapExample

   ![](..\..\笔记图片\8\6.png)

   可以看到上面两个实验的计数结果都不能达到5000。

3. 计数count，同时并发的线程数为1，参见代码com.bravedawn.concurrency.example.count.CountExample

   ![](..\..\笔记图片\8\7.png)

4. 计数map，同时并发的线程数为1，参加代码com.bravedawn.concurrency.example.count.MapExample

   ![](..\..\笔记图片\8\8.png)

   当同时并发的线程数修改为1时，计数器每次都能到达5000。

### 1-3 并发与高并发基本概念   

![](..\..\笔记图片\8\9.png)

![10](..\..\笔记图片\8\10.png)

![11](..\..\笔记图片\8\11.png)

![12](..\..\笔记图片\8\12.png)

![](..\..\笔记图片\8\13.png)

## 第2章 并发基础

### 2-1 CPU多级缓存-缓存一致性

![](..\..\笔记图片\8\14.png)

![15](..\..\笔记图片\8\15.png)

![16](..\..\笔记图片\8\16.png)

![17](..\..\笔记图片\8\17.png)

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

![](..\..\笔记图片\8\18.png)

#### MESI协议的操作

* **Local Read**：读本地缓存中的数据
* **Local Write**：将数据写到本地的缓存中
* **Remote Read**：读取内存中的数据
* **Remote Write**：将数据写回到内存中去

![](..\..\笔记图片\8\19.png)

参考文章：https://www.cnblogs.com/z00377750/p/9180644.html

###2-2 CPU多级缓存-乱序执行优化

为了使得处理器内部的运算单元尽量被充分利用，提高运算效率，处理器可能会对输入的代码进行乱序执行。

![](..\..\笔记图片\8\20.png)

### 2-3 JAVA内存模型

Java 内存模型（Java Memory Model，JMM）是一种规范，**定义了 Java 虚拟机（JVM）在多线程环境下如何进行内存管理和线程间通信。它描述了线程之间共享变量的可见性、顺序性以及同步原语的行为**。
Java 内存模型的主要目标是确保在多线程环境下，程序的正确性和一致性。它通过定义一系列规则和语义，来保证线程之间的可见性、顺序性和原子性。

* 可见性：可见性确保当一个线程修改了共享变量的值时，其他线程能够立即看到这个修改。Java 内存模型通过将变量存储在主内存中，并在每个线程的本地内存中维护一个副本，来实现可见性。当一个线程修改了变量的值时，会将新值刷新到主内存中，并使其他线程的本地内存中的副本失效。
* 顺序性：顺序性保证了在单线程环境下程序的执行顺序与代码的顺序一致。在多线程环境下，Java 内存模型通过happens-before 关系来定义顺序性，**即如果一个操作的结果对另一个操作可见，那么这两个操作之间存在 happens-before 关系**。
* 原子性：原子性确保了对共享变量的操作是不可分割的，要么全部完成，要么都不完成。Java 内存模型提供了一些原子操作，如 volatile 关键字修饰的变量读写、synchronized 关键字修饰的代码块等。

> 参考文章：https://zhuanlan.zhihu.com/p/29881777

#### 定义

我们常说的JVM内存模式指的是JVM的内存分区；而Java内存模式是一种虚拟机规范。

**Java虚拟机规范中定义了Java内存模型（Java Memory Model，JMM），用于屏蔽掉各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的并发效果，JMM规范了Java虚拟机与计算机内存是如何协同工作的：规定了一个线程如何和何时可以看到由其他线程修改过后的共享变量的值，以及在必须时如何同步的访问共享变量。**

原始的Java内存模型存在一些不足，因此Java内存模型在Java1.5时被重新修订。这个版本的Java内存模型在Java8中仍然在使用。

#### JVM对Java内存模型的实现

下图展示了**调用栈和本地变量都存储在栈区**，**对象都存储在堆区**：

![](..\..\笔记图片\8\21.png)

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

![](..\..\笔记图片\8\23.png)

- **多CPU**：一个现代计算机通常由两个或者多个CPU。其中一些CPU还有多核。从这一点可以看出，在一个有两个或者多个CPU的现代计算机上同时运行多个线程是可能的。每个CPU在某一时刻运行一个线程是没有问题的。这意味着，如果你的Java程序是多线程的，在你的Java程序中每个CPU上一个线程可能同时（并发）执行。
- **CPU寄存器**：每个CPU都包含一系列的寄存器，它们是CPU内存的基础。CPU在寄存器上执行操作的速度远大于在主存上执行的速度。这是因为CPU访问寄存器的速度远大于主存。
- **高速缓存cache**：由于计算机的存储设备与处理器的运算速度之间有着几个数量级的差距，所以现代计算机系统都不得不加入一层读写速度尽可能接近处理器运算速度的高速缓存（Cache）来作为内存与处理器之间的缓冲：将运算需要使用到的数据复制到缓存中，让运算能快速进行，当运算结束后再从缓存同步回内存之中，这样处理器就无须等待缓慢的内存读写了。CPU访问缓存层的速度快于访问主存的速度，但通常比访问内部寄存器的速度还要慢一点。每个CPU可能有一个CPU缓存层，一些CPU还有多层缓存。在某一时刻，一个或者多个缓存行（cache lines）可能被读到缓存，一个或者多个缓存行可能再被刷新回主存。
- **内存**：一个计算机还包含一个主存。所有的CPU都可以访问主存。主存通常比CPU中的缓存大得多。
- **运作原理**：通常情况下，当一个CPU需要读取主存时，它会将主存的部分读到CPU缓存中。它甚至可能将缓存中的部分内容读到它的内部寄存器中，然后在寄存器中执行操作。当CPU需要将结果写回到主存中去时，它会将内部寄存器的值刷新到缓存中，然后在某个时间点将值刷新回主存。

#### Java内存模型和硬件内存架构之间的桥接

Java内存模型与硬件内存架构之间存在差异。硬件内存架构没有区分线程栈和堆。对于硬件，所有的线程栈和堆都分布在主内存中。部分线程栈和堆可能有时候会出现在CPU缓存中和CPU内部的寄存器中。如下图所示：

![](..\..\笔记图片\8\24.jpg)

下图是**Java内存模型抽象结构示意图**，从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：

- 线程之间的**共享变量**存储在**主内存**（Main Memory）中
- **每个线程**都有一个私有的**本地内存**（Local Memory），本地内存是JMM的一个抽象概念，并不真实存在，它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。本地内存中存储了该线程以读/写共享变量的拷贝副本。
- 从更低的层次来说，主内存就是硬件的内存，而为了获取更好的运行速度，虚拟机及硬件系统可能会让工作内存优先存储于寄存器和高速缓存中。
- Java内存模型中的**线程的工作内存**（working memory）是cpu的寄存器和高速缓存的抽象描述。而**JVM的静态内存储模型（JVM内存模型）**只是一种对内存的物理划分而已，它只局限在内存，而且只局限在JVM的内存。

![](..\..\笔记图片\8\25.jpg)

#### JMM模型下的线程间通信

线程间通信必须要经过主内存。

如下，如果线程A与线程B之间要通信的话，必须要经历下面2个步骤：

1）线程A把本地内存A中更新过的共享变量刷新到主内存中去。

2）线程B到主内存中去读取线程A之前已更新过的共享变量。

![](..\..\笔记图片\8\26.jpg)

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

![](..\..\笔记图片\8\27.png)



## 第4章 线程安全性

### 1.线程安全性-原子性

* 线程安全性的定义

  ![](..\..\笔记图片\8\8-1.png)

* 线程安全性的组成

  ![8-2](..\..\笔记图片\8\8-2.png)

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
  
  1. 可以发现，CAS实现的过程是先取出内存中某时刻的数据，在下一时刻比较并替换，那么在这个时间差会导致数据的变化，此时就会导致出现“ABA”问题。 
  
  2. 什么是”ABA”问题？ 
  
      在 Java 中，ABA 问题通常指的是在使用volatile关键字修饰变量时可能出现的一种情况。volatile关键字用于确保变量的可见性和顺序性，但在某些情况下，可能会导致 ABA 问题。
      ABA 问题的具体表现是：
  
      * 线程 A 读取变量的值为 X。
      * 线程 B 将变量的值修改为 Y。
      * 线程 C 将变量的值修改回 X。
      * 线程 A 再次读取变量的值，仍然为 X，但实际上变量的值已经被线程 B 和线程 C 先后修改过。
  
      在这种情况下，线程 A 无法得知变量的值是否被其他线程修改过，因为最后一次读取到的值仍然是 X。
  
 * `AtomicStampReference`中的核心方法`compareAndSet`。除了对象值，AtomicStampedReference内部还维护了一个“状态戳” **stamp**。状态戳可类比为时间戳，是一个整数值，每一次修改对象值的同时，也要修改状态戳，从而区分相同对象值的不同状态。当AtomicStampedReference设置对象值时，对象值以及状态戳都必须满足期望值，写入才会成功
  
 * `AtomicLongArray`：根据索引原子更新长整型数组里的元素

### 2.原子性——锁（JDK中提供原子性的除了atomic之外还有锁——下面的两个锁）

![8-3](..\..\笔记图片\8\8-3.png)

* **synchronized**它是一种同步锁

  ![8-4](..\..\笔记图片\8\8-4.png)

  

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

  ![8-5](..\..\笔记图片\8\8-5.png)

### 3.可见性

**可见性**则更为微妙，它必须确保释放锁之前对共享数据做出的更改对于随后获得该锁的另一个线程是可见的。 —— 如果没有同步机制提供的这种可见性保证，线程看到的共享变量可能是修改前的值或不一致的值，这将引发许多严重问题。

![8-6](..\..\笔记图片\8\8-6.png)

* 可见性——synchronized

  ![8-7](..\..\笔记图片\8\8-7.png)

* 可见性——volatile

  ![8-8](..\..\笔记图片\8\8-8.png)

* volatile读写操作加入内存屏障和禁止重排序的CPU指令示意图
    ![8-10](..\..\笔记图片\8\8-10.png)

    ![8-9](..\..\笔记图片\8\8-9.png)

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

  * 对变量的写操作不依赖于当前值，也就是说不适合做计数操作，很明显该操作依赖当前值。
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

![8-11](..\..\笔记图片\8\8-11.png)

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

![](..\..\笔记图片\8\28.png)

### volatile关键字

在 Java 中，volatile关键字用于修饰变量，它可以确保变量的可见性和顺序性，但不保证原子性。下面是volatile关键字的原理和使用场景：
1. 原理：

    * 可见性：当一个线程修改了被volatile修饰的变量时，其他线程可以立即看到这个变量的最新值，而不需要通过锁机制来保证可见性。这是因为 Java 内存模型会在写操作后将变量的值立即刷新到主内存中，并使其他线程的缓存失效。
    * 顺序性：在多线程环境下，volatile关键字可以保证代码的执行顺序，即在一个线程中，对volatile变量的操作会按照顺序执行，不会被重排序。这是因为 Java 编译器在生成指令时，会遵循happens-before 原则，确保对volatile变量的写操作一定在其读操作之前完成。

2. 使用场景：

    * 共享变量：当一个变量被多个线程共享时，如果不使用volatile关键字，可能会出现可见性问题。使用volatile关键字可以确保变量在多个线程之间的可见性。

    * 状态标志：在某些情况下，需要使用一个变量来表示某个状态，例如线程的启动或停止状态。使用volatile关键字可以确保线程在读取状态标志时获取到最新的值。
    * 双重检查锁定：在双重检查锁定（Double-Checked Locking）的实现中，需要使用volatile关键字来保证变量的可见性和顺序性。

## 第5章 安全发布对象

### 1.发布与逸出

> com.bravedawn.concurrency.example.publish

![](..\..\笔记图片\8\29.png)

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

![8-12](..\..\笔记图片\8\8-12.png)

* 创建不可变对象

  这里可以采用的方式有：

  * 将类声明为final，不允许继承
  * 将类所有的成员变量声明为私有的，这样就不能直接访问类成员
  * 对变量不提供set方法，将所有可变的成员声明为final，这样只能对他们赋值一次
  * 在构造方法中所有变量进行赋值，进行深度拷贝
  * 在get方法中不直接返回变量本身而是克隆对象，并返回对象的拷贝。

  对于要创建不可变对象，我们应多参考String和final类

* final关键字

  ![](..\..\笔记图片\8\30.png)

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

  ![8-13](..\..\笔记图片\8\8-13.png)

  * Java提供的Collections类，值得注意的是Collections修改的对象不能是final的。使用参见com.bravedawn.concurrency.example.immutable.ImmutableExample2
  * Guava提供的ImmutableXXX，值得注意的是Collections修改的对象可以是final的。使用参见com.bravedawn.concurrency.example.immutable.ImmutableExample3

### 2.线程封闭

避免并发除了将对象声明为不可变对象之外，还有一个简单的方法就是线程封闭。

为了确保线程的安全，通常需要保证可变的共享数据的同步访问，具体采用的方式有很多；但还有一种方法可以保证线程的安全，即是使可变数据不共享，或者是使数据不可变。**所谓“线程封闭”即是仅在单线程中访问数据，也就是通过让可变数据不被多个线程共享以确保数据的正确性。**

![8-14](..\..\笔记图片\8\8-14.png)

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
  
    ThreadLocal是解决线程安全问题一个很好的思路，它**通过为每个线程提供一个独立的变量副本，解决了变量并发访问的冲突问题**。在很多情况下，ThreadLocal比直接使用synchronized同步机制解决线程安全问题更简单，更方便，且结果程序拥有更高的并发性。
  
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
  * 当多个线程同时对 ArrayList 进行修改操作时，可能会导致数据不一致或者出现异常。这是因为 ArrayList 的内部结构不是线程安全的，它没有提供对并发修改的支持。例如，当一个线程正在向 ArrayList 中添加元素，而另一个线程同时在删除元素，就有可能导致**索引越界**或者**元素丢失**的问题。
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

![](..\..\笔记图片\8\31.png)

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

![](..\..\笔记图片\8\32.png)

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

![](..\..\笔记图片\8\33.png)

![34](..\..\笔记图片\8\34.png)

![35](..\..\笔记图片\8\35.png)

## 第7章 J.U.C之AQS

### 7-1 J.U.C之AQS-介绍

![](..\..\笔记图片\8\36.png)

* AQS的设计
  * 基于队列的等待机制：AQS 使用队列来管理等待锁的线程。当一个线程尝试获取锁而锁不可用时，该线程会被添加到等待队列中，并在锁可用时被唤醒。
  * 支持多种锁类型：AQS 是一个通用的同步器框架，可以支持多种不同类型的锁，如互斥锁、读写锁、信号量等。通过继承 AQS 并实现特定的方法，可以创建不同类型的锁。
  * 可重入性：AQS 支持锁的可重入性，即同一个线程可以多次获取同一个锁而不会造成死锁。
  * 公平性：AQS 提供了公平锁和非公平锁的实现。公平锁按照先来先服务的原则分配锁，而非公平锁可能会优先唤醒等待时间最长的线程。
  * 状态共享：AQS 内部维护了一个共享的状态变量，用于表示锁的状态。通过原子操作来修改状态变量，可以保证线程安全性。
  * 锁释放：在释放锁时，AQS 会唤醒等待队列中的一个线程，并通过线程安全的方式更新状态变量。
  * 线程安全：AQS 中的方法都是线程安全的，通过使用内部的同步机制（如volatile关键字和原子操作）来保证线程安全性。

  总之，AQS 的设计思想是基于队列的等待机制、支持多种锁类型、可重入性、公平性、状态共享、锁释放和线程安全等方面。AQS 为 Java 并发包中的许多同步工具提供了基础支持，使得它们能够以高效和安全的方式实现线程同步。

* AQS的组件
  * CountDownLatch
  * Semaphore
  * 
  * ReentrantLock
  * Condition
  * FutureTask

### 7-2 J.U.C之AQS-CountDownLatch

1. 概念

   这个图应该从上往下看。

   ![](..\..\笔记图片\8\37.png)

   countDownLatch这个类使一个线程等待其他线程各自执行完毕后再执行。

   是通过一个计数器来实现的，计数器的初始值是线程的数量。每当一个线程执行完毕后，计数器的值就-1，当计数器的值为0时，表示所有线程都执行完毕，然后在闭锁上等待的线程就可以恢复工作了。

2. 实践

   1. 使用CountDownLatch计数：com.bravedawn.concurrency.example.aqs.CountDownLatchExample1
   2. 使用CountDownLatch计数，若在有限的时间内未完成任务，则不再等待，直接结束；或者是有得时候，忘记调用`countDown()`方法时可以设置该值：com.bravedawn.concurrency.example.aqs.CountDownLatchExample2

### 7-3 J.U.C之AQS-Semaphore（信号量）

![](..\..\笔记图片\8\38.png)

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

![](..\..\笔记图片\8\39.png)

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

      * 线程进入读锁的前提条件：当前线程已经持有了写锁或者当前线程没有持有任何锁且其他线程没有持有写锁

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

   ![](..\..\笔记图片\8\40.png)

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

   ![](..\..\笔记图片\8\41.png)

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
* 线程缺乏统一管理，可能无限制的新建线程，相互竞争，有坑内占用过多系统资源导致死机或OOM
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

     1. **AbortPolicy**：默认测策略，抛出异常RejectedExecutionException拒绝提交任务；
     2. **CallerRunsPolicy**：由调用execute方法提交任务的线程来执行这个任务；
     3. **DiscardPolicy**：直接丢弃新提交的任务；
     4. **DiscardOldestPolicy**：如果执行器没有关闭，队列头的任务将会被丢弃，然后执行器重新尝试执行任务（如果失败，则重复这一过程）；


### 9-2 线程池-2

#### 1.线程池的状态

参考博文：https://www.cnblogs.com/-wyl/p/9760670.html

线程池的5种状态：Running、ShutDown、Stop、Tidying、Terminated。

![](..\..\笔记图片\8\42.jpg)

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

![](..\..\笔记图片\8\42.png)

#### 5.Executor框架接口

* Executors.newCachedThreadPool：创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
* Executors.newFixedThreadPool： 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
* Executors.newScheduledThreadPool：创建一个定长线程池，支持定时及周期性任务执行。
* Executors.newSingleThreadExceutor：创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

### 9-3 线程池-3

关于9-2第5点中提到的四种不同的线程池，其实现方法都在java.util.concurrent.Executors类中，其核心都是通过ThreadPoolExecutor类实现的。

#### 1.线程池的使用

* Executors.newCachedThreadPool：参见com.bravedawn.concurrency.example.threadPool.ThreadPoolExample1
* Executors.newFixedThreadPool：参见com.bravedawn.concurrency.example.threadPool.ThreadPoolExample2
* Executors.newSingleThreadExecutor：参见com.bravedawn.concurrency.example.threadPool.ThreadPoolExample3
* Executors.newScheduledThreadPool：参见
  * schedule：com.bravedawn.concurrency.example.threadPool.ThreadPoolExample4
  * scheduleAtFixedRate与Timer：com.bravedawn.concurrency.example.threadPool.ThreadPoolExample5
  * scheduleWithFixedDelay：com.bravedawn.concurrency.example.threadPool.ThreadPoolExample6

#### 2.线程池的合理配置

线程池大小的设置：

1. CPU密集型任务，就需要尽量压榨CPU，参考值可以设置为**CPU数量+1**

    在配置线程池的核心线程数时，将核心线程数设置为 CPU 数量加 1 的原因是**为了充分利用 CPU 资源并提供一定的冗余**。

    每个 CPU 核心都可以同时执行一个线程，因此将核心线程数设置为 CPU 数量可以确保每个 CPU 核心都有一个线程在执行任务。这样可以最大限度地利用 CPU 资源，提高程序的性能。

    然而，有时候可能会出现一些意外情况，例如某个线程因为等待 I/O 操作或其他原因而**被阻塞**。如果此时所有的 CPU 核心都被占用，那么就没有线程可以继续执行任务，导致 CPU 资源的浪费。

    因此，将核心线程数设置为 CPU 数量加 1 可以提供一个额外的线程作为冗余。这样，即使有一个线程被阻塞，仍然有一个线程可以继续执行任务，从而充分利用 CPU 资源。

2. IO密集型任务，参考值可以设置为**2*CPU数量**

    如果你的应用程序有大量的阻塞操作或 I/O 密集型任务，可能需要更多的核心线程数来提高性能。

#### 3.JUC总览

![](..\..\笔记图片\8\43.png)

## 第10章 多线程并发拓展

### 10-1 死锁

1. 定义

   死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。

2. 产生死锁的四个必要条件

   * 互斥条件

     进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。

   * 请求和保持条件

     进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。

   * 不剥夺条件

     进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能由获得该资源的进程自己来释放（只能是主动释放)。

   * 循环等待条件

     若干进程间形成首尾相接循环等待资源的关系。

3. 死锁实践：com.bravedawn.concurrency.example.deadLock.DeadLock

4. 如何避免死锁

   1. 调整加锁顺序：应该通过代码控制，两个线程同时对o1加锁或者同时对o2加锁。这样才可以，否则就会产生死锁。等于破坏了循环等待条件
   2. 添加加锁时限：在线程要求加锁时超过一定的时限，就放弃请求并释放自己的资源。破坏请求和保持条件
   3. 死锁检测

### 10-2 并发最佳实践

1. **使用本地变量**

   应该总是使用本地变量，而不是创建一个类或实例变量，通常情况下，开发人员使用对象实例作为变量可以节省内存并可以重用，因为他们认为每次在方法中创建本地变量会消耗很多内存。

   这里的本地变量指的就是使用使用**方法内的局部变量**。

2. **使用不可变类**

   不可变类比如String Integer等一旦创建，不再改变，不可变类可以降低代码中需要的同步数量。

3. **最小化锁的作用域范围：S=1/(1-a+a/n)**

  a：并行计算部分所占比例

  n：并行处理结点个数

  S：加速比

  当1-a等于0时，没有串行只有并行，最大加速比 S=n

  当a=0时，只有串行没有并行，最小加速比 S = 1

  当n→∞时，极限加速比 s→ 1/（1-a）

  例如，若串行代码占整个代码的25%，则并行处理的总体性能不可能超过4。

  该公式称为："阿姆达尔定律"或"安达尔定理"。

  任何在锁中的代码将不能被并发执行，如果你有5%代码在锁中，那么根据Amdahl's law，你的应用形象就不可能提高超过20倍，因为锁中这些代码只能顺序执行，降低锁的涵括范围，上锁和解锁之间的代码越少越好。

4. **使用线程池的Executor，而不是直接new  Thread 执行**

  创建一个线程的代价是昂贵的，如果你要得到一个可伸缩的Java应用，你需要使用线程池，使用线程池管理线程。JDK提供了各种ThreadPool线程池和Executor。

5. **宁可使用同步也不要使用线程的wait和notify**

   从Java 1.5以后增加了需要同步工具如CycicBariier, CountDownLatch 和 Sempahore，你应当优先使用这些同步工具，而不是去思考如何使用线程的wait和notify，通过BlockingQueue实现生产-消费的设计比使用线程的wait和notify要好得多，也可以使用CountDownLatch实现多个线程的等待。

6. **使用BlockingQueue实现生产-消费模式**

   大部分并发问题都可以使用producer-consumer生产-消费设计实现，而BlockingQueue是最好的实现方式，堵塞的队列不只是可以处理单个生产单个消费，也可以处理多个生产和消费。

7. **使用并发集合而不是加了锁的同步集合**

   Java提供了 ConcurrentHashMap CopyOnWriteArrayList 和 CopyOnWriteArraySet以及BlockingQueue Deque and BlockingDeque五大并发集合，宁可使用这些集合，也不用使用Collections.synchronizedList之类加了同步锁的集合， CopyOnWriteArrayList 适合主要读很少写的场合，ConcurrentHashMap更是经常使用的并发集合

8. **使用Semaphore创建有界**

   为了建立稳定可靠的系统，对于数据库、文件系统和socket等资源必须要做有机的访问，Semaphore可以限制这些资源开销的选择，Semaphore可以以最低的代价阻塞线程等待，可以通过Semaphore来控制同时访问指定资源的线程数。

9. **宁可使用同步代码块，也不使用加同步的方法**

   使用synchronized 同步代码块只会锁定一个对象，而不会将当前整个方法锁定；如果更改共同的变量或类的字段，首先选择原子性变量，然后使用volatile。如果你需要互斥锁，可以考虑使用ReentrantLock 

10. **避免使用静态变量**

   静态变量在并发执行环境下会制造很多问题，如果必须使用静态变量，那么优先是它成为final变量，如果用来保存集合collection，那么可以考虑使用只读集合，否则一定要做特别多的同步处理和并发处理操作。

参考博客：

* https://www.cnblogs.com/shamo89/p/9777795.html
* https://www.jdon.com/concurrent/java-multithreading-best-pratices.html

### 10-3 [Spring与线程安全](https://www.cnblogs.com/shamo89/p/9777904.html)

Spring作为一个IOC/DI容器，帮助我们管理了许许多多的“bean”。但其实，**Spring并没有保证这些对象的线程安全**，需要由开发者自己编写解决线程安全问题的代码。

Spring对每个bean提供了一个scope属性来表示该bean的作用域。它是bean的生命周期。例如，一个scope为singleton的bean，在第一次被注入时，会创建为一个单例对象，该对象会一直被复用到应用结束。

- **singleton**：**默认的scope**，每个scope为singleton的bean都会被定义为一个单例对象，该对象的生命周期是与Spring IOC容器一致的（但在第一次被注入时才会创建）。
- **prototype**：bean被定义为在每次注入时都会创建一个新的对象。
- **request**：bean被定义为在每个HTTP请求中创建一个单例对象，也就是说在单个请求中都会复用这一个单例对象。
- **session**：bean被定义为在一个session的生命周期内创建一个单例对象。
- **application**：bean被定义为在ServletContext的生命周期中复用一个单例对象。
- **websocket**：bean被定义为在websocket的生命周期中复用一个单例对象。

**我们交由Spring管理的大多数对象其实都是一些无状态的对象**，这种不会因为多线程而导致状态被破坏的对象很适合Spring的默认scope，每个单例的无状态对象都是线程安全的（也可以说只要是无状态的对象，不管单例多例都是线程安全的，不过单例毕竟节省了不断创建对象与GC的开销）。

**无状态的对象即是自身没有状态的对象，自然也就不会因为多个线程的交替调度而破坏自身状态导致线程安全问题。无状态对象包括我们经常使用的DO、DTO、VO这些只作为数据的实体模型的贫血对象，还有Service、DAO和Controller，这些对象并没有自己的状态，它们只是用来执行某些操作的。**例如，每个DAO提供的函数都只是对数据库的CRUD，而且每个数据库Connection都作为函数的局部变量（局部变量是在用户栈中的，而且用户栈本身就是线程私有的内存区域，所以不存在线程安全问题），用完即关（或交还给连接池）。

有人可能会认为，我使用request作用域不就可以避免每个请求之间的安全问题了吗？这是完全错误的，因为Controller默认是单例的，一个HTTP请求是会被多个线程执行的，这就又回到了线程的安全问题。当然，你也可以把Controller的scope改成prototype，实际上Struts2就是这么做的，但有一点要注意，Spring MVC对请求的拦截粒度是基于每个方法的，而Struts2是基于每个类的，所以把Controller设为多例将会频繁的创建与回收对象，严重影响到了性能。

通过阅读上文其实已经说的很清楚了，Spring根本就没有对bean的多线程安全问题做出任何保证与措施。对于每个bean的线程安全问题，根本原因是每个bean自身的设计。**不要在bean中声明任何有状态的实例变量或类变量**，如果必须如此，那么就使用ThreadLocal把变量变为线程私有的，如果bean的实例变量或类变量需要在多个线程之间共享，那么就只能使用synchronized、lock、CAS等这些实现线程同步的方法了。

###  10-4 HashMap与ConcurrentHashMap解析

#### HashMap

![](..\..\笔记图片\8\45.webp)

1. hashMap由**数组+链表**组成，影响HashMap性能的2个因素：

   * **初始容量**，默认为16，表示哈希表中桶的数量；

     ```java
     static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
     ```

   * **加载因子**，默认值为0.75，表示哈希表的容量在自动增加之前可以达到多满的尺度，当哈希表中的条目数=哈希表的容量*加载因子，将触发扩容操作resize()；

     ```
     static final float DEFAULT_LOAD_FACTOR = 0.75f;
     ```

     *注：初始容量和加载因子2个值在哈希表初始化的时候可以设定；*

2. HashMap的寻址方式

   * 对于新插入的数据和要读取的数据，HashMap会将其key按照一定计算规则计算出哈希值，并对数组长度进行取模，结果作为其插入数组的index；

   * 取模的代价远高于位运算的代价，所以HashMap中数组的长度要求必须是2^n， 具体为，将取模操作换成对2^n-1的与运算；

   * HashMap初始化的时候传入的容量可以不是2^n， HashMap会根据传入的值得到一个相应的2^n的值；（具体方法：java.util.HashMap#tableSizeFor）

   * HashMap是线程不安全的，主要体现在resize的时候可能出现**死循环**，以及在使用迭代器的时候会出现**ParsedFailed**；

   * 扩容操作的具体做法：创建一个新的，长度为原来容量2倍的数组，保证新的容量仍为2^n，并将原来的数组重新插入新数组，这个过程称为**再哈希（rehash）**，这个方法不是线程安全的；

3. 单线程下的rehash操作

   单线程情况下，rehash无问题。下图演示了单线程条件下的rehash过程

   ![](..\..\笔记图片\8\46.webp)

4. 多线程rehash时候出现死循环

   * 这里假设有两个线程同时执行了put操作并引发了rehash，处理5到一半，5的next指向了9，因为线程调度所分配时间片用完而“暂停”，此时线程二完成了rehash方法的执行。此时状态如下。

   ![](..\..\笔记图片\8\47.png)

   * 接着线程1被唤醒，继续执行第一轮循环的剩余部分，把5挂在索引为1的元素后面；

     ![](..\..\笔记图片\8\48.png)

   * 由于在线程1中，接着会处理9，线程1会把9再放到5前面，这是在新数组中完成的；

     ![](..\..\笔记图片\8\49.webp)

   * 处理完9后，线程1看到9后面还有5，这是因为线程2已经完成了ReHash动作，然后又将5排在9前面。9在5前面，5在9前面，这里就会出现一个循环链表。而元素11是不会再被加到线程1的新数组中了；当下一次再次访问到这个链表就会出现死循环。

     ![](..\..\笔记图片\8\50.webp)

5. ParsedFailed分析

   在遍历HashMap的时候，如果HashMap中的元素被修改了，就会抛出ConcurrentModifacationException，这就是所说的ParseFailed；

   解决方法：

   - Collections.synchronizedMap()构造出一个同步的Map；
   - 或者直接使用ConcurrentHashMap()；

#### ConcurrentHashMap

![](..\..\笔记图片\8\51.webp)

* 与HashMap不同的是，ConcurrentHashMap外面不是一个大数组，而是多个小数组，每个小数组是一个Segment；

* 在读写某个key时，先计算出该key的哈希值，并将哈希值的高sshift位对Segment个数取模，从而得到该key属于哪个Segment；

* 接着就像操作HashMap一样操作Segment；

* Segment继承自J.U.C的ReentrantLock，所以可以很方便的对每个Segment上锁，就是所谓**分段锁**；

#### HashMap与ConcurrentHashMap的区别

* ConcurrentHashMap是线程安全的，HashMap不是线程安全的；

* HashMap允许key和value为空，ConcurrentHashMap不允许；

* HashMap在用iterator遍历的同时，不允许修改HashMap；ConcurrentHashMap允许该行为，并且更新对后续的遍历是可见的；

#### Java8对ConcurrentMap的改进

* Java7中的ConcurrentHashMap理论上最大并发数和Segment个数相等；

* Java8废弃了分段锁的方案，直接使用一个大数组，并对哈希碰撞下的寻址做了优化，当链表的长度超过一定值（默认8），列表将转换成红黑树，寻址的时间复杂度从O(n)降为O(logn)；

  ![](..\..\笔记图片\8\52.webp)

### 10-5 多线程并发与线程安全总结

![](..\..\笔记图片\8\53.png)

## 第11章 高并发之扩容思路

#### 扩容

* 垂直扩容（纵向扩展）：提高系统部件能力
* 水平扩容（横向扩展）：增加耕读系统成员来实现

#### 扩容-数据库

* 读操作扩展（读多）：memcache、redis、CDN等缓存
* 写操作扩展（写多）：Cassandra、Hbase等

## 第12章 高并发之缓存思路

### 12-1 高并发之缓存-特征、场景及组件介绍

#### 概述

![](..\..\笔记图片\8\54.PNG)

#### 缓存的特征

- **命中率**（高并发中的重要指标）：简单表示为：命中数/(命中数+未命中数)；未命中即未通过缓存获取到想要的数据，可能数据不存在或缓存已过期。
- **最大元素（或最大空间）**：缓存中可以存放的`元素数量`。一旦缓存中的数据数量超过该值，会触发缓存的清除策略。
- **清空策略**：FIFO、LFU（使用频率最小策略）、LRU（最近最少被使用策略）、过期时间、随机清除等。
  - FIFO是first in first out，在缓存中最先被创建的数据再被清除时会被优先考虑清除。适用于`对数据的实时性要求较高的场景`，保证最新的数据可用。
  - LFU是无论数据是否过期，通过比较各数据的命中率，`优先清除命中率最低的数据`。
  - LRU是无论数据是否过期，通过比较数据最近一次被使用（即调用get()）的时间戳，优先清除距今最久的数据，以`保证热点数据的可用性`。（**多数缓存框架都采用LRU策略**）
  - 过期时间策略是，通过比较过期时间`清除过期时间最长的数据`，或通过过期时间，`清除最近要过期的数据`。

####  缓存命中率的影响因素

1. 业务场景和业务需求：显然，缓存是为了减轻读数据操作的缓冲方式。

- 适用于`读多写少`的场景。
- 对`数据的实时性要求不高`。因为数据缓存的时间越长，数据的命中率越高。

2. 缓存的设计：粒度和策略
   * 缓存的`粒度越小`，灵活度高，命中率越高；粒度越小，越不易被其他操作（修改）涉及到，保存在缓存中的时间越久，越易被命中。
   * 当缓存中的数据变化时，直接更新缓存中的相应数据（虽然一定程度上会提高系统复杂度），而不是移除或设置数据过期，会提高缓存的命中率。

3. 缓存容量和基础设施。
   * 当然，`缓存容量越大`，缓存命中率越高。
   * 缓存的技术选型：采用应用内置的`内地缓存`容易造成`单机瓶颈`，而采用`分布式缓存`则具有更好的拓展性，伸缩性。

4. 不同的缓存框架中的缓存命中率也不尽相同。

5. 当`缓存结点发生故障`，需要避免缓存失效并最大程度地减少影响。可通过`一致性hash`算法或结点冗余方式避免结点故障。

6. 多线程并发越高，缓存的效益越高，即收益越高，即使缓存时间很短。

#### 提高缓存命中率的方法

- 要求应用尽可能地通过缓存访问数据，避免缓存失效。
- 结合上面介绍的命中率影响因素：缓存粒度、策略、容量、技术选型等结合考虑。

#### 缓存的分类

根据缓存和应用的耦合度进行分类：

**1、本地缓存：**

- 指`应用中的缓存组件`，主要通过编程实现（使用成员变量、局部变量、静态变量）或使用现成的Guava Cache框架。
- 优点：应用和cache都在同一个进程内部。`请求缓存速度快`，`无网络开销`。单机应用中，不需要集群支持时；或集群情况下，`各结点不需要互相通知时`，适合使用本地缓存。
- 缺点：`应用和cache的耦合度高`，各应用间`无法共享缓存`，导致各结点或单机应用需要维护自己的缓存，可能会造成内存浪费。

**2、分布式缓存：**

- 指的是`应用分离的缓存组件或服务`，主要现在流行的有：`Memcache、Redis`。
- 优点：自己本身就是一个独立的服务，`与本地应用是隔离的`，`多个应用可以共享缓存`；

#### Guava Cache介绍

它是`本地缓存的实现框架`。

原理图如下：

![](..\..\笔记图片\8\55.png)

可以看出来，它的实现原理`类似ConcurrentHashMap`，使用`多个segment细粒度锁`，既保证了线程安全，又支持高并发场景需求。该类Cache类似于一个Map，也是存储键值对的集合，但它还需要处理缓存过期、动态加载等算法逻辑；根据面向对象的思想，它还需要做方法与数据关联性的封装。

Guava Cache实现的主要功能有：自动将结点加入到缓存结构中；当缓存中结点超过设置的最大元素值时，使用LRU算法实现缓存清除；它缓存的key封装在weakReference（弱引用）中，它缓存的value缓存在weakReference（弱引用）或softReference（软引用）中；它可以统计缓存中各数据的命中率、异常率、未命中率等数据；

#### Memcache

它是一个高效的`分布式内存cache`，是众多广泛应用的开源分布式缓存的框架之一。

内存结构图如下：

![](..\..\笔记图片\8\56.png)

**其中涉及到四个部分：按部分作用区域由大到小分别为**

1、slab_class：板层类。
2、slab：板层。
3、page：页。
4、chunk：块。是真正存放数据的地方。

- 同一个slab内的chunk的大小是固定的。而具有相同大小的chunk和slab被组织到一起，称为slab_class。
- Memcache的内存的分配方式称为allocator。其中slab的数量有限，与启动参数的配置有关。
- 其中的value总是会被存放到与value占用空间大小最接近的chunk的slab中，以在不降低系统性能情况下节约内存空间。
- 在创建slab时，首先申请内存；其中是以page为单位分配给slab空间，其中，一般page为1M大小；page按照该slab的chunk大小进行切分并形成数组。

**Memcache处理原理图如下：**

![](..\..\笔记图片\8\57.png)

它本身不提供分布式的解决方案。在服务端，Memcache的集群环境实际上就是一个个Memcache服务器的堆积。它cache的分布式机制是在客户端实现。通过客户端的路由来处理，以达到分布式解决方案的目的。

**客户端做路由的原理：**

- 客户端采用`hash一致性算法`，上图中右侧即是其路由的计算方法。
- 相对于一般的hash算法，如取模方式，它除了`计算key的hash外`，还计算`每一个server的hash值`，最后`将这些hash值映射到一个指定域上`，如图中的 **0-2^32**。
- 顺时针找到的第一个node节点，即为key存放的位置：通过寻找`大于key的hash值的最小server`作为存储该key的目标server；如果`未找到`，则直接将`具有最小hash值的server`作为目标server。
- 该机制一定程度上解决了扩容问题。增加或删除某个结点对于该集群来说不会有大影响。
- 最近版本中，Memcache又增加了`虚拟结点`的设计，进一步提高系统可用性

**Memcache的内存分配和回收算法**

- 在Memcache的内存分配中，chunk空间并不会被value完全占用，总会有内存浪费；
- Memcache的LRU回收算法不是针对全局的，而是slab的。
- Memcache中对允许存放的value占用空间大小有限制。因为内存空间分配是slab以page为单位被分配空间，而page大小规定`最大为1M`。

**Memcache的限制和特性**

1、Memcache`对存储的item数量没有限制`；

2、Memcache单进程在32位机器上限制的内存大小为2G，即2的32次方的bit。而64位机器则没有内存大小限制。

3、Memcache的key最大为250个字节；若超过则无法存储。其可存储的value最大为1M，即page最大可分配的大小，此时page中只分配了（形成）一个chunk。

4、Memcache的服务器端是非安全的。当已知一个Memcache结点，外部进入后可通过flushAll命令，使已经存在的键值对立即失效。

5、Memcache不能遍历其中存储的所有item。因为该操作会使其他的访问、创建等操作阻塞，且该进程十分缓慢。

6、Memcache的高性能来源于两个阶段的hash结构：第一个是客户端（该hash算法根据key值算出一个结点）；第二个是服务端（通过一个内部的hash算法，查找真正的item并返回给客户端）。

7、Memcache是一个`非阻塞的基于事件的服务器程序`。

#### Redis

Redis即Remote Dictionary Service的简称，即远程字典服务。

它是一个远程的`非关系型内存数据库`。性能强劲。具有复制特性，以及为解决数据而生的独一无二的数据模型：可以存储键值对、以及五种不同类型的值之间的映射。并提供将内存中数据持久化到硬盘功能。可以使用复制特性扩展读性能；可以使用客户端分片扩展写性能。

如下图所示：

![](..\..\笔记图片\8\58.png)

**Redis特点**

1、支持数据的持久化，将内存中数据持久化到硬盘，重启后可以再次加载至内存。
2、支持上图中string、hash、list、set、sorted set等数据类型。
3、支持数据的备份，即master-slave主从数据备份。
4、功能强大：

- 性能极高：`读取速度高达11W/s`，`写速度高达8.1W/s`（官方数据）。
- 支持丰富的数据类型。
- 其操作均具有原子性，包括多个操作后另外操作的原子性。
- 支持publish、subscribe、通知、key过期等等功能。

**Redis适用场景**

1、当需要取出n个最新数据的操作时。
2、当需要实现排行榜类似的应用时。
3、当需要精准设定过期时间的应用时。
4、当需要使用计数器的应用时。
5、当需要做唯一性检查时。
6、当需要获取某一时间段内所有数据排头的数据时。
7、实时系统。
8、垃圾系统。
9、使用pub、sub构建消息系统。
10、构建队列系统。
11、实现最基础的缓存功能时。

https://blog.csdn.net/csdnlijingran/article/details/83216601

[https://suprisemf.github.io/categories/%E9%AB%98%E5%B9%B6%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/](https://suprisemf.github.io/categories/高并发解决方案/)

### 12-3 高并发之缓存-redis的使用

详细代码参见：https://github.com/depers/concurrency/commit/9315d5a38eadf5b35b31d7150948ac311ed81f8b

### 12-4 高并发之缓存-高并发场景问题及实战讲解

#### 高并发场景下缓存常见问题

1. 缓存一致性

   当数据时效性要求很高时，需要保证缓存中的数据与数据库中的保持一致，而且需要保证缓存节点和副本中的数据也保持一致，不能出现差异现象。这就比较依赖缓存的过期和更新策略。一般会在数据发生更改的时，主动更新缓存中的数据或者移除对应的缓存。

   ![](..\..\笔记图片\8\59.png)

2. 缓存并发问题

   缓存过期后将尝试从后端数据库获取数据，这是一个看似合理的流程。但是，在高并发场景下，有可能多个请求并发的去从数据库获取数据，对后端数据库造成极大的冲击，甚至导致 “雪崩”现象。此外，当某个缓存key在被更新时，同时也可能被大量请求在获取，这也会导致一致性的问题。那如何避免类似问题呢？我们会想到类似“锁”的机制，在缓存更新或者过期的情况下，先尝试获取到锁，当更新或者从数据库获取完成后再释放锁，其他的请求只需要牺牲一定的等待时间，即可直接从缓存中继续获取数据。

   ![](..\..\笔记图片\8\60.png)

3. 缓存穿透问题

   缓存穿透在有些地方也称为“击穿”。很多朋友对缓存穿透的理解是：由于缓存故障或者缓存过期导致大量请求穿透到后端数据库服务器，从而对数据库造成巨大冲击。

   这其实是一种误解。真正的缓存穿透应该是这样的：

   在高并发场景下，如果某一个key被高并发访问，没有被命中，出于对容错性考虑，会尝试去从后端数据库中获取，从而导致了大量请求达到数据库，而当该key对应的数据本身就是空的情况下，这就导致数据库中并发的去执行了很多不必要的查询操作，从而导致巨大冲击和压力。

   可以通过下面的几种常用方式来避免缓存传统问题：

   1. 缓存空对象

      对查询结果为空的对象也进行缓存，如果是集合，可以缓存一个空的集合（非null），如果是缓存单个对象，可以通过字段标识来区分。这样避免请求穿透到后端数据库。同时，也需要保证缓存数据的时效性。这种方式实现起来成本较低，比较适合命中不高，但可能被频繁更新的数据。

   2. 单独过滤处理

      对所有可能对应数据为空的key进行统一的存放，并在请求前做拦截，这样避免请求穿透到后端数据库。这种方式实现起来相对复杂，比较适合命中不高，但是更新不频繁的数据。

   ![](..\..\笔记图片\8\61.png)

4. 缓存颠簸问题

   缓存的颠簸问题，有些地方可能被成为“缓存抖动”，可以看做是一种比“雪崩”更轻微的故障，但是也会在一段时间内对系统造成冲击和性能影响。一般是由于缓存节点故障导致。业内推荐的做法是通过一致性Hash算法来解决。这里不做过多阐述，可以参照其他章节

5. 缓存的雪崩现象

   缓存雪崩就是指由于缓存的原因，导致大量请求到达后端数据库，从而导致数据库崩溃，整个系统崩溃，发生灾难。导致这种现象的原因有很多种，上面提到的“缓存并发”，“缓存穿透”，“缓存颠簸”等问题，其实都可能会导致缓存雪崩现象发生。这些问题也可能会被恶意攻击者所利用。还有一种情况，例如某个时间点内，系统预加载的缓存周期性集中失效了，也可能会导致雪崩。为了避免这种周期性失效，可以通过设置不同的过期时间，来错开缓存过期，从而避免缓存集中失效。

   从应用架构角度，我们可以通过限流、降级、熔断等手段来降低影响，也可以通过多级缓存来避免这种灾难。

   此外，从整个研发体系流程的角度，应该加强压力测试，尽量模拟真实场景，尽早的暴露问题从而防范。

   ![](..\..\笔记图片\8\62.png)

#### 参考博文

https://www.imooc.com/article/20918

## 第13章 高并发之消息队列思路

### 13-1 高并发之消息队列-1

![](..\..\笔记图片\8\63.png)

消息被处理的过程相当于流程A被处理。我们这里以一个实际的模型来讨论下，比如用户下单成功时给用户发短信，如果没有这个消息队列，我们会选择同步调用发短信的接口，并等待短息发送成功，这时候假设短信接口实现出现问题了，或者短信调用端超时了，又或者短信发送达到上限了，我们是选择重试几次还是放弃，还是选择把这个放到数据库

过一段时间再看看呢，不管怎样，实现都很复杂。

我们可以将发短信这个请求放在消息队列里，消息队列按照一定的顺序挨个处理队列里的消息，当处理到发送短信的任务时，通知短信服务发送消息，如果出现之前出现的问题，那么把这个消息重新放到消息队列中。

### 13-2 高并发之消息队列-2

1. 消息队列的特性

![](..\..\笔记图片\8\64.png)

2. 为什么需要消息队列

   【生产】和【消费】的速度或稳定性等因素不一致

   参考博文：

   * https://zhuanlan.zhihu.com/p/99733519
   * https://blog.csdn.net/liitdar/article/details/87637492

3. 消息队列的使用场景

   ![](..\..\笔记图片\8\65.png)

   * **业务解耦**：

     成功完成了一个**异步解耦**的过程。短信发送时只要保证放到消息队列中就可以了，接着做后面的事情就行。一个事务只关心本质的流程，需要依赖其他事情但是不那么重要的时候，有通知即可，无需等待结果。每个成员不必受其他成员影响，可以更独立自主，只通过一个简单的容器来联系。

     对于我们的订单系统，订单最终支付成功之后可能需要给用户发送短信积分什么的，但其实这已经不是我们系统的核心流程了。如果外部系统速度偏慢（比如短信网关速度不好），那么主流程的时间会加长很多，用户肯定不希望点击支付过好几分钟才看到结果。那么我们只需要通知短信系统“我们支付成功了”，不一定非要等待它处理完成。

   * **最终一致性**：

     保证了**最终一致性**，通过在队列中存放任务保证它最终一定会执行。

     最终一致性指的是两个系统的状态保持一致，要么都成功，要么都失败。当然有个时间限制，理论上越快越好，但实际上在各种异常的情况下，可能会有一定延迟达到最终一致状态，但最后两个系统的状态是一样的。
     业界有一些为“最终一致性”而生的消息队列，如Notify（阿里）、QMQ（去哪儿）等，其设计初衷，就是为了交易系统中的高可靠通知。
     以一个银行的转账过程来理解最终一致性，转账的需求很简单，如果A系统扣钱成功，则B系统加钱一定成功。反之则一起回滚，像什么都没发生一样。
     然而，这个过程中存在很多可能的意外：

     1. A扣钱成功，调用B加钱接口失败。
     2. A扣钱成功，调用B加钱接口虽然成功，但获取最终结果时网络异常引起超时。
     3. A扣钱成功，B加钱失败，A想回滚扣的钱，但A机器down机。

     可见，想把这件看似简单的事真正做成，真的不那么容易。所有跨VM的一致性问题，从技术的角度讲通用的解决方案是：

     1. 强一致性，分布式事务，但落地太难且成本太高，后文会具体提到。
     2. 最终一致性，主要是用“记录”和“补偿”的方式。在做所有的不确定的事情之前，先把事情记录下来，然后去做不确定的事情，结果可能是：成功、失败或是不确定，“不确定”（例如超时等）可以等价为失败。成功就可以把记录的东西清理掉了，对于失败和不确定，可以依靠定时任务等方式把所有失败的事情重新搞一遍，直到成功为止。
        回到刚才的例子，系统在A扣钱成功的情况下，把要给B“通知”这件事记录在库里（为了保证最高的可靠性可以把通知B系统加钱和扣钱成功这两件事维护在一个本地事务里），通知成功则删除这条记录，通知失败或不确定则依靠定时任务补偿性地通知我们，直到我们把状态更新成正确的为止。

   * **广播**：

     消息队列的基本功能之一是进行广播。如果没有消息队列，每当一个新的业务方接入，我们都要联调一次新接口。有了消息队列，我们只需要关心消息是否送达了队列，至于谁希望订阅，是下游的事情，无疑极大地减少了开发和联调的工作量。

     **提速**。假设我们还需要发送邮件，有了消息队列就不需要同步等待，我们可以直接并行处理，而下单核心任务可以更快完成。增强业务系统的异步处理能力。甚至几乎不可能出现并发现象。

   * **削峰和流控**：

     不对于不需要实时处理的请求来说，当并发量特别大的时候，可以先在消息队列中作缓存，然后陆续发送给对应的服务去处理

     试想上下游对于事情的处理能力是不同的。比如，Web前端每秒承受上千万的请求，并不是什么神奇的事情，只需要加多一点机器，再搭建一些LVS负载均衡设备和Nginx等即可。但数据库的处理能力却十分有限，即使使用SSD加分库分表，单机的处理能力仍然在万级。由于成本的考虑，我们不能奢求数据库的机器数量追上前端。
     这种问题同样存在于系统和系统之间，如短信系统可能由于短板效应，速度卡在网关上（每秒几百次请求），跟前端的并发量不是一个数量级。但用户晚上个半分钟左右收到短信，一般是不会有太大问题的。如果没有消息队列，两个系统之间通过协商、滑动窗口等复杂的方案也不是说不能实现。但系统复杂性指数级增长，势必在上游或者下游做存储，并且要处理定时、拥塞等一系列问题。而且每当有处理能力有差距的时候，都需要单独开发一套逻辑来维护这套逻辑。所以，利用中间系统转储两个系统的通信内容，并在下游系统有能力处理这些消息的时候，再处理这些消息，是一套相对较通用的方式。

     总而言之，消息队列不是万能的。对于需要强事务保证而且延迟敏感的，RPC是优于消息队列的。
     对于一些无关痛痒，或者对于别人非常重要但是对于自己不是那么关心的事情，可以利用消息队列去做。
     支持最终一致性的消息队列，能够用来处理延迟不那么敏感的“分布式事务”场景，而且相对于笨重的分布式事务，可能是更优的处理方式。
     当上下游系统处理能力存在差距的时候，利用消息队列做一个通用的“漏斗”。在下游有能力处理的时候，再进行分发。
     如果下游有很多系统关心你的系统发出的通知的时候，果断地使用消息队列吧。

### 13-3 高并发之消息队列-3

![](..\..\笔记图片\8\66.png)

#### Kafka的介绍

Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据。
Kafka 有如下特性：

- 以时间复杂度为O(1)的方式提供消息持久化能力，即使对TB级以上数据也能保证常数时间复杂度的访问性能。
- 高吞吐率。即使在非常廉价的商用机器上也能做到单机支持每秒100K条以上消息的传输。
- 支持Kafka Server间的消息分区，及分布式消费，同时保证每个Partition内的消息顺序传输。
- 同时支持离线数据处理和实时数据处理。
- Scale out：支持在线水平扩展。

#### kafka的术语

- Broker：Kafka集群包含一个或多个服务器，这种服务器被称为broker。
- Topic：每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）
- Partition：Partition是物理上的概念，每个Topic包含一个或多个Partition。
- Producer：负责发布消息到Kafka broker。
- Consumer：消息消费者，向Kafka broker读取消息的客户端。
- Consumer Group:每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。

![](..\..\笔记图片\8\67.png)

下面来介绍RabbitMQ里的一些基本定义，主要如下：

* **RabbitMQ Server**：提供消息一条从Producer到Consumer的处理。
* **Exchange**：一边从发布者方接收消息，一边把消息推送到队列。
  producer只能将消息发送给exchange。而exchange负责将消息发送到queues。Procuder Publish的Message进入了exchange，exchange会根据routingKey处理接收到的消息，判断消息是应该推送到指定的队列还是是多个队列，或者是直接忽略消息。这些规则是通过交换机类型（exchange type）来定义的主要的type有direct,topic,headers,fanout。具体针对不同的场景使用不同的type。
  queue也是通过这个routing keys来做的绑定。交换机将会对绑定键（binding key）和路由键（routing key）进行精确匹配，从而确定消息该分发到哪个队列。
* **Queue**：消息队列。接收来自exchange的消息，然后再由consumer取出。exchange和queue可以一对一，也可以一对多，它们的关系通过routingKey来绑定。
* **Producer**：Client A & B,生产者，消息的来源,消息必须发送给exchange。而不是直接给queue
* **Consumer**：Client 1，2，3消费者，直接从queue中获取消息进行消费，而不是从exchange中获取消息进行消费。

## 第14章 高并发之应用拆分思路

### 14-1 高并发之应用拆分

一个系统拆分的案例：

![](..\..\笔记图片\8\68.png)

前面我们已经提到单个服务器再优化，它的处理能力都是有上限的，因此我们选择多扩容以及使用缓存和消息队列等对程序进行优化。

下面介绍另一种方法，随着项目需求完成越来越多，应用自然也会越来越大，架构师将一个应用整体拆分成多个应用。

#### 拆分原则

* 业务优先
  每个系统都会有多个模块，每个模块又有多个业务功能；按照业务边界进行切割，再对模块进行拆分。

* 循序渐进
  边拆分边测试，保证系统的正常运行。

* 兼顾技术：重构、分层
  不能为了分布式而分布式，拆分过程不仅是业务梳理也是代码重构的过程，根据技术进行分层来分配工作，ui对用户体验，熟悉C和C++对服务器，熟悉数据库的对数据库，做到术业有专攻，合适的人去做合适的事情。

* 可靠测试
  测试完毕后，才可进行下一步，每一步都要有足够的测试才可进行下一步，避免小错误引起蝴蝶效应。

#### 思考

![](..\..\笔记图片\8\69.png)

- 应用之间通信: RPC ( dubbo等)、消息队列
  消息队列通常用于传输数据包小但是数据量大，对实时性要求低的场景，比如下单后短信通知客户。而采用RPC要求实时性高，这里通常不会http或者service服务，原因是使用PRC调用service方法无感知，在配置好后和本地方法很像。
- 应用之间数据库设计:每个应用都有独立的数据库
  通常情况下，每个应用都有自己独立的数据库，如果共同使用的信息，可以考虑放在common中使用。
- 避免事务操作跨应用
  分布式事务是一个很消耗资源的问题，应用之间服务分开开发，能够保持相互独立。

#### 要实践微服务要解决4个问题

![](..\..\笔记图片\8\70.png)

![71](..\..\笔记图片\8\71.png)

1. 客户端如何访问这些服务
   API Gateway提供统一的服务入口，对前台透明，同时可以聚合后台的服务，提供安全过滤流控等api的管理功能
2. 服务之间是如何通信的
   异步的话使用消息队列，同步调用使用REST或者是RPC，Rest可以使用springboot，RPC通常使用Dubbo
   同步调用一致性强但是出现调用问题，REST一般基于http实现，能够跨客户端，同时对客户端没有更多的要求。
   RPC的传输协议更高效，安全也更加可控。特别是在一个公司内部如果有统一的开发规范和统一的框架，它的开发效率会更加明显。而异步消息在分布式系统中有特别广泛的应用，它既能减少调用服务之间的耦合，又能成为调用之间的缓冲，确保消息积压不会冲垮被调用方。
   同时保证调用方的用户的体验，继续干自己的活。付出的代价是一致性的减慢，需要接受数据的最终一致性
3. 如何实现如此多服务
   在微服务架构中一般每一服务都会拷贝进行负载均衡，服务如何相互感知，如何相互管理，这就是服务发现的问题了，一般都是进行服务注册信息的分布式管理。
4. 服务挂了该如何解决，有什么备份方案和应急处理机制
   分布式最大的特性就是网络是不可靠的，当系统是由一系列的调用链组成的时候，其中任何一个出问题都不至于影响到整个链路。
   相应的手段有：重试机制、应用的限流、熔断机制、负载均衡、系统降级

## 第15章 高并发之应用限流思路

### 15-1 高并发之应用限流-介绍

限流就是通过对并发访问/请求进行限速（限制总并发数、限制瞬时的并发数）或一个时间窗口内的请求进行限速（一个窗口内的平均速率），从而达到保护系统的目的。一般系统可以通过压测来预估能处理的峰值，一旦达到设定的峰值阀值，则可以拒绝服务（定向错误页或告知资源没有了）、排队或等待（例如：秒杀、评论、下单）、降级（返回默认数据）。

限流不能乱用，否则正常流量会出现一些奇怪的问题，从而导致用户抱怨。

![](..\..\笔记图片\8\78.png)


假设有130W到140W的数据插入到数据库中，如果没有做限流，数据库的主库会突然接收到130w的插入操作。

首先是网络上的开销，很可能直接把带宽占满，导致其他请求无法正常传输和处理，其次会是数据库的负载突然增高，导致无法处理某些数据库的操作，也有可能数据库没有足够的连接导致某些数据库插入查询失败；

还有一点就是现在数据库都做了主从设计，主数据库的数据还要同步给从库，这时瞬间插入了大量的数据，会带来从库和主库的延迟特别大，这时从库查询不准确的概率也会跟着提升。

如果我们放慢插入数据库的速度，这时插入数据库主库的速率会很正常，同步到从库也很正常。网络消耗也可以接收不会影响其他服务。

### 15-2 高并发之应用限流-相关算法

参考博文：https://blog.csdn.net/bingdianone/article/details/84392011

#### 1.计数器法

计数器法是限流算法里最简单也是最容易实现的一种算法。比如我们规定，对于A接口来说，我们1分钟的访问次数不能超过100个。那么我们可以这么做：在一开始的时候，我们可以设置一个计数器counter，每当一个请求过来的时候，counter就加1，如果counter的值大于100并且该请求与第一个 请求的间隔时间还在1分钟之内，那么说明请求数过多；如果该请求与第一个请求的间隔时间大于1分钟，且counter的值还在限流范围内，那么就重置 counter，具体算法的示意图如下：

![](..\..\笔记图片\8\79.png)

这个算法虽然简单，但是有一个十分致命的问题，那就是临界问题，如下图：

![](..\..\笔记图片\8\80.png)

从上图中我们可以看到，假设有一个恶意用户，他在0:59时，瞬间发送了100个请求，并且1:00又瞬间发送了100个请求，那么其实这个用户在 2秒时间内，瞬间发送了200个请求。我们刚才规定的是1分钟最多100个请求，也就是每秒钟最多1.7个请求，用户通过在时间窗口的重置节点处突发请求， 可以瞬间超过我们的速率限制。用户有可能通过算法的这个漏洞，瞬间压垮我们的应用。

聪明的朋友可能已经看出来了，刚才的问题其实是因为我们统计的精度太低。那么如何很好地处理这个问题呢？或者说，如何将临界问题的影响降低呢？我们可以看下面的滑动窗口算法。

#### 2.滑动窗口

滑动窗口，又称rolling window。为了解决计数器法统计精度太低的问题，引入了滑动窗口算法。如果学过TCP网络协议的话，那么一定对滑动窗口这个名词不会陌生。下面这张图，很好地解释了滑动窗口算法：

![](..\..\笔记图片\8\81.png)

在上图中，整个红色的矩形框表示一个时间窗口，在我们的例子中，一个时间窗口就是一分钟。然后我们将时间窗口进行划分，比如图中，我们就将滑动窗口划成了6格，所以每格代表的是10秒钟。每过10秒钟，我们的时间窗口就会往右滑动一格。每一个格子都有自己独立的计数器counter，比如当一个请求 在0:35秒的时候到达，那么0:30~0:39对应的counter就会加1。

那么滑动窗口怎么解决刚才的临界问题的呢？在上图中，0:59到达的100个请求会落在灰色的格子中，而1:00到达的请求会落在橘色的格子中。当时间到达1:00时，我们的窗口会往右移动一格，那么此时时间窗口内的总请求数量一共是200个，超过了限定的100个，所以此时能够检测出来触发了限流。

我再来回顾一下刚才的计数器算法，我们可以发现，计数器算法其实就是滑动窗口算法。只是它没有对时间窗口做进一步地划分，所以只有1格。

由此可见，当滑动窗口的格子划分的越多，那么滑动窗口的滚动就越平滑，限流的统计就会越精确。

#### 3.漏桶算法

漏桶算法，又称leaky bucket。为了理解漏桶算法，我们看一下对于该算法的示意图：

![](..\..\笔记图片\8\82.png)

从图中我们可以看到，整个算法其实十分简单。首先，我们有一个固定容量的桶，有水流进来，也有水流出去。对于流进来的水来说，我们无法预计一共有多少水会流进来，也无法预计水流的速度。但是对于流出去的水来说，这个桶可以固定水流出的速率。而且，当桶满了之后，多余的水将会溢出。

我们将算法中的水换成实际应用中的请求，我们可以看到漏桶算法天生就限制了请求的速度。当使用了漏桶算法，我们可以保证接口会以一个常速速率来处理请求。所以漏桶算法天生不会出现临界问题。

#### 4.令牌桶算法

令牌桶算法，又称token bucket。同样为了理解该算法，我们来看一下该算法的示意图：

![](..\..\笔记图片\8\83.png)

从图中我们可以看到，令牌桶算法比漏桶算法稍显复杂。首先，我们有一个固定容量的桶，桶里存放着令牌（token）。桶一开始是空的，token以 一个固定的速率r往桶里填充，直到达到桶的容量，多余的令牌将会被丢弃。每当一个请求过来时，就会尝试从桶里移除一个令牌，如果没有令牌的话，请求无法通过。

若仔细研究算法，我们会发现我们默认从桶里移除令牌是不需要耗费时间的。如果给移除令牌设置一个延时时间，那么实际上又采用了漏桶算法的思路。Google的Guava库下的SmoothWarmingUp类就采用了这个思路。

我们再来考虑一下临界问题的场景。在0:59秒的时候，由于桶内积满了100个token，所以这100个请求可以瞬间通过。但是由于token是以较低的速率填充的，所以在1:00的时候，桶内的token数量不可能达到100个，那么此时不可能再有100个请求通过。所以令牌桶算法可以很好地解决临界问题。下图比较了计数器（左）和令牌桶算法（右）在临界点的速率变化。我们可以看到虽然令牌桶算法允许突发速率，但是下一个突发速率必须要等桶内有足够的 token后才能发生：

![](..\..\笔记图片\8\84.jpg)

#### 小结

* 计数器 VS 滑动窗口：

  计数器算法是最简单的算法，可以看成是滑动窗口的低精度实现。滑动窗口由于需要存储多份的计数器（每一个格子存一份），所以滑动窗口在实现上需要更多的存储空间。也就是说，如果滑动窗口的精度越高，需要的存储空间就越大。

* 漏桶算法 VS 令牌桶算法：

  漏桶算法和令牌桶算法最明显的区别是令牌桶算法允许流量一定程度的突发。因为默认的令牌桶算法，取走token是不需要耗费时间的，也就是说，假设桶内有100个token时，那么可以瞬间允许100个请求通过。

  令牌桶算法由于实现简单，且允许某些流量的突发，对用户友好，所以被业界采用地较多。当然我们需要具体情况具体分析，只有最合适的算法，没有最优的算法

#### RateLimiter使用示例

Google开源工具包Guava提供了限流工具类RateLimiter，该类基于令牌桶算法(Token Bucket)来完成限流，非常易于使用。RateLimiter经常用于限制对一些物理资源或者逻辑资源的访问速率，它支持两种获取permits接口，一种是如果拿不到立刻返回false（tryAcquire()），一种会阻塞等待一段时间看能不能拿到（tryAcquire(long timeout, TimeUnit unit)）。

使用tryAcquire方法获取令牌的示例代码：

```java
@Slf4j
public class RateLimiterExample1 {
    /**
     * 每秒钟放入5个令牌，相当于每秒只允许执行5个请求
     */
    private static final RateLimiter RATE_LIMITER = RateLimiter.create(5);

    public static void main(String[] args) {
        // 模拟有100个请求
        for (int i = 0; i < 100; i++) {
            // 尝试从令牌桶中获取令牌，若获取不到则等待300毫秒看能不能获取到
            if (RATE_LIMITER.tryAcquire(300, TimeUnit.MILLISECONDS)) {
                // 获取成功，执行相应逻辑
                handle(i);
            }
        }
    }

    private static void handle(int i) {
        log.info("{}", i);
    }
}
12345678910111213141516171819202122
```

若想保证所有的请求都被执行，而不会被抛弃的话，可以选择使用acquire方法：

```java
@Slf4j
public class RateLimiterExample2 {
    /**
     * 每秒钟放入5个令牌，相当于每秒只允许执行5个请求
     */
    private static final RateLimiter RATE_LIMITER = RateLimiter.create(5);

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            // 从令牌桶中获取一个令牌，若没有获取到会阻塞直到获取到为止，所以所有的请求都会被执行
            RATE_LIMITER.acquire();
            // 获取成功，执行相应逻辑
            handle(i);
        }
    }

    private static void handle(int i) {
        log.info("{}", i);
    }
}
```

## 第16章 高并发之服务降级与服务熔断思路

### 16-1 高并发之服务降级与服务熔断思路

#### 服务降级

服务压力剧增的时候根据当前的业务情况及流量对一些服务和页面有策略的降级，以此缓解服务器的压力，以保证核心任务的进行。同时保证部分甚至大部分任务客户能得到正确的响应。也就是当前的请求处理不了了或者出错了，给一个默认的返回。

#### 服务熔断

在股票市场，熔断这个词大家都不陌生，是指当股指波幅达到某个点后，交易所为控制风险采取的暂停交易措施。相应的，服务熔断一般是指软件系统中，由于某些原因使得服务出现了过载现象，为防止造成整个系统故障，从而采用的一种保护措施，所以很多地方把熔断亦称为过载保护。

#### 降级分类

* 降级按照**是否自动化**可分为：**自动开关降级**和**人工开关降级**。

* 降级按照**功能**可分为：**读服务降级**、**写服务降级**。

* 降级按照处于的**系统层次**可分为：**多级降级**。

#### 自动降级分类

（1）超时降级：主要配置好超时时间和超时重试次数和机制，并使用异步机制探测回复情况

（2）失败次数降级：主要是一些不稳定的api，当失败调用次数达到一定阀值自动降级，同样要使用异步机制探测回复情况

（3）故障降级：比如要调用的远程服务挂掉了（网络故障、DNS故障、http服务返回错误的状态码、rpc服务抛出异常），则可以直接降级。降级后的处理方案有：默认值（比如库存服务挂了，返回默认现货）、兜底数据（比如广告挂了，返回提前准备好的一些静态页面）、缓存（之前暂存的一些缓存数据）

（4）限流降级

当我们去秒杀或者抢购一些限购商品时，此时可能会因为访问量太大而导致系统崩溃，此时开发者会使用限流来进行限制访问量，当达到限流阀值，后续请求会被降级；降级后的处理方案可以是：排队页面（将用户导流到排队页面等一会重试）、无货（直接告知用户没货了）、错误页（如活动太火爆了，稍后重试）。

#### 服务熔断和服务降级比较

##### 两者其实从有些角度看是有一定的类似性的：

1. 目的很一致，都是从可用性可靠性着想，为防止系统的整体缓慢甚至崩溃，采用的技术手段；
2. 最终表现类似，对于两者来说，最终让用户体验到的是某些功能暂时不可达或不可用；
3. 粒度一般都是服务级别，当然，业界也有不少更细粒度的做法，比如做到数据持久层（允许查询，不允许增删改）；
4. 自治性要求很高，熔断模式一般都是服务基于策略的自动触发，降级虽说可人工干预，但在微服务架构下，完全靠人显然不可能，开关预置、配置中心都是必要手段；

##### 而两者的区别也是明显的：

1. 触发原因不太一样，服务熔断一般是某个服务（下游服务）故障引起，而服务降级一般是从整体负荷考虑；
2. 管理目标的层次不太一样，熔断其实是一个框架级的处理，每个微服务都需要（无层级之分），而降级一般需要对业务有层级之分（比如降级一般是从最外围服务开始）
3. 实现方式不太一样

#### **服务降级要考虑的问题**

1.核心和非核心服务

2.是否支持降级，降级策略

3.业务放通的场景，策略

#### Hystrix

Hystrix，该库旨在通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。Hystrix具备拥有回退机制和断路器功能的线程和信号隔离，请求缓存和请求打包（request collapsing，即自动批处理，译者注），以及监控和配置等功能。

##### Hystrix能做什么？

1. 在通过第三方客户端访问（通常是通过网络）依赖服务出现高延迟或者失败时，为系统提供保护和控制
2. 在分布式系统中防止级联失败
3. 快速失败（Fail fast）同时能快速恢复
4. 提供失败回退（Fallback）和优雅的服务降级机制
5. 提供近实时的监控、报警和运维控制手段

##### Hystrix设计原则？

1. 防止单个依赖耗尽容器（例如 Tomcat）内所有用户线程
2. 降低系统负载，对无法及时处理的请求快速失败（fail fast）而不是排队
3. 提供失败回退，以在必要时让失效对用户透明化
4. 使用隔离机制（例如『舱壁』/『泳道』模式，熔断器模式等）降低依赖服务对整个系统的影响
5. 针对系统服务的度量、监控和报警，提供优化以满足近实时性的要求
6. 在 Hystrix 绝大部分需要动态调整配置并快速部署到所有应用方面，提供优化以满足快速恢复的要求
7. 能保护应用不受依赖服务的整个执行过程中失败的影响，而不仅仅是网络请求

##### Hystrix怎样实现目标？

1. 在一个单独的线程中通过 HystrixCommand 或 HystrixObservableCommand 对象包装所有外部系统（或依赖）的调用。
2. 调用超时比设置的阈值更长。虽然有默认值，但是大多数依赖自己配置的这些超时“属性”，所以每个依赖都略高于实测性能的99.5%。
3. 为每个依赖保持一个小的线程池；如果线程池满了，新来的请求会立即拒绝掉，而不是排队等候。
4. 测试成功、失败（客户端抛出异常）、超时和线程拒绝。
5. 当请求失败、被拒、超时或者短路时的性能反馈逻辑。
6. 准实时的监控指标和配置改变。

##### 开发文档

- [入门文档](https://github.com/Netflix/Hystrix/wiki/Getting-Started)
- [原理说明](https://github.com/Netflix/Hystrix/wiki/How-it-Works)
- [使用说明](https://github.com/Netflix/Hystrix/wiki/How-To-Use)
- [Javadoc](http://netflix.github.com/Hystrix/javadoc)

开源地址：https://github.com/Netflix/Hystrix

## 第17章 高并发之数据库切库分库分表思路

### 17-1 高并发之数据库切库分库分表

#### 1. 数据库瓶颈

![](..\..\笔记图片\8\84.png)

#### 2. 数据库切库

* 切库的基础和实际运用

  现在大型的系统在数据库层面大多采用了读写分离技术，就是一个主库，多个从库。主库主要负责数据更新和实时数据的查询，从库负责的就是非实时数据的查询。因为在实际情况下，数据库大多是读多写少的。而读取数据通常耗时比较长，占用服务器CPU时间也较多，从而影响用户体验。我们通常做法就是将查询从主库中抽取出来，从而多个从库使用负载均衡，减轻每个从库的查询压力。采用“读写分离”的目标就是减轻主库的压力，又可以把用户查询数据的请求分发到不同的从库上，把数据源动态的置入到程序中，让指定的程序选择连接操作主库还是连接从库进行操作。这里用到的技术主要是注解，SpringAOP等等。

  切库主要有两种方式：

  * 自定义注解完成数据库切库
  * 在项目中配置两个数据库连接，一个是主库连接，一个是从库连接。更新数据的时候使用主库连接，查询数据的时候使用从库连接。可以参考下面**3. 数据库支持多个数据源和分库**

* 自定义注解完成数据库切库

  具体实现可以参考：http://www.imooc.com/article/22556

#### 3. 数据库支持多个数据源和分库

* “多数据源（切库）”与“分库”的比较区别：

  它们都是底层是多个数据库在提供服务。

  分库是属于在微服务应用拆分的时候都有自己的数据库，而多数据源是在没有进行应用拆分的时候就已经分成两个库了，根据业务使用不同的代码连接不同的数据库。

  ![](..\..\笔记图片\8\85.png)

* 数据库支持多个数据源

  具体实现可以参考：http://www.imooc.com/article/22609

#### 4. 数据库分表

1. 什么时候考虑分表？

   当一个数据表很大，大到我们做了sql和索引优化之后，基本操作的速度还是影响使用，我们就必须考虑分表了。

2. 分表策略

   （1）横向分表（水平拆分）

   将表中不同的数据行按照一定规律分布到不同的数据库表中（这些表保存在同一个数据库中），这样来降低单表数据量，优化查询性能，其中表结构是一样的。

   （2）纵向分表（垂直拆分）

   一般根据数据的活跃度进行划分。比如对于博客系统，它对于作者标题之类变化频率慢的数据（冷数据），而博客的访问量，点赞数之类（活跃数据）

（3）mybatis分表插件shardbatis2.0

具体实现：[http://www.imooc.com/article/25256](利用mybatis插件实现数据库分表)

## 第18章 高并发之高可用手段介绍

### 18-1 高并发之高可用一些手段

![](..\..\笔记图片\8\86.png)

* elastic-job：https://gitee.com/elasticjob/elastic-job
* zookeeper：https://zookeeper.apache.org/
* apache curator：http://curator.apache.org/
* 监控报警机制：http://www.imooc.com/article/20891

## 第19章 课程总结

### 19-1 课程总结

![](..\..\笔记图片\8\87.png)

![88](..\..\笔记图片\8\88.png)

![89](..\..\笔记图片\8\89.png)

![90](..\..\笔记图片\8\90.png)

![91](..\..\笔记图片\8\91.png)