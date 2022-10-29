# GCC的使用

## GCC的编译过程

![](../../笔记图片/30-GCC的使用/GCC_CompilationProcess.png)

GCC 将一个 C/C + + 程序编译成可执行文件，步骤如上图所示。例如，`gcc-o hello.exe hello.c`的执行步骤如下:

1. 预处理：通过 GNU C 预处理器 (cpp.exe)，包括头文件 (#include) 并扩展宏 (#define)

   ```
   > cpp hello.c > hello.i
   ```

   生成的中间文件“hello.i”包含扩展的源代码。

2. 编译：编译器将预处理的源代码编译成特定处理器的汇编代码

   ```
   > gcc -S hello.i
   ```

   `-S` 选项指定生成汇编代码，而不是目标代码。 生成的程序集文件是“hello.s”。

3. 汇编：汇编程序（as.exe）将汇编代码转换为目标文件“hello.o”中的机器码

   ```
   > as -o hello.o hello.s
   ```

4. 链接器：最后，链接器（ld.exe）将目标代码与库代码链接起来，生成一个可执行文件“hello.exe”

   ```
   > ld -o hello.exe hello.o ...libraries...
   ```

   这一步我试了好久，就是编译不过，醉了，后面再说吧。

## GCC的参数

* `-E`：只预编译
* `-S`：只编译
* `-c`：编译和汇编（装配），但不链接
* `-o <file>`：将输出放入file文件中
* `-v`：可以通过启用 -v (Verbose)选项查看详细的编译过程

