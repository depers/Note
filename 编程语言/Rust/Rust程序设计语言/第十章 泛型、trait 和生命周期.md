# 第十章 泛型、trait 和生命周期

Created: November 8, 2021 9:08 AM
Created by: Anonymous
Last edited time: November 14, 2021 10:54 AM

# 10-1 泛型数据类型

## 使用函数来减少重复

### 1. 使用main函数

如果我们要分别查找两个数组中的最大值，我们直接main函数中编写代码的话，如下所示：

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```

运行结果：

```rust
The largest number is 100
The largest number is 6000
```

### 2. 将重复代码整合到一个函数中

从main函数中直接执行逻辑到抽象到一个函数中，这里经历了如下几步：

1. 找出重复代码。
2. 将重复代码提取到了一个函数中，并在函数签名中指定了代码中的输入和返回值。
3. 将重复代码的两个实例，改为调用函数。

```rust
fn largest(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
   assert_eq!(result, 100);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
   assert_eq!(result, 6000);
}
```

## 在函数定义中使用泛型

下面的这段代码，largest_i32和largest_char两个函数的区别只是入参不一样，函数的执行逻辑都是一样的。

```rust
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);
   assert_eq!(result, 100);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
   assert_eq!(result, 'y');
}
```

通过使用泛型之后，我们可以将这两个函数合二为一：

```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}

```

运行输出：

```rust
error[E0369]: binary operation `>` cannot be applied to type `&T`
  --> src\bin\GenericTypeExample.rs:45:17
   |
45 |         if item > largest {
   |            ---- ^ ------- T
   |            |
   |            &T
   |
help: consider restricting type parameter `T`
   |
41 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {
   |             ^^^^^^^^^^^^^^^^^^^^^^
```

这里我们可以先忽略这段报错。

因为两者函数体的代码是一样的，我们可以定义一个函数，再引进泛型参数来消除这种重复。也就是说**泛型可以用来消除函数体代码重复**。

## 结构体中的泛型

- 特征
    1. 必须在结构体名称后面的尖括号中声明泛型参数的名称。
    2. 在结构体定义中可以指定具体数据类型的位置使用泛型类型。
    3. 结构体具体属性字段只能使用尖括号中声明的泛型类型。
- 一个泛型参数
  
    ```rust
    struct Point<T> {
        x: T,
        y: T,
    }
    
    fn main() {
        let integer = Point { x: 5, y: 10 };
        let float = Point { x: 1.0, y: 4.0 };
    }
    ```
    
- 多个泛型参数
  
    ```rust
    struct Point<T, U> {
        x: T,
        y: U,
    }
    
    fn main() {
        let both_integer = Point { x: 5, y: 10 };
        let both_float = Point { x: 1.0, y: 4.0 };
        let integer_and_float = Point { x: 5, y: 4.0 };
    }
    ```
    

## 枚举定义中的泛型

- Option<T>枚举
  
    ```rust
    enum Option<T> {
        Some(T),
        None,
    }
    ```
    
- Result<T>枚举
  
    ```rust
    enum Result<T, E> {
        Ok(T),
        Err(E),
    }
    ```
    
- 自定义枚举
  
    ```rust
    fn main() {
        let a = Cuboid::height(1);
        let b = Cuboid::width(2);
        let c = Cuboid::length(3);
    }
    
    enum Cuboid<T> {
        length(T),
        width(T),
        height(T)
    }
    ```
    

## 方法定义中的函数

我们可以为结构体和枚举实现方法时，一样也可使用泛型。

- 结构体方法使用泛型
  
    ```rust
    fn main() {
        let p = Point{x: 5, y: 6};
        println!("p.x = {}", p.x());
        println!("p = {:?}", p);
    }
    
    #[derive(Debug)]
    struct Point<T> {
        x: T,
        y: T,
    }
    
    impl<T> Point<T> {
    
        fn x(&self) -> &T{
            &self.x
        }
    }
    ```
    
    运行输出：
    
    ```rust
    p.x = 5
    p = Point { x: 5, y: 6 }
    ```
    
- 枚举方法使用泛型
  
    ```rust
    fn main() {
        let m = Message::Content(String::from("hello enum generic"));
        println!("message content = {}", m.get())
    }
    
    #[derive(Debug)]
    enum Message<T> {
        Content(T)
    }
    
    impl<T> Message<T> {
        fn get(&self) -> &T{
            match self {
                Message::Content(s) => s,
                _ => panic!("error")
            }
        }
    }
    ```
    
    运行输出：
    
    ```rust
    message content = hello enum generic
    ```
    
- 为拥有泛型参数的结构体构建**具体类型**的impl块
  
    在下面这段代码中，我们为结构体实现了具体类型的`impl`块，使用`u32`类型的`Point`实例才能调用。
    
    ```rust
    use num::integer::Roots;
    
    fn main() {
        let p = Point{x: 5, y: 6};
        println!("p distance from origin = {}", p.distance_from_origin());
        println!("p = {:?}", p);
    }
    
    #[derive(Debug)]
    struct Point<T> {
        x: T,
        y: T,
    }
    
    impl<T> Point<T> {
    
        fn x(&self) -> &T{
            &self.x
        }
    }
    
    impl Point<u32> {
        fn distance_from_origin(&self) -> u32{
            (self.x.pow(2) + self.y.pow(2)).sqrt()
        }
    
    }
    ```
    
    运行输出：
    
    ```rust
    p distance from origin = 7
    p = Point { x: 5, y: 6 }
    ```
    
- 结构体定义中的泛型类型参数并不总是与结构体方法签名中使用的泛型是同一类型
  
    在下面这段代码中，我们混合交换了两个结构体的属性，新的结构体的属性使用了新的类型。
    
    ```rust
    struct Point<T, U> {
        x: T,
        y: U,
    }
    
    impl<T, U> Point<T, U> {
        fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
            Point {
                x: self.x,
                y: other.y,
            }
        }
    }
    
    fn main() {
        let p1 = Point { x: 5, y: 10.4 };
        let p2 = Point { x: "Hello", y: 'c'};
    
        let p3 = p1.mixup(p2);
    
        println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
    }
    ```
    
    运行输出：
    
    ```rust
    p3.x = 5, p3.y = c
    ```
    

## 泛型代码的性能

Rust 实现了泛型，使得使用泛型类型参数的代码相比使用具体类型并没有任何速度上的损失。

Rust 通过在编译时进行泛型代码的 **单态化**（*monomorphization*）来保证效率。单态化是一个通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程。

# 10.2 trait：定义共享的行为

- *trait* 告诉 Rust 编译器某个特定类型拥有可能与其他类型共享的功能。可以通过 trait 以一种抽象的方式定义共享的行为。
- 可以使用 *trait bounds* 指定泛型是任何拥有特定行为的类型。
- 通过trait和trait bound，我们获取了如下优势：
    1. 使用泛型类型参数来减少重复，并仍然能够向编译器明确指定泛型类型需要拥有哪些行为;
    2. 向编译器提供了 trait bound 信息，它就可以检查代码中所用到的具体类型是否提供了正确的行为；

## 定义trait

注意：*trait* 类似于其他语言中的常被称为 **接口**（*interfaces*）的功能，虽然有一些不同。

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

- 特征
    1. trait 体中可以有多个方法：一行一个方法签名且都以分号结尾。
    2. 将 `pub` 置于 `trait` 之前，公有 trait 使得其他 crate 可以实现它。

## 为类型实现trait

在下面的这段代码中，我们分别为结构体`NewsArticle` 和`Tweet`实现了`Summary`trait。

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {

    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

fn main() {

    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize())
}
```

运行输出：

```rust
1 new tweet: horse_ebooks: of course, as you probably already know, people
```

**注意：**

- 实现 trait 时需要注意的一个限制是，只有当 trait 或者要实现 trait 的类型位于 crate 的本地作用域时，才能为该类型实现 trait。
- 不能为外部类型实现外部 trait。例如，不能在 `aggregator` crate 中为 `Vec<T>` 实现 `Display` trait。这是因为 `Display` 和 `Vec<T>` 都定义于标准库中，它们并不位于 `aggregator` crate 本地作用域中。
- 上面的这个限制称为 相干性（coherence）或是孤儿原则。其目的就是确保其他人写的代码不会破坏你的代码。

## 默认实现

除了实现trait定义的方法外，在trait中的方法还可以有自己的默认实现的方法，这样就不用实现该trait的类型每次都要重载这个方法。

重载一个默认实现的语法与实现没有默认实现的 trait 方法的语法一样的。

```rust
pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }

}

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```

运行输出：

```rust
1 new tweet: (Read more from @horse_ebooks...)
```

## trait作为参数

如下面的`notify`方法所示，我们可以函数参数中声明该函数接收的参数类型，从而控制函数参数的类型。

```rust
pub trait Summary {

    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}

//***********************************************************
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize())
}
//***********************************************************

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    };

    notify(tweet);
}
```

运行输出：

```rust
Breaking news! (Read more from @horse_ebooks...)
```

### 1. impl Trait语法

一个函数参数：

```rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize())
}
```

多个函数参数：

```rust
pub fn notify(item1: impl Summary, item2: impl Summary) {
```

### 2. Trait Bound语法

这里其实是泛型的trait bound。

一个函数参数：

```rust
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize())
}
```

多个函数参数：

```rust
pub fn notify<T: Summary>(item1: T, item2: T) {
```

### 3. 通过`+`指定多个trait bound

- impl Trait语法
  
    ```rust
    pub fn notify(item: impl Summary + Display) {
    ```
    
- trait bound语法
  
    ```rust
    pub fn notify<T: Summary + Display>(item: T) {
    ```
    

### 4. 通过where简化 trait bound

使用过多的trait bound语法也有缺点，这样会使函数的签名难以阅读。这里可以在函数签名之后的 `where` 从句中指定 trait bound 的语法

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
```

使用where从句后：

```rust
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

## 返回实现了trait的类型

在返回值中使用 `impl Trait` 语法，来返回实现了某个 trait 的类型，进而控制返回值的类型。

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
```

### 使用trait bounds来修复 largest函数

这里我们使用trait bound语法来对之前的`largest`函数进行修改。

```rust
fn largest<T: PartialOrd>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

运行输出：

```rust
error[E0508]: cannot move out of type `[T]`, a non-copy slice
   --> src\bin\TraitExample.rs:104:23
    |
104 |     let mut largest = list[0];
    |                       ^^^^^^^
    |                       |
    |                       cannot move out of here
    |                       move occurs because `list[_]` has type `T`, which does not implement the `Copy` trait
    |                       help: consider borrowing here: `&list[0]`
```

上面这段程序，根据之前在所有权那一节中所学，list[0]如果没有实现`Copy` trait，直接赋值给largest，那么list[0]会直接失效，所以这里报错。所以这里对代码做如下修改：

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

运行输出：

```rust
The largest number is 100
The largest char is y
```

## 使用 trait bound 有条件地实现方法

通过使用带有 trait bound 的泛型参数的 `impl` 块，可以有条件地只为那些实现了特定 trait 的类型实现方法，如下图的`cmp_display`。

```rust
struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self{
        Self {
            x,
            y
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {

    fn cmp_display(&self) {
        if self.x >= self.y{
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }

    }
}

fn main() {
    let a = Pair::new(1,2);
    a.cmp_display();

    let b = Pair{x: 3, y: 4};
    b.cmp_display();
}
```

运行输出：

```rust
The largest member is y = 2
The largest member is y = 4
```

### 覆盖实现

可以对任何实现了特定 trait 的类型有条件地实现 trait。对任何满足特定 trait bound 的类型实现 trait 被称为 *blanket implementations。*

例如，标准库为任何实现了 `Display` trait 的类型实现了 `ToString` trait。这个 `impl` 块看起来像这样：

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

可以对任何实现了 `Display` trait 的类型调用由 `ToString` 定义的 `to_string` 方法。例如，可以将整型转换为对应的 `String` 值，因为整型实现了 `Display`：

```rust
let s = 3.to_string();
```

# 10.3 生命周期与引用有效性

Rust 中的每一个引用都有其 **生命周期**（*lifetime），*也就是**引用保持有效的作用域**。但是有时候编译器也无法计算出参数的声明周期时，Rust 就需要我们使用泛型生命周期参数来注明他们的关系，这样就能确保运行时实际使用的引用绝对是有效的。

## 生命周期避免的垂直引用

### 1. 垂直引用

垂直引用：就是一个引用指向的内存已经被销毁。

```rust
fn main() {
    dangling_reference()
}

fn dangling_reference() {
    let r;
    {
        let x = 5;
        r = &x;
    } // x离开了作用域，从这里x指向的内存就被销毁了

    println!("r: {}", r);
}
```

运行输出：

```rust
error[E0597]: `x` does not live long enough
  --> src\bin\LifetimeExample.rs:10:13
   |
10 |         r = &x;
   |             ^^ borrowed value does not live long enough
11 |     }
   |     - `x` dropped here while still borrowed
12 | 
13 |     println!("r: {}", r);
   |                       - borrow later used here
```

从运行输出上就可以看出，说`x`存活的时间太短了。作用域越大，我们说生命周期越长，存活的时间越长。Rust是通过 `借用检查器`控制上面的生命周期逻辑的。

### 2. 借用检查器

Rust 编译器有一个 **借用检查器**（*borrow checker*），它比较作用域来确保所有的借用都是有效的。

错误的例子：

```rust
{
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}}                        // ---------+
```

变量r的生命周期标记为'a，变量x的生命周期标记为'b。因为x的生命周期小于r的生命周期，被引用者的生命周期`小于`引用者的生命周期并`早于`引用者结束了生命周期，所以这段程序会报错。

正确的例子：

```rust
{
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```

在上图中，被引用者`x`的生命周期大于引用者`r`的生命周期，所以这段程序是正确的。

## 函数中的泛型生命周期

让我们来编写一个返回两个字符串 slice 中较长者的函数。这个函数获取两个字符串 slice 并返回一个字符串 slice。

```rust
fn longest(x: &str, y: &str) -> &str{
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(&string1, string2);
    println!("The longest string is {}", result);
}
```

运行输出：

```rust
18 | fn longest(x: &str, y: &str) -> &str{
   |               ----     ----     ^ expected named lifetime parameter
   |
   = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
   |
18 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str{
   |           ^^^^    ^^^^^^^     ^^^^^^^     ^^^
```

提示文本揭示了返回值需要一个**泛型生命周期参数**，因为 Rust 并不知道将要返回的引用是指向 `x` 或 `y`。

**因为Rust需要根据函数入参的生命周期，进而控制返回参数的生命周期，保证引用的有效性**。这里Rust或者是这个函数的开发人员都不清楚x和y的生命周期，所以这里会报错。所以就引出了生命周期注解来解决这个问题。

## 生命周期注解语法

生命周期注解并不改变任何引用的生命周期的长短。

生命周期注解描述了多个引用生命周期相互的关系，而不影响其生命周期。

具体语法：

- 生命周期参数名称必须以撇号（`'`）开头，其名称通常全是小写，类似于泛型其名称非常短。
- `'a` 是大多数人默认使用的名称。生命周期参数注解位于引用的 `&` 之后，并有一个空格来将引用类型与生命周期注解分隔开。
- 例如：
  
    ```rust
    &i32        // 引用
    &'a i32     // 带有显式生命周期的引用
    &'a mut i32 // 带有显式生命周期的可变引用
    ```
    

作用：标记参数引用的生命周期。例如如果函数有一个生命周期 `'a` 的 `i32` 的引用的参数 `first`。还有另一个同样是生命周期 `'a` 的 `i32` 的引用的参数 `second`。这两个生命周期注解意味着引用 `first` 和 `second` 必须与这泛型生命周期存在得一样久。

## 函数签名中的生命周期注解

加生命周期注解的原因：函数被外部代码使用时，Rust无法分析函数入参引用和返回值引用的生命周期。

函数返回值引用的使用生命周期注解的控制原则：函数返回的引用的生命周期与传入该函数的引用的生命周期的**较小者**一致。如果不这么做的话，较小者离开作用域被销毁，返回值引用内存地址就会出现垂直引用。

接下来看两个例子。

第一个例子：返回值引用的生命周期是正确的

这里result引用的生命周期是string2的，内存指向却是string1的，但是在使用result的时候，并没有离开string2的作用域。

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str{
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

运行输出：

```rust
The longest string is long string is long
```

第二个例子：返回值引用的生命周期是错误的

这里result引用的生命周期是string2的，内存指向却是string1的，但是在使用result的时候，并且离开了string2的作用域，所以报错了。

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str{
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("long string is long");

    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }

    println!("The longest string is {}", result);
}
```

运行输出：

```rust
error[E0597]: `string2` does not live long enough
  --> src\bin\LifetimeExample.rs:84:44
   |
84 |         result = longest(string1.as_str(), string2.as_str());
   |                                            ^^^^^^^ borrowed value does not live long enough
85 |     }
   |     - `string2` dropped here while still borrowed
86 | 
87 |     println!("The longest string is {}", result);
   |                                          ------ borrow later used here
```

## 深入理解生命周期

### 1. 函数返回值的生命周期可以指定成唯一的一个入参的

这里返回值的生命周期和入参x的生命周期同，因为返回值就是x，这样是ok的。

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

### 2. 函数入参没有标明生命周期注解，返回值却有，肯定会产生垂直引用

下面这段代码result变量返回后就离开了作用域，就会产生一垂直引用。解决方法就是返回拥有所有权的String，而不是字符串slice。

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

## 结构体定义中的生命周期注解

这里就要求结构体中的`part`的属性`first_sentence`引用存在的生命周期要大于`ImportantExcept`的实例。

```rust
#[derive(Debug)]
struct ImportantExcept<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me IsHmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");

    let i = ImportantExcept{part: first_sentence};
    println!("i = {:?}", i)
}
```

运行输出：

```rust
i = ImportantExcept { part: "Call me IsHmael" }
```

## 生命周期省略（Lifetime Elision）

下面的这段函数是取一个字符串中的第一个遇到空格的片段。函数的入参和返回值都是引用，为什么没有声明他的生命周期呢？

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

这是因为 借用检查器可以自己推断出生命周期，而不需要程序员显式的增加注解。被编码进 Rust 引用分析的模式被称为 **生命周期省略规则**（*lifetime elision rules*）。

省略规则并不提供完整的推断：如果 Rust 在明确遵守这些规则的前提下变量的生命周期仍然是模棱两可的话，它不会猜测剩余引用的生命周期应该是什么。在这种情况，编译器会给出一个错误，这可以通过增加对应引用之间相联系的生命周期注解来解决。

函数或方法的参数的生命周期被称为 **输入生命周期**（*input lifetimes*），而返回值的生命周期被称为 **输出生命周期**（*output lifetimes*）。

编译器采用三条规则来判断函数签名（包含方法）引用何时不需要明确的注解：

1. 【适用输入生命周期】每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：`fn foo<'a>(x: &'a i32)`，有两个引用参数的函数有两个不同的生命周期参数，`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。
2. 【适用输出生命周期】如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`。
3. 【适用方法的输出生命周期】如果方法有多个输入生命周期参数并且其中一个参数是 `&self` 或 `&mut self`，说明是个对象的方法(method)(译者注： 这里涉及rust的面向对象参见17章), 那么所有输出生命周期参数被赋予 `self` 的生命周期。也就是对于方法而言，输出的生命周期和其实例同。

## 方法定义中的生命周期注解

方法的声明周期需要在`impl` 关键字之后和在结构体名称之后声明。

方法`level`不涉及引用，方法`announce_and_return_part`根据第三条生命周期注解忽略原则可以省略。

```rust
impl <'a> ImportantExcept<'a> {
    fn level(&self) ->i32 {
        2
    }

    fn announce_and_return_part(&self, announcement: & str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }

}
```

## 静态生命周期

`'static`，其生命周期**能够**存活于整个程序期间。所有的字符串字面值都拥有 `'static` 生命周期，我们也可以选择像下面这样标注出来：

```rust
fn main() {
    let s: &'static str = "I have a static lifetime";
}
```

## 结合泛型类型参数、trait bounds 和生命周期

在同一函数中指定泛型类型参数、trait bounds 和生命周期的语法如下：

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T> (x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement！{}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }

}
```

- 生命周期注解和泛型参数都放在函数名后面的尖括号中
- 函数参数`x`，`y`和返回参数都标注的生命周期注解
- 函数参数`ann`则使用了trait bounds语法