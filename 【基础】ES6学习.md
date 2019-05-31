

# ES6学习

## ES6项目构建

### 1.基础架构

![6-1](E:\markdown笔记\笔记图片\6-1.png)

### 2.任务自动化(gulp)

* [浅谈前端自动](https://www.cnblogs.com/kasmine/p/6436131.html)
* [gulp](https://www.gulpjs.com.cn/)

### 3.编译工具(babel、webpack)

* [babel](https://babel.bootcss.com/)

  Babel 是一个广泛使用的 ES6 转码器，可以将 ES6 代码转为 ES5 代码，从而在现有环境执行。这意味着，你可以用 ES6 的方式编写程序，又不用担心现有环境是否支持。

* [webpack](https://www.webpackjs.com/)

  WebPack可以看做是模块打包机：它做的事情是，分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（Scss，TypeScript等），并将其转换和打包为合适的格式供浏览器使用。

### 4.项目目录创建

* 创建下面三个文件夹

  ```
  es6
  |--app
  |--server
  |--task
  ```

* 进入app创建文件夹和文件

  ```
  es6
  |--app
  |--|--css
  |--|--js
  |--|--|--class
  |--|--|--|--test.js
  |--|--|--index.js
  |--|--views
  |--|--|--error.ejs
  |--|--|--index.ejs
  ```

* 进入server创建环境

  * [nodejs环境配置](https://www.cnblogs.com/goldlong/p/8027997.html)

    * 下载安装nodejs，安装成功的话在powershell输入`node -v`和`npm -v`
    * 在nodejs安装目录下新建两个文件夹node_cache和node_global，运行命令`npm config set prefix "D:\nodejs\node_global"`和`npm config set cache "D:\nodejs\node_cache"`，然后输入`npm config set registry=http://registry.npm.taobao.org `配置镜像站。
    * 输入命令npm config list 显示所有配置信息，我们关注一个配置文件
      C:\Users\Administrator\.npmrc。这里面存储了我们刚才设置的信息。
    * 设置node环境变量，新建NODE_PATH为D:\nodejs\node_global，然后设置path为%NODE_PATH%

  * 安装[express-generator](http://www.expressjs.com.cn/starter/generator.html)

    ```
    npm install express-generator -g
    ```

  * 然后在server目录下执行`express -e .`表示使用ejs模板引擎在当前目录

  * 然后在server目录执行`npm install`

* 进入task创建环境

  ```
  mkdir util
  touch util/args.js
  ```

* 进入es6文件下

  * 命令`npm init`，然后一直回车。这时项目下就会多一个package.json文件
  * 命令`touch .babelrc`
  * 命令`touch gulpfile.babel.js`

### 5.命令行处理（[git](https://github.com/depers/ES6_learn/commit/e55e2cb9ab6fb47a83e98e2cec7e9e24322c5a74)）

* 编辑tasks\util\args.js

* 编辑tasks\scripts.js

* 然后在命令行输入：

  ```
  npm install gulp gulp-if gulp-concat webpack webpack-stream vinyl-named gulp-livereload gulp-plumber gulp-rename gulp-uglify gulp-util yargs --save-dev
  ```

### 6.创建JS编译任务、创建模板、服务任务脚本（[git](https://github.com/depers/ES6_learn/commit/a759c079afa62d8991218175c66ec487a0b70f10))

* 继续编辑tasks\scripts.js
* 编辑tasks\pages.js，tasks\css.js，tasks\server.js

### 7.文件自动监听，项目构建测试（[git]())

* 编辑tasks\browser.js，tasks\clean.js，tasks\build.js

* 编辑tasks\default.js，该文件名会被gulp识别

* 编辑gulpfile.babel.js和.babelrc

* 然后输入`gulp`进行测试

* 文件自动监听输入`gulp --watch`，启动服务后在浏览器中输入http://localhost:3000/ 测试

* 此时我们还没有实现热更新的配置，需在server\app.js中配置

  ```
  app.use(express.static(path.join(__dirname, 'public')));
  // 一定要在上面这句下面添加这一句：
  app.use(require('connect-livereload')());
  ```

  配置完成后继续`gulp --watch`即可完成热更新。

## ES6语法

### 1.let和const

* 作用域的概念

  先看第一段代码：

  ```js
  function test(){
      for(let i = 1; i < 3; i++){
          console.log(i);
      }
      console.log(i);
  }
  
  test();
  ```

  结果：

  ![6-2](E:\markdown笔记\笔记图片\6-2.png)

  接下来看第二段代码：

  ```js
  function test(){
      for(var i = 1; i < 3; i++){
          console.log(i);
      }
      console.log(i);
  }
  
  test();
  ```

  结果：

  ![6-3](E:\markdown笔记\笔记图片\6-3.png)

  有上面的实验我们得到三点结论：

  **`{}`两个大括号之间的区域称为块作用域**

  **let声明的变量有块作用域的属性，只在块作用域内有效；var声明的变量在全局范围内有效**

  **在es5中使用严格模式一定要声明`"use strict";`，es6默认启动严格模式，即变量未声明不能使用**

* let命令

  * let声明的变量有块作用域的属性，只在块作用域内有效

  * 使用let声明变量时不能重复声明变量

    ```
    Module build failed: E:/Dome/js/es6/app/js/class/lesson1.js: Duplicate declaration "a"
    
       6 | 
       7 |     let a = 1;
    >  8 |     let a = 2;
         |         ^
       9 | }
    ```

  * 不存在变量提升

    `var`命令会发生“变量提升”现象，即变量可以在声明之前使用，值为`undefined`。

    * **let命令所声明的变量一定要在声明后使用，否则报错。**

    ```js
    // var 的情况
    console.log(foo); // 输出undefined
    var foo = 2;
    
    // let 的情况
    console.log(bar); // 报错ReferenceError
    let bar = 2;
    ```

  * 暂时性死区(**let命令所声明的变量一定要在声明后使用，否则报错**)

    在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

    * 如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在**声明之前**就使用这些变量，就会报错。

    * 变量x使用let命令声明，所以在声明之前，都属于x的“死区”，只要用到该变量就会报错。**typeof**运行时就会抛出一个ReferenceError。作为比较，如果一个变量根本没有被声明，使用typeof反而不会报错。

      ```js
      typeof x; // ReferenceError
      let x;
      ```

      ```js
      typeof undeclared_variable // "undefined"
      ```

* 块级作用域

  * 为什么需要块级作用域

    * 内层变量可能会覆盖外层变量
    * 用来计数的循环变量泄露为全局变量

  * ES6 的块级作用域

    * let实际上为 JavaScript 新增了块级作用域。let声明的变量有块作用域的属性，只在块作用域内有效

    * ES6 允许块级作用域的任意嵌套

      ```js
      {{{{{let insane = 'Hello World'}}}}};
      ```

    * 外层作用域无法读取内层作用域的变量

      ```js
      {{{{
        {let insane = 'Hello World'}
        console.log(insane); // 报错
      }}}};
      ```

    * 内层作用域可以定义外层作用域的同名变量

      ```js
      {{{{
        let insane = 'Hello World';
        {let insane = 'Hello World'}
      }}}};
      ```

    * 块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了

      ```js
      // IIFE 写法
      (function () {
        var tmp = ...;
        ...
      }());
      
      // 块级作用域写法
      {
        let tmp = ...;
        ...
      }
      ```

  * 

* const命令

  * const用来声明常量`const PI = 3.1415926;`

  * const声明的变量也有块作用域的属性

  * const声明的变量必须初始化

  * const声明对象，对象的指针没有变，但他的内容可以变

    ```
    function last(){
        const PI = 3.1415926;
        const k={
            a:1
        }
        k.b = 2; //可以修改
    
        console.log(PI,k);
    }
    // test();
    last();
    ```

### 2.解构赋值

* 什么是解构赋值

  左边一种结构，右边一种结构，左右赋值一一对应。

* 解构赋值的分类

  * 数组解构赋值

    * 左边一个数组，右边一个数组对应赋值。

        ```js
        {
            let a, b, rest;
            [a, b] = [1, 2];
            console.log(a, b);
        }
        ```

    * 另一种解构赋值，rest将会被赋值为一个数组。

        ``` js
        {
            let a, b, rest;
            [a, b, ...rest] = [1, 2, 3, 4, 5];
            console.log(a, b, rest);
        }
        ```

    * 默认值，看c变量，如果c没有默认值也没有赋值，则为undefined。

      ``` js
      {
          let a, b, c, rest;
          [a, b, c=3] = [1, 2];
          console.log(a, b, c);
      }
      ```

    * 使用场景1：变量的交换

        ```js
        {
            let a = 1;
            let b = 2;
            [a, b] = [b, a];
            console.log(a, b);
        }
        ```

    * 使用场景2：函数返回值

        ```js
        {
            function f(){
                return [1, 2];
            }
        
            let a, b;
            [a, b] = f();
            console.log(a, b);
        }
        ```

    * 使用场景3：选择性接收函数返回值

        ```js
        {
            function f(){
                return [1, 2, 3, 4, 5];
            }
        
            let a, b;
            [a,,,b] = f();
            console.log(a, b);
        }
        ```

    * 使用场景4：数组接收函数返回值

        ```js
        {
            function f(){
                return [1, 2, 3, 4, 5];
            }
        
            let a, b;
            [a,...b] = f();
            console.log(a, b);
        }
        ```

    * 使用场景5：选择性接收和数组接收混合使用函数返回值

        ```js
        {
            function f(){
                return [1, 2, 3, 4, 5];
            }
        
            let a, b;
            [a,,...b] = f();
            console.log(a, b);
        }
        ```

  * 对象解构赋值

    * 常见的对象解构赋值,赋值符号的两侧都是对象：

        ```js
        {
            let a, b;
            ({a, b} = {a:1, b:2});
            console.log(a, b);
        }
        ```

        ```js
        {
            let o = {p:12,q:true};
            let {p, q} = o;
            console.log(p, q);
        }
        ```

    * 默认值的处理：

        ```js
        {
            let {a=10, b=2} = {a:3}
            console.log(a, b);
        }
        ```

    * json对象的解构赋值：

        ```js
        {
            let metaData={
                title:'abc',
                test:[{
                    title:'test',
                    desc:'description'
                }]
            }
        
            let {title:esTitle, test:[{title:cnTitle}]} = metaData;
            console.log(esTitle, cnTitle);
        }
        ```

  * 字符串解构赋值

  * 布尔值解构赋值

  * 函数参数解构赋值

  * 数值解构赋值

### 3.正则扩展

* 正则新增特性
* 构造函数的变化
* 正则方法的扩展
* u修饰符
* y修饰符
* s修饰符

