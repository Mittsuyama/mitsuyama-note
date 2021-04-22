# Docker Basis

AIM: Build, Ship and Run Any App, Anywhere

## BASIC CONSTITUTION

- 镜像（image）：只读模板，可以创建多个容器
- 容器（container）：独立运行的一个或者一组应用
- 仓库（Repository）：存放镜像

Docker 是一个 Client-Server 系统，Docker 的守护进程运行在主机上，通过 Socket 连接从客户端访问，接受客户端命令并管理容器。

## DOCKER & VIRTUAL MACHINE

1. 更少的抽象层，不需要 Hypervisor 实现硬件资源虚拟化，Docker 中的容器都是实际物理机的硬件资源，有更高的 CPU，内存的利用率。
2. 利用宿主机的内核，减少引导加载操作系统的操作。

## COMMAND

### 镜像命令 image command

### HELP

```bash
# HELP 帮助
docker version
docker info
docker --help
```

### IMAGES 管理

```bash
docerk iamges [OPTIONS]

# -a：列出本地所有镜像（含中间镜像层）
# -q：只显示 ID
# --digest：显示镜像的摘要信息
# --no-trunc：显示完整的镜像信息
```

### SEARCH

```bash
docker search [OPTIONS] image_name

# --no-trunc：显示完整的镜像描述
# -s：列出收藏数不小于制定值的镜像
# --automated：只列出 automated build 类型的镜像
```

### PULL

```bash
# 从仓库拉取 image
docker pull image_name[:TAG]
```

### RMI

```bash
# 删除一个和多个 image
docker rmi -f image_id[:TAG][ image2_id[:TAG] ...]

# 删除所有 image
docker rmi -f $(docker images -qa)
```

### 容器命令 container command

### RUN

```bash
docker run [OPTIONS] image_[name | id] [COMMAND] [ARG...]

# --name="container_name"：为容器指定一个名字，否则 docker 会随机一个名字
# -d：后台运行容器，并返回一个容器 ID，也叫「启动守护式容器」
# -P：随机端口映射
# -p：[ip:]hostPort:containerPort 指定端口映射
#     其他格式：-p ip::containerPort, -p containerPort

```

COMMAND] 为 `/bin/bash` 时，以交互式模式启动。

### PS

```bash
# 列出正在运行的容器
docker ps [OPTIONS]

# -a：列出所有正在运行和历史上运行过的容器
# -l：最近创建的容器
# -n：最近 n 个创建的容器
# -q：静默模式，只显示容器编号
# --no-trunc：不截断输出
```

### EXIT

```bash
# 退出容器的方式
exit          # 停止并退出
ctrl + p + q  # 不停止退出
```

### OTHER

```bash
# 其他命令
docker start [name | id]    # 启动容器
docker restart [name | id]  # 重启容器
docker stop [name | id]     # 停止
docker kill                 # 强制停止

# 删除
docker rm container_id
# 删除多个
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm
```

### IMPORANT 重要的指令

### 启动守护式进程

```bash
docker run -d name
```

如果运行 `docker run -d centos`，再查看当前运行容器，`docker ps -a`，会发现这个容器已经退出了。
注意：**只有存在前台进程，Docker 才会保留后台运行**，让程序以前台的方式运行即可。

### 打印容器日志

```bash
docker logs -f -t --tail number container_id

# -f: 最新日志
# -t：日志前加上时间错
# --tail number: 显示多事少条 
```

### 与容器进行交互

```bash
# 直接进入容器启动命令的终端
docker attach container_id

# 重新打开一个新的终端
docker exec -it container_id /bin/bash
```

### 拷贝容器内文件

```bash
docker cp container_id:container_path host_path
```

### 容器细节

```bash
# 查看容器内运行进程
docker top container_id

# 查看容器细节
docker inspect container_id
```

## Docker Image

### 联合文件系统（UnionFS）

Union 文件系统（UnionFS）是一种**分层、轻量级并且高性能**的文件系统。**它支持对文件系统的修改作为一次提交一层层的叠加**。同时可以将不同的目录挂载到同一个虚拟文件系统下。

### Docker 镜像加载原理

bootFS（boot file system），主要包含 bootLoader 和 kernel，bootLoader 主要是引导加载 kernel，Linux 刚启动时会在 BootFS 文件系统，这一层与典型的 Linux/Unix 系统一样，当 boot 加载完成后，内核就都在内核中了，此时内存的使用权由 BootFS 转交给内核，系统也会卸载 BootFS。
BootFS（root file system），在 BootFS 上，包含的就是典型的 Linux 系统中的 /dev, /bin, /etc 等标准目录和文件。BootFS 就是各种不同的操作系统的发行版。
对于精简的 OS（docker 中的 centos），rootfs 可以很小，只包含最基本的命令，工具和程序库，底层直接使用宿主机（HOST）的 kernerl

### 为什么要用分层结构

主要目标：共享资源

如果好多镜像都是从相同的 base 镜像构建而来的，那么宿主机只需要在磁盘上保存一份 base 镜像。内存中只要有一份 base 镜像，就可以为所有容器服务了，镜像的每一层都可以被共享。

Docker 镜像都是只读的，当容器启动时，一个新的可写层被加载到被加载到顶部，这一层通常被称做「容器层」，「容器层」之下的都叫做「镜像层」。

### Docker 镜像 commit 操作

```bash
# 提交容器副本使之成为一个新的镜像
docker commit container_id [-m="description" -a="author" image_name[:TAG]]
```

## Docker 容器数据卷

目的：

1. 容器的持久化
2. 容器间继承 + 共享数据

### 数据卷

数据卷：容器内数据直接映射到本地主机环境。

### 直接命令添加

```bash
# 直接命令添加
docker run -it -v /host_path:/container_path image_name

# 容器只读 :ro
docker run -it -v /host_path:/container_path:ro image_name

# 容器可读可写 :rw
docker run -it -v /host_path:/container_path:rw image_name

# 查看是否挂在成功
docker inspect container_id
# "volumes": {
#   "/host_path": "/container_path",
#},
```

挂在成功后，即使容器停止退出，宿主机中修改文件，重新运行容器也会同步。

### DockerFile 添加

出于可移植和可分享的考虑，用 `v host_path:container_path` 不能够直接在 DockerFile 中实现。（由于不能保证所有的宿主机都存在这样的目录）
在 DockerFile 中加入：

```bash
VOLUME ["/dataVolumeContainer1", "/dataVolumeContainer2"]

# 主机中的默认目录为：/var/lib/docker/volumes/id
```

如果访问出现：「cannot open director: Permission denied」，加上 `-privileged=true` 即可。
DockerFile 在后文会更详细地描述。

### 数据卷容器

使用特定容器维护数据卷。能够在容器与主机，容器与容器之间共享数据，并实现数据的备份和回复。

```bash
# 命令
docker run --volumes-from container_name image_name
```

有下面的继承关系：

```bash
container-1
        L container-2
        L container-3
                L container-4
```

即使停止并删除 container-1 和 container-3，container-2 和 container-4 的数据卷之间依然能够同步。**容器之间的配置信息的传递，数据卷的声明周期一直持续到没有容器使用它为止。**

## DockerFile 解析

用来构建 Docker 镜像的构建文件，是由一系列命令和参数构成的脚本。

### DockerFile 构建过程

基础知识：

1. 保留字指令必须大写，指令之后至少跟随一个参数
2. 指令从上到下，顺序执行
3. `#` 表示注释
4. 每条指令都会创建一个新的镜像层，并对镜像进行提交

Docker 执行 DockerFile 的过程：

1. Docker 从基础镜像运行一个容器
2. 执行一条指令并对容器做出修改
3. 执行类似 `docker commit` 的操作提交一个新的镜像层
4. 再基于刚提交的的镜像运行一个新的容器
5. 执行 DockerFile 中的下一条指令直到所有指令都执行完成

### DockerFile 保留字指令

1. FROM：基础镜像，基于哪个镜像
2. MAINTAINER：维护者的姓名和邮箱
3. RUN：容器构建时需要运行的命令
4. EXPOSRE：对外暴露出的端口
5. WORKDIR：容器运行后，终端默认登录进来的工作目录
6. ENV：设置构建镜像过程中的环境变量
7. ADD：拷贝文件进镜像并且会自动处理 URL 和 tar 压缩包
8. COPY：COPY src path（同 ADD 但是不会自动处理）
9. VOLUME：容器数据卷
10. CMD ：指定容器启动时运行的名命令，只有最后一个 CMD 会生效
11. ENTRYPOINT：同 CMD，多个命令是叠加而不是覆盖
12. ONBUILD：子镜像 build 时触发

### 指令解释

镜像最终都继承于 scratch：`FROM scratch`