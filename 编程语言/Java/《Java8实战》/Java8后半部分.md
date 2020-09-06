# Java8学习笔记

## 7.重构或测试

### 增加代码的灵活性

1. 通用模式一：有条件的延迟
   * 使用场景：频繁的查询一个对象的状态，只是为了传递参数、调用该对象的一个方法。可以考虑使用Lambda表达式
2. 环绕执行
   * 使用场景：业务代码千差万别，但是他们都有着同样的准备和清理阶段的逻辑。可以考虑使用Lamdba表达式实现

### 使用Java8重构面向对象的设计模式

* 访问者模式
  * 作用：分离程序的算法和它的操作对象
* 单例模式
  * 作用：一般用于限制类的实例化，仅生成一份对象
* 策略模式
  * 内容：
    * 一个代表某个算法的接口
    * 一个或多个该接口的具体实现
    * 一个或多个使用策略对象的客户
  * 使用Lambda表达式后：
    * 省去第二个内容，不需要声明新的类来实现不同的策略，直接通过传递Lamdba表达式就能达到同样的目的
* 模板方法
  * 使用场景：希望使用某个算法，但是需要对其中的某些行进行改进
  * 使用Lambda表达式后：
    * 创建算法框架，让具体的实现作为组件进行插入。
    * 插入的不同算法的组件可以通过Lamdba表达式或者方法引用的方式实现
* 观察者模式
  * 内容：某些事件发生时，如果一个对象（称为主题）需要自动的通知其他多个对象（称为观察者）
  * 具体实现：
    * 一个观察者接口
    * 声明多个实现观察者接口的观察者
    * 一个主题接口，包含注册观察者方法和通知观察者方法
    * 一个实现主题接口的Feed类
  * 使用Lamdba表达式后：
    * 具体实现中的第二步具体观察者类的实现由Lambda表达式实现
* 责任链模式
  * 作用：一种创建处理对象序列的通用方案。一个处理对象可能需要在完成一些工作之后，将结果传递给另一个处理对象，这个对象接着做一些工作，然后再转交给另一个对象，依次类推。
  * 具体实现：
    * 定义一个代表处理对象的抽象类
    * 创建多个继承抽象类的处理对象，实现相关方法
  * 使用Lambda表达式之后：
    * 使用函数式接口UnaryOperator<T>和andThen方法对其进行构造
* 工厂模式
  * 作用：无需向客户暴露实例化的逻辑就能完成对象的创建
  * 使用Lambda表达式之后：
    * 使用方法引用来创建无参构造函数创建对象引用

### 测试Lambda表达式
* 测试可见Lambda函数的行为
* 测试使用Lambda的方法的行为
* 测试非常复杂的Lambda表达式
* 高阶函数的测试

### 调试Lambda表达式
* 查看栈跟踪
  * Lambda表达式和栈追踪
    * Lambda表达式没有名字，他的栈追踪可能很难分析
    * 方法引用指向的是同一个类中声明的方法（方法的声明和方法引用的代码在同一个类中），他的名称就可以在栈追踪中显示
* 输出日志
  * 流提供的**peek**方法在分析Stream流水线时，能够将中间变量的值输出到日志中


## 8.默认方法

### 什么是默认方法

* 背景：Java8中的接口支持在声明方法的同时提供实现
* 实现方法：
  * 在接口内声明静态方法
  * 在接口内声明默认方法

* 默认方法就是在接口内声明且定义方法，以defualt修饰的方法
* 常见的默认方法：
  * Comparator.naturalOrder()（静态默认方法）
  * Collection.stream()
* 默认方法的目标用户：类库的设计者
* 引入默认方法的目的：让实现类自动的继承接口的一个默认实现
* 代码变更对Java程序的影响分为三种类型的兼容性：
  * 二进制代码的兼容性
  * 源代码级的兼容性
  * 函数行为的兼容性

* Java8中抽象类和接口之间的区别：
  1. 一个类只能继承一个抽象类，但是一个类可以实现多个接口
  2. 抽象类有抽象方法和非抽象方法，接口可以有自己自己方法的实现(默认方法和静态方法)，二者实现方法略有区别
  3. 接口只能有static、final修饰的变量，不能实例变量，而抽象类可以有实例变量
  4. 接口方法的默认修饰符是public，抽象方法可以有public、protected修饰符修饰(public可以修饰接口和方法签名；protect只能修饰接口签名和带有具体实现的方法签名，不能修饰方法声明)
  5. 从设计层面来说，抽象类是对类的抽象，是一种模板设计，而接口是对行为的抽象，是一种行为的规范

### 默认方法的使用模式

1. 可选方案   
  
   > 接口若有默认方法的实现，该接口的实现类就无需再去实现改方法了。
2. 行为的多继承
   > 一种让类从多个来源重用代码的能力
   * 实现步骤：
     * 定义精简的接口
     * 通过组合实现接口，来赋予实体类不同的功能

### 解决冲突的规则

* 冲突：一个类实现多个接口，而接口中的方法使用的却是相同的函数签名
* 解决冲突的三条规则：
   1. 类或父类中声明的方法的优先级高于任何默认方法
   2. 若无法依据第一条进行判断，优先选择同函数签名的方法中实现的最具体的那个接口的方法（子接口的优先级更高）
   3. 如果一、二规则都失效，则需要显式的覆盖和调用期望的方法
		```java
		public class C implemenets A, B {
			void hello(){
				B.super.hello();
			}		
		}
		```
	

## 9.用Optional取代null

### null带来的种种问题

1. null是错误之源
2. null使得代码冗余膨胀
3. null自身毫无意义
4. null破坏了Java的哲学
5. null在Java类型系统上开了个口子

### Optional类入门

* 引入Optional类的目的：帮助开发人员更好的设计出普适的API，而不是消除每一个null引用

* 应用Optional的几种模式：

  * 构建Optional对象

    1. 声明一个空的Optional

       ```java
       Optional<Car> optCar = Optional.empty();
       ```

    2. 依据一个非空值创建Optional

       ```
       Optional<Car> optCar = Optional.of(car);
       ```

    3. 可接受null的Optional

       ```java
       Optional<Car> optCar = Optional.ofNullable(car);
       ```

  * 使用map从Optional对象中提取和转换值

    与Stream中的map方法类似，map会将提供的函数应用于流的每个元素。

  * 使用flatMap链接Optional对象；

    map的操作结果是一个Optional<Optional<T>>类型的对象，使用flatMap将其处理为Optional<T>类型。该操作与Stream中的Optional相似；

  * 默认行为及解引用Optional对象

    * get()

      如变量存在，就将被Optional封装的值返回，否则就抛出NoSuchElementException异常

    * orElse(T other)

      如果有值则将其返回，否则返回一个默认值

    * orElseGet(Supplier<? extends T> other)

      如果有值则将其返回，否则返回一个由指定的Supplier接口生成的值

    * orElseThrow(Supplier<? extends X> exceptionSupplier)

      如果有值则将其返回，否则返回一个由指定的Supplier接口生成的异常

    * ifPresent(Consumer<? super T>)

      如果值存在，就执行使用该值的方法调用，否则什么都不做

  * 使用filter剔除特定的值

    * filter

      如果值存在并且满足提供的谓词，就返回包含该值的Optional对象，否则返回一个空的Optional对象

### Optional实战示例

1. 用Optional封装可能为null的值

   采用Optional.ofNullable方法：

   ```java
   Optional<Object> value = Optional.ofNullable(map.get("key"));
   ```

2. 基础类型的Optional

   * 基础类型的Optional有OptionalInt、OptionalLong、OptionalDouble
   * 不推荐使用基础类型的Optional，因为Optional不支持map、flatMap和filter操作

## 10.CompletableFuture:组合式异步编程

并发：是指两个或者多个事件在同一时间间隔内发生；

并行：是指两个或者多个事件在同一时刻发生；

### Future接口

* 功能：通过使用Future接口，你只需将耗时的操作封装在一个Callable对象中，再将其提交给ExecutorService即可；

* 局限性：

  * 只提供了检测异步计算是否结束，等待异步操作结束、获取计算结果这几个方法；
  * 不足以处理多个Future结果之间的联系

* CompletableFuture构建异步应用

  CompletableFuture类实现了Future接口，他主要提供了以下功能：

  * 提供异步操作
  * 流水线非阻塞实现
  * 响应式处理异步操作

### 使用CompletableFuture实现异步API

* 使用CompletableFuture实现异步操作

* 使用CompletableFuture的completeExceptionally方法将导致CompletableFuture内发生的问题异常抛出

  因为CompletableFuture是通过另一个线程去执行计算任务的，当计算过程抛出异常，则该线程会被杀死。从而导致**等待get方法返回值的客户端被永久阻塞**。

* 使用工厂方法supplyAsync方法创建CompletableFuture

  * 第一个参数：接受生产者Supplier
  * 第二个参数：指定执行生产者方法的执行线程

### 使用并行优化请求操作

1. 使用并行流对请求进行并行操作

   采用流行流处理并行操作时，因为其内部采用的是通用线程池，默认使用固定数目的线程。在面对较多的计算任务时，会出现阻塞，无法充分利用CPU内核的计算能力

2. 使用CompletableFuture发起异步请求

   * join操作：他与get不同的是join不会抛出任何检测到的异常
   * 在流水线处理流的过程中，新的CompletableFuture对象只有在前一个操作完全结束之后，才能创建

3. 为CompletableFuture使用定制的执行器

   * 实现集合并行计算的方法
     * 转化为并行流
     * 枚举出集合中的每一个元素，使用定制执行器的CompletableFuture
   * 实现集合并行计算的场景判断
     * 计算密集型操作，并且没有IO：推荐使用并行流
     * 并行的工作单元还涉及等待IO的操作：使用CompletableFuture灵活性更好

### 对多个异步任务进行流水线操作

涉及的工厂方法有：

* supplyAsync()：创建CompletableFuture对象
* thenApply()：对CompletableFuture对象的返回值进行操作
* thenCompose()：对两个异步操作进行流水线，第一个操作的完成后，将其结果作为第二个操作的参数（两个有依赖的CompletableFuture）
* thenCombine()：将两个异步操作的结果通过BiFunction作为第二个参数对其进行合并

### 响应CompletableFuture的completion事件

涉及的工厂方法：

* thenAccept()：在每一个CompletableFuture上注册一个操作，该操作会在CompletableFuture完成执行后使用它的返回值；
* allOf()：该方法接收一个由CompletableFuture构成的数组，返回一个CompletableFuture<void>对象；
* anyOf()：该方法接收一个CompletableFuture对象构成的数组，返回由第一个执行完毕的CompletableFuture对象的返回值构成的CompletableFuture<Object>   


## 11.新的日期和时间API

### LocalDate、LocalTime、Instant、Duration以及Period

* 使用LocalDate和LocalTime
  * LocalDate：管理年、月、星期、日
  * LocalTime：管理时、分、秒

* 合并日期和时间——LocalDateTime
  * LocalDateTime：同时表示时间和日期，单不带有时区信息

* 机器的时间和时间格式——Instant
  * 时间格式：对于计算机而言，建模时间最自然的格式是表示一个持续时间段上某个点的单一大整型数，也是新的java.time.Instant类对时间建模的方式
  * Java8中使用Instant对象来表示适用于计算机的时间

* 定义Duration或Period
  * Duration：主要用来以秒和纳秒衡量时间的长短
  * Period：主要用来以年、月、日的方式衡量时间的长短

* 注意
  * 日期和时间对象都是不可修改的
  * Java8也提供了创建可变对象的方法
  * 上面的五个类都实现了Temporal接口

### 操作、解析和格式化日期
* 创建可变对象的方法
  1. with
    * 第一个参数：TemporalField对象
    * 第二个参数：具体要修改的数值

  2. 使用TemporalAdjuster
    * TemporalAdjuster类提供了丰富的工厂方法
    * TemporalAdjuster类的函数式接口，我们可以把它看作是一个UnaryOperator<Temporal>
    * 如果要使用Lambda表达式定义TemporalAdjuster，他接受一个UnaryOperator<LocalDate>

* 打印输出及解析日期时间对象
  * DateTimeFormatter类
    * format()
      * 参数：
        * DateTimeFormatter预定义常量(BASIC_ISO_DATE、ISO_LOCAL_DATE)
    * ofPattern()
  * 更细粒度的控制
    * DateTimeFormatterBuilder类

### 处理不同的时区和历法
* ZoneId：时区信息对象
* ZoneDateTime：在DateTime基础上增加了ZoneId时区信息
* ZoneOffset：时区的偏差计算
* 别的日历系统
  * ChronoLocalDate：所有的日期时间类都实现了ChronoLocalDate接口。推荐使用LocalDate，尽量避免使用ChronoLocalDate

## 12.函数式编程思想

### 声明式编程

* 含义：（函数式编程的基石）是一种编程范式，与命令式编程相对立。它描述目标的性质，让计算机明白目标，而非流程。声明式编程不用告诉计算机问题领域，从而避免随之而来的副作用。而命令式编程则需要用算法来明确的指出每一步该怎么做。 

### 函数式编程

  * 目的：

    * 使用不相互影响的表达式，描述想要做什么，由系统来选择如何实现

    * 无副作用计算

      > 副作用是指函数的效果已经超出了函数自身的范畴。体现在编码中就是函数是否对参数进行了修改。

  * 概念：

    * 一种使用函数进行编程的方式。
    *  它将电脑运算视为函数的计算。函数编程语言最重要的基础是λ演算（lambda calculus），而且λ演算的函数可以接受函数当作输入（参数）和输出（返回值） 
    * 如果程序有一定的副作用，不过该副作用不会为其他的调用者感知或者调用者完全不在意这些副作用，因为这对他完全没有影响（也就是说存在可变部分）

  * 编程准则

    1. “函数式”的函数或方法都只能修改本地变量
    2. “函数式”的函数或方法引用的对象都应该是不可修改的对象
    3. “函数式”的函数或方法期望的所有字段都为final类型
    4. “函数式”的函数或方法的所有引用类型字段都指向不可变对象
    5. “函数式”的函数或方法不应该抛出任何异常

  * 引用透明性

    * 定义：如果一个函数只要传递同样的参数值，总是能返回同样的结果，那这个函数就是引用透明的。

### 递归和迭代

* 为什么在纯粹的函数式编程语言中通常不包含像while或者for这样的迭代构造器 ？

  因为这种类型的构造器经常隐藏着陷阱，诱使你修改对象。

* 在函数式编程语言中，如何消除while或者for这样的迭代构造器 ？

  在程序中使用无需修改的递归重写，通过这种方式避免使用迭代。

* 递归操作往往效率较低，有什么改进方法吗？

  采用**尾调优化（tail-call optimization）**,通过该方法实现编译器优化。传统的递归运算中会每一次递归计算都要在内存中创建一个栈帧，而尾调优化实现了编译器对某个栈帧的复用。

  

