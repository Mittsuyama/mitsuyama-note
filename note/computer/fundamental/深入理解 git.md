# 深入了解 Git

文档地址：[https://git-scm.com/book/zh/v2/Git-内部原理-Git-对象](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%AF%B9%E8%B1%A1)

Git 的核心是一个**键值对数据库**，一个**内容寻址文件系统**。

## Git 对象

`git hash-object`，参数 `-w`：不只是返回 hash 值，同时存入 .git/objects 下。.git/objects 文件夹下，hash 值结果：2 + 38：目录 + 文件名。

`git cat-file` 命令

1. `-p` 参数自动判断类型，大致显示内容。
2. `-t` 参数返回对象类型

`git cat-file -p 0d34beae… > file.type`，只要能记住 SHA-1 的值，甚至可以直接取回文件内容。

## 树对象

![tree-object](../../../img/深入理解%20git/2021-04-22-15-18-17.png)

查看树对象：`git cat-file -p master^{tree}`

文件中包含：数据对象，子树对象（指针 + 模式 + 类型 + 信息）