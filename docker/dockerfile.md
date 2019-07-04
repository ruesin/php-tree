# Dockerfile

Dockerfile 由一行行命令语句组成，并且支持以 # 开头的注释行。

一般的，Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

其中，一开始必须指明所基于的镜像名称，接下来推荐说明维护者信息。

后面则是镜像操作指令，例如 RUN 指令，RUN 指令将对镜像执行跟随的命令。每运行一条 RUN 指令，镜像添加新的一层，并提交。

最后是 CMD 指令，来指定运行容器时的操作命令。

下面是一个更复杂的例子

## 指令

指令的一般格式为`INSTRUCTION arguments`，指令包括 FROM、MAINTAINER、RUN 等。

- FROM

  格式为`FROM <image>`或`FROM <image>:<tag>`。

  第一条指令必须为`FROM`指令。如果在同一个Dockerfile中创建多个镜像时，可以使用多个 FROM 指令（每个镜像一次）。

- MAINTAINER

  格式为`MAINTAINER <name>`，指定维护者信息。

- RUN
  
  格式为`RUN <command>`或`RUN ["executable", "param1", "param2"]`。

  前者将在 shell 终端中运行命令，即 /bin/sh -c；后者则使用 exec 执行。指定使用其它终端可以通过第二种方式实现，例如 RUN ["/bin/bash", "-c", "echo hello"]。

  每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 \ 来换行。

- CMD
  
  支持三种格式：
  - `CMD ["executable","param1","param2"]`使用 exec 执行，推荐方式；
  - `CMD command param1 param2` 在 /bin/sh 中执行，提供给需要交互的应用；
  - `CMD ["param1","param2"]` 提供给 ENTRYPOINT 的默认参数；

  指定启动容器时执行的命令，每个 Dockerfile 只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。

  如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。

- EXPOSE
  
  格式为`EXPOSE <port> [<port>...]`。告诉 Docker 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 -P，Docker 主机会自动分配一个端口转发到指定的端口。

- ENV
  
  格式为`ENV <key> <value>`。 指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持。

- ADD
  
  格式为`ADD <src> <dest>`。该命令将复制指定的 <src> 到容器中的 <dest>。 其中 <src> 可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）。

- COPY
  
  格式为`COPY <src> <dest>`。复制本地主机的 <src>（为 Dockerfile 所在目录的相对路径）到容器中的 <dest>。当使用本地目录为源目录时，推荐使用 COPY。

- ENTRYPOINT
  
  两种格式：
  - `ENTRYPOINT ["executable", "param1", "param2"]`
  - `ENTRYPOINT command param1 param2`（shell中执行）。

  配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。

  每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。

- VOLUME
  
  格式为`VOLUME ["/data"]`。创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

- USER
  
  格式为`USER daemon`。指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。

  当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如：`RUN groupadd -r postgres && useradd -r -g postgres postgres`。要临时获取管理员权限可以使用 gosu，而不推荐 sudo。

- WORKDIR
  
  格式为`WORKDIR /path/to/workdir`。为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录。

  可以使用多个 WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如
  ```
  WORKDIR /a
  WORKDIR b
  WORKDIR c
  RUN pwd  ### /a/b/c。
  ```

- ONBUILD
  
  格式为`ONBUILD [INSTRUCTION]`。配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。

> 例如，Dockerfile 使用如下的内容创建了镜像 image-A。

```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
```

> 如果基于 image-A 创建新的镜像时，新的Dockerfile中使用 FROM image-A指定基础镜像时，会自动执行ONBUILD 指令内容，等价于在后面添加了两条指令。

```dockerfile
FROM image-A

#Automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build --dir /app/src
```

> 使用 ONBUILD 指令的镜像，推荐在标签中注明，例如 ruby:1.9-onbuild。

## 创建镜像
编写完成 Dockerfile 之后，可以通过 docker build 命令来创建镜像。

基本的格式为`docker build [选项] 路径`，该命令将读取指定路径下（包括子目录）的 Dockerfile，并将该路径下所有内容发送给 Docker 服务端，由服务端来创建镜像。因此一般建议放置`Dockerfile`的目录为空目录。也可以通过`.dockerignore` 文件（每一行添加一条匹配模式）来让 Docker 忽略路径下的目录和文件。

要指定镜像的标签信息，可以通过 -t 选项，例如
```
$ sudo docker build -t myrepo/myapp /tmp/test1/
```

## 最佳实践

这里的最佳实践很多都依赖于[Docker原理](./docker.md)

- 编写.dockerignore 文件
  
  构建镜像时，Docker 需要先准备context，将所有需要的文件收集到进程中。默认的context包含 Dockerfile 目录中的所有文件，但是实际上，我们并不需要.git 目录，node_modules 目录等内容。`.dockerignore`的作用和语法类似于`.gitignore`，可以忽略一些不需要的文件，这样可以有效加快镜像构建时间，同时减少 Docker 镜像的大小。

- 容器只运行单个应用
  
  从技术角度讲，你可以在 Docker 容器中运行多个进程。你可以将数据库，前端，后端，ssh，supervisor 都运行在同一个 Docker 容器中。但是，这会让你非常痛苦：
  - 非常长的构建时间(修改前端之后，整个后端也需要重新构建)
  - 非常大的镜像大小
  - 多个应用的日志难以处理(不能直接使用 stdout，否则多个应用的日志会混合到一起)
  - 横向扩展时非常浪费资源(不同的应用需要运行的容器数并不相同)
  - 僵尸进程问题 - 你需要选择合适的 init 进程

  应该为每个应用构建单独的 Docker 镜像，然后使用 Docker Compose 运行多个 Docker 容器。

- 将多个 RUN 指令合并为一个

  Docker 镜像是分层的：
  - Dockerfile 中的每个指令都会创建一个新的镜像层。
  - 镜像层将被缓存和复用
  - 当 Dockerfile 的指令修改了，复制的文件变化了，或者构建镜像时指定的变量不同了，对应的镜像层缓存就会失效
  - 某一层的镜像缓存失效之后，它之后的镜像层缓存都会失效
  - 镜像层是不可变的，如果我们再某一层中添加一个文件，然后在下一层中删除它，则镜像中依然会包含该文件(只是这个文件在 Docker 容器中不可见了)。
  
  Docker 镜像类似于洋葱。它们都有很多层。为了修改内层，则需要将外面的层都删掉。

  将变化频率一样的指令合并在一起。将 node.js 安装与 npm 模块安装放在一起的话，则每次修改源代码，都需要重新安装 node.js，这显然不合适。

- 基础镜像的标签不要用 latest
  
  当镜像没有指定标签时，将默认使用latest 标签。`FROM ubuntu`指令等同于`FROM ubuntu:latest`。当时，当镜像更新时，latest 标签会指向不同的镜像，这时构建镜像有可能失败。如果你的确需要使用最新版的基础镜像，可以使用 latest 标签，否则的话，最好指定确定的镜像标签。

- 每个 RUN 指令后删除多余文件
  
  假设更新了 apt-get 源，下载，解压并安装了一些软件包，它们都保存在/var/lib/apt/lists/目录中。但是，运行应用时 Docker 镜像中并不需要这些文件。最好将它们删除，因为它会使 Docker 镜像变大。

- 选择合适的基础镜像(alpine 版本最好)
  
  alpine 是一个极小化的 Linux 发行版，只有 4MB，这让它非常适合作为基础镜像。

  apk是 Alpine 的包管理工具。它与apt-get有些不同，但是非常容易上手。另外，它还有一些非常有用的特性，比如no-cache和 --virtual选项，它们都可以帮助减少镜像的大小。

- 设置 WORKDIR 和 CMD
  
  WORKDIR指令可以设置默认目录，也就是运行`RUN/CMD/ENTRYPOINT`指令的地方。

  CMD指令可以设置容器创建是执行的默认命令。另外，应该将命令写在一个数组中，数组中每个元素为命令的每个单词。

- 使用 ENTRYPOINT (可选)

  ENTRYPOINT指令并不是必须的，因为它会增加复杂度。ENTRYPOINT是一个脚本，它会默认执行，并且将指定的命令错误其参数。它通常用于构建可执行的 Docker 镜像。

- 在 entrypoint 脚本中使用 exec

  不使用exec的话，不能顺利地关闭容器，因为 SIGTERM 信号会被 bash 脚本进程吞没。exec命令启动的进程可以取代脚本进程，因此所有的信号都会正常工作。

- COPY 与 ADD 优先使用前者

  COPY指令非常简单，仅用于将文件拷贝到镜像中。ADD相对来讲复杂一些，可以用于下载远程文件以及解压压缩包。

- 合理调整 COPY 与 RUN 的顺序
  
  应该把变化最少的部分放在 Dockerfile 的前面，这样可以充分利用镜像缓存。

- 设置默认的环境变量，映射端口和数据卷
  
  运行 Docker 容器时很可能需要一些环境变量。在 Dockerfile 设置默认的环境变量是一种很好的方式。另外，应该在 Dockerfile 中设置映射端口和数据卷。

- 使用 LABEL 设置镜像元数据

  使用LABEL指令，可以为镜像设置元数据，例如镜像创建者或者镜像说明。旧版的 Dockerfile 语法使用MAINTAINER指令指定镜像创建者，但是它已经被弃用了。有时，一些外部程序需要用到镜像的元数据，例如nvidia-docker需要用到com.nvidia.volumes.needed。

- 添加 HEALTHCHECK

  运行容器时，可以指定--restart always选项。这样的话，容器崩溃时，Docker 守护进程(docker daemon)会重启容器。对于需要长时间运行的容器，这个选项非常有用。但是，如果容器的确在运行，但是不可(陷入死循环，配置错误)用怎么办？使用HEALTHCHECK指令可以让 Docker 周期性的检查容器的健康状况。我们只需要指定一个命令，如果一切正常的话返回 0，否则返回 1。
  ```
  EXPOSE $APP_PORT
  HEALTHCHECK CMD curl --fail http://localhost:$APP_PORT || exit 1
  ```
