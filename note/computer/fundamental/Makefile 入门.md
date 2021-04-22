## 简介

make 是一个工具，它包含有许多的内容，我们这里只讨论最简单的用法.
这个工具使用一个 make 命令来编译源代码 而这个命令将会在当前目录下自动寻找一个叫做
Makefile (或者 makefile 或者 make) 的无类型文档 根据里面的内容生成相应的文件

简单起见 我们只讨论用 gcc (MinGW) 来使用 make 工具

无论是 C 还是 CPP 源文件 编译器首先要把它编译 (compile) 成中间代码文件
这种文件在 Windows 下的扩展名是 .obj UNIX 下则是 .o 即 Object File
然后再把一个或多个 Object File 链接 (link) 成可执行文件，win 下的扩展名是.exe UNIX 下则是.out

## gcc 简介

gcc 是 Linux 平台下最重要的开发工具 它是 GNU 的 C 和 CPP 编译器 其基本用法为：

```bash
$ gcc [options] [filenames] # 用于 C 源文件
$ g++ [options] [filenames] # 用于 CPP 源文件
```

而 gcc 的 win 下版本称为 MinGW，它所使用的命令基本与 gcc 相同，我们只讨论 gcc 最简单常用的几种命令。

比方我们用 CPP 写了一个最简单的 Hello World 源文件，保存为到 Hello.cpp，最简单的编译方法是不指定任何编译选项：

```bash
$ g++ Hello.cpp
```

它会为目标程序生成默认的文件名 a.exe

我们可用 -o 编译选项来为将产生的可执行文件指定一个文件名来代替 a.exe
例如，将上述名为 Hello.cpp 的 CPP 源文件编译为名叫 Hello.exe 的可执行文件
需要输入如下命令

```bash
$ g++ Hello.cpp -o Hello.exe
```

可执行文件的扩展名可以省略 故可简写为

```bash
$ g++ Hello.cpp -o Hello
```

- `c` 选项告诉 gcc 仅把源代码编译为目标代码 (.obj 文件) 而跳过汇编和链接的步骤
比方说 将上述 Hello.cpp 文件编译为 Hello.obj 文件可以使用如下命令

```bash
$ g++ -c Hello.cpp -o Hello.obj
```

然后 读者可能会想到提供一个命令列表，让每个使用你的源代码的人都可以通过一个简单的命令编译生成可执行文件，这就是 makefile 的功能

首先 我们还是使用 Hello World 源文件，我用记事本创建了它》

![C++](../../../img/Makefile%20入门/2021-04-22-15-22-18.png)

并保存到了我的工作目录 D:\space\Hello\Hello.cpp

所谓 makefile 可以理解为一种简单的语言，它至少需要 3 个部分：

1. 目标文件 (要生成的文件)
2. 所需文件 (生成目标文件所需的文件列表)
3. 生成规则

它的语法是这样的

![makefile-syntax](../../../img/Makefile%20入门/2021-04-22-15-22-37.png)

[目标文件] : [所需文件]
[生成规则]

注意 需要用分隔符代替空格 也就是说所需文件后要有一个分隔符和一个换行符，也就是先按 TAB 键再按回车 然后输入生成规则，Hello.cpp 的 makefile 文件内容如下：

![makefile-content](../../../img/Makefile%20入门/2021-04-22-15-23-30.png)
￼
我们来逐行分析：

1. 第一行的意思是说 要生成 Hello 可执行文件 (exe) 需要使用到 Hello.obj 文件
2. 第二行则是 MinGW 下使用 Hello.obj 来链接生成 Hello (Hello.exe) 所需要的命令
3. 第三行为了易读留白
4. 第四行的意思是说 要生成 Hello.obj 文件需要使用到 Hello.cpp 文件
5. 第五航是 MinGW 下使用 Hello.cpp 来编译生成 Hello.obj 所需要的命令

我把它保存到了 Hello.cpp 相同的路径下 并命名为 Makefile 即 D:\workspace\Hello\Makefile
注意 Makefile 文件是无类型文档 保存时请选择 "所有类型" 不要有扩展名

然后我们启动 cmd 或者 msys 并 cd 到工作目录下 输入 make 即可自动完成编译链接步骤：

![run](../../../img/Makefile%20入门/2021-04-22-15-23-48.png)

可以看到 已经成功了 至于 Makefile 和 gcc 的更多用法 请参考相关文档
推荐的文档 《跟我一起写 makefile》《Linux 下的 C 编程实战》《GCC 中文手册》等

简单的吐槽。

1. 映像和格式的问题在对应楼的楼中楼说过了，不再重复。
2. 哪里冒出来 - c 是〔跳过〕汇编和链接了？汇编不能跳过，链接还没开始。
3. 〔无类型文档〕这异教徒的高烧言论不吐槽了。make? Makefile, makefile 一般都可以，GNU make 也接受 GNU makefile. 参见 make (1).
4. 散弹注意。吐槽一下某楼的 gedit, 大概你真的找不到更平庸的编辑器了？