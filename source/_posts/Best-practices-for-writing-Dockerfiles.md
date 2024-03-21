---
title: Dockerfile的最佳实践
date: 2022-09-12 00:00:00
categories:
- Docker
tags:
- Docker
---

## 简介

Docker通过读取Dockerfile文件中的指令自动构建镜像。Dockerfile文件为一个文本文件，里面包含构建镜像所需的所有的命令。Dockerfile文件遵循特定的格式和指令集 Docker镜像由只读层组成，每个层都代表一个Dockerfile指令。这些层是堆叠的，每个层都是前一层变化的增量。示例：

```
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

每个指令构建一层： 
FROM 根据ubuntu18.04docker镜像创建一个层 
COPY 将Docker客户端当前的目录文件添加到镜像中 
RUN 使用make构建应用程序 
CMD 指定在容器中运行的命令

通过镜像启动一个容器的时候，会在基础层上添加一个可写层。对正在运行的容器所做的所有更改（例如，写入新文件，修改现有文件和删除文件）都将写入此可写容器层。

## 建议

### 1.创建临时容器

通过Dockerfile文件定义的镜像，产生的容器尽可能的是临时的。所谓的临时，意思是，容器可停止，销毁，重建和替代为最小设置和配置。

### 2.理解构建上下文

在我们构建镜像的时候，一般情况下，会使用

```
docker build -t name:tag .
```

当你发出一个docker build命令时，当前目录将作为构建的上下文。Dockerfile默认存放在此目录下。但是，你可以通过-f选项指定一个不同的目录。不论Dockerfile文件实际存放在何处。当前目录中的所有文件和目录的递归内容都将作为构建上下文发送到Docker守护程序。

示例： 创建一个用于构建上下文的目录，并cd进入该目录下。写入"Hello"字符串到一个文本文件中，命名为hello并创建一个Dockerfile文件运行cat命令，通过`.`构建上下文构建镜像。

```
mkdir myproject && cd myproject
echo "hello" > hello
echo -e "FROM busybox\nCOPY /hello /\nRUN cat /hello" > Dockerfile
docker build -t helloapp:v1 .
```

将Dockerfile和hello移到不同的目录下，构建第二版镜像（不依赖于上次构建的缓存）。使用-f 指定Dockerfile文件的目录，并指定构建的上下文目录。

```
mkdir -p dockerfiles context
mv Dockerfile dockerfiles && mv hello context
docker build --no-cache -t helloapp:v2 -f dockerfiles/Dockerfile context
```

构建镜像时，不经意包含不必要的文件将会导致一个臃肿的构建上下文和一个臃肿的镜像，这将导致构建镜像时长增加，提交到仓库和从仓库拉取时长，容器运行时大小都将增加。构建镜像时可以看到构建上下文的大小。

#### 通过.dockerignore文件排除

要排除与构建无关的文件（不重构源存储库），请使用.dockerignore文件。此文件支持类似于.gitignore文件的排除模式

#### 使用多级构建

多级构建允许你大幅减小最终图像的大小而无需努力减少中间层和文件的数量。由于图像是在构建过程的最后阶段构建的，因此可以通过利用构建缓存来最小化图像层。例如，如果您的构建包含多个图层，则可以从较不频繁更改（以确保构建缓存可重用）到更频繁更改的顺序对它们进行排序：

-   build程序所需的工具安装
-   安装升级库依赖
-   生产应用程序

示例： 一个go应用程序的Dockerfile文件如下：

```
FROM golang:1.11-alpine AS build

# Install tools required for project
# Run `docker build --no-cache .` to update dependencies
RUN apk add --no-cache git
RUN go get github.com/golang/dep/cmd/dep

# List project dependencies with Gopkg.toml and Gopkg.lock
# These layers are only re-built when Gopkg files are updated
COPY Gopkg.lock Gopkg.toml /go/src/project/
WORKDIR /go/src/project/
# Install library dependencies
RUN dep ensure -vendor-only

# Copy the entire project and build it
# This layer is rebuilt when a file changes in the project directory
COPY . /go/src/project/
RUN go build -o /bin/project

# This results in a single layer image
FROM scratch
COPY --from=build /bin/project /bin/project
ENTRYPOINT ["/bin/project"]
CMD ["--help"]
```

#### 不要安装不必要的包

为了降低复杂性，依赖性，文件大小和构建时间，请避免安装额外的或不必要的软件包，因为它们可能“很好”。例如，您不需要在数据库映像中包含文本编辑器。

#### 解耦应用程序

每个容器应该只有一个关注点。将应用程序解耦到多个容器可以更容易地水平伸缩和重用容器。例如，web应用程序堆栈可能由三个独立的容器组成，每个容器都有自己独特的映像，以解耦的方式管理web应用程序、数据库和内存缓存。

将每个容器限制为一个进程是一个很好的经验法则，但它不是一个硬性规则。 例如，不仅可以使用init进程生成容器，而且某些程序可能会自行生成其他进程。 例如，Celery可以生成多个工作进程，Apache可以为每个请求创建一个进程。

使用你最好的判断来保持容器尽可能的干净和模块化。如果容器彼此依赖，可以使用Docker容器网络来确保这些容器能够通信。

#### 减少图层的数量

在旧版本的Docker中，最大限度地减少图像中的图层数量以确保它们具有高性能非常重要。添加了以下特性来减少这种限制:

-   只用RUN,COPY,ADD指令会创建层，其他指令创建临时中间层，并不增加构建的大小。
-   在可能的情况下，使用多阶段构建，并仅将所需的工件复制到最终图像中。这允许您在中间构建阶段中包含工具和调试信息，而不会增加最终图像的大小

#### 多行参数排序

只要有可能，通过按字母顺序排序多行参数来缓解以后的更改。这有助于避免重复包并使列表更容易更新。这也使PR更容易阅读和审查。 在反斜杠（\\）之前添加空格也有帮助。

示例：

```
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

#### 缓存利用

构建映像时，Docker会逐步执行Dockerfile中的指令，按指定的顺序执行每个指令。 在检查每条指令时，Docker会在其缓存中查找可以重用的现有映像，而不是创建新的（重复）映像。

如果你不想使用缓存，可以在docker build命令中使用`--no-cache=true`选项，然而，如果你让Docker使用缓存，理解什么时候使用，什么时候不能，使用一个匹配的镜像将非常重要，Docker遵循的基本规则概述如下：

-   从已在缓存中的父镜像开始，接下来的指令将对比所有的子镜像看是否有一个使用相同的指令构建而成，如果没有，缓存则是无效的。
-   在大多数情况下，只需将Dockerfile中的指令与其中一个子映像进行比较就足够了。然而，某些指示需要更多的检查和解释。
-   对于ADD和COPY指令，将检查映像中文件的内容，并为每个文件计算校验和。 在这些校验和中不考虑文件的最后修改时间和最后访问时间。 在高速缓存查找期间，将校验和与现有映像中的校验和进行比较。 如果文件中的任何内容（例如内容和元数据）发生了任何更改，则缓存将失效。
-   除了ADD和COPY命令之外，高速缓存检查不会查看容器中的文件以确定高速缓存匹配。 例如，在处理RUN apt-get -y update命令时，不检查容器中更新的文件以确定是否存在缓存命中。 在这种情况下，只需使用命令字符串本身来查找匹配项。

一旦缓存无效，Dockerfile命令将产生新的镜像而不使用缓存。

## Dockerfile指令

### FROM

`FROM <image> [AS <name>]`

或者

`FROM <image>[:<tag>] [AS <name>]`

或者

`FROM <image>[@<digest>] [AS <name>]`

-   FROM指令初始化新的构建阶段并为后续指令设置基本映像。因此，有效的Dockerfile必须以FROM指令开头.
-   FROM可以出现多次在同一个Dockerfile文件中，为了创建多个镜像，或者使用一个作为另一个镜像的依赖，只要在每个新的FROM执行之前记录上一个镜像的ID。每个FROM指令都会清空之前命令创建的任何状态。
-   可选的，每个FROM指令都可以通过`AS name`提供一个名词,该名词可以在子FROM指令和COPY --from<name|index>指令中指待该镜像。
-   tag和digest值是可选的，如果省略他们，将使用latest版本，如果未找到将会抛出异常。 尽可能的使用官方镜像作为基础镜像。

示例：

```
FROM golang:1.10.3 as builder
WORKDIR /app/
RUN mkdir -p src/github.com \ 
    && mkdir -p src/golang.org \
    && mkdir -p src/gopkg.in \
    && mkdir -p src/qiniupkg.com \
    && mkdir -p src/google.golang.org \
    && mkdir -p src/go4.org
```

了解ARG和FROM如何互动： FROM指令支持在第一个FROM之前通过任意ARG指令声明变量 示例：

```
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```

### LABEL

你可以为你的镜像添加labels，用来组织镜像，记录证书信息，或者其他原因，对应每个label，增加以LABEL开头的行，和一个或者多个键值对。下面的示例中展示的是不同形式的：

```
# Set one or more individual labels
LABEL com.example.version="0.0.1-beta"
LABEL vendor1="ACME Incorporated"
LABEL vendor2=ZENITH\ Incorporated
LABEL com.example.release-date="2015-02-12"
LABEL com.example.version.is-production=""
```

上面的也可以写成如下形式：

```
# Set multiple labels at once, using line-continuation characters to break long lines
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```

### RUN

-   RUN
-   RUN \["executable", "param1", "param2"\]

RUN指令将执行任何命令在当前镜像的一个新层上并提交结果。提交后的镜像将会在下一步中使用。

在使用反斜杠分隔的多行上拆分长或复杂的RUN语句，以使Dockerfile更具可读性，可理解性和可维护性。

`RUN apt-get update && apt-get install -y`这些命令不要分开，

示例：

```
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    build-essential \
    curl \
    dpkg-sig \
    libcap-dev \
    libsqlite3-dev \
    mercurial \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.* \
 && rm -rf /var/lib/apt/lists/*

```

上面的这个指令是建议这样使用的，最后的执行用于清除安装过程中的缓存。

### CMD

`CMD ["executable","param1","param2"]`

`CMD ["param1","param2"]`

`CMD command param1 param2`

**一个Dockerfile文件中，只能有一个CMD指令，如果出现多个，只有最后一个起作用。**

CMD的主要目的是为执行容器提供默认值。 这些默认值可以包含可执行文件，

也可以省略可执行文件，在这种情况下，您还必须指定ENTRYPOINT指令。

如果CMD用于为ENTRYPOINT指令提供默认参数，则应使用JSON数组格式指定CMD和ENTRYPOINT指令。exec表单被解析为JSON数组，这意味着您必须使用双引号（“）来围绕单词而不是单引号（'）。

CMD指令应该被用于运行被镜像包含的软件，并使用提供的参数。CMD指令应尽可能的使用CMD \["executable", "params1","params2"...\].因此，如果镜像是一个服务，比如Apache，应这样运行CMD \["apache2", "-DFOREGROUND"\].事实上，这种形式的指令被建议使用在以服务为基础的镜像上。

大多数情况，CMD应该提供一个交互式的shell。例如bash.python.perl.示例： `CMD ["perl", "-de0"]`, `CMD ["python"]`,或者`CMD ["php" "-a"]`.以这种形式的指令，意味着当你执行如这样的命令时`docker run -it python`,你将进入一个可用的shell里面，CMD指令在和ENTRYPOINT指令结合使用时，将很少使用`CMD ["param", "param"]`这种形式，除非你和客户都很熟悉ENTRYPOINT是如何工作的。

示例：

```
FROM busybox as final
COPY --from=builder /app/src /opt/app/src
EXPOSE 8080
WORKDIR  /opt/app/
CMD ["./server"]
```

### EXPOSE

`EXPOSE <port> [<port>/<protocol>...]`

EXPOSE指令通知Docker容器在运行时侦听指定的网络端口。 您可以指定端口是侦听TCP还是UDP，如果未指定协议，则默认为TCP。EXPOSE指令实际上不会发布端口。 它在构建映像的人和运行容器的人之间起到一种文档的作用，关于哪些端口要发布。要在运行容器时实际发布端口，请在docker run上使用-p标志发布和映射一个或多个端口，或使用-P标志发布所有公开的端口并将它们映射到高阶端口。默认情况下，EXPOSE假定为TCP。 您还可以指定UDP：

`EXPOSE 80/udp`

要同时在TCP和UDP上公开，需要包含两行:

```
EXPOSE 80/tcp
EXPOSE 80/udp
```

在这种情况下，如果将-P与docker run一起使用，则端口将为TCP公开一次，对UDP公开一次。 请记住，-P在主机上使用短暂的高阶主机端口，因此TCP和UDP的端口不同。

无论EXPOSE设置如何，您都可以使用-p标志在运行时覆盖它们。 例如: `docker run -p 80:80/tcp -p 80:80/udp ...`

要在主机系统上设置端口重定向，请参阅使用-P标志。 docker network命令支持创建用于容器之间通信的网络，而无需公开或发布特定端口，因为连接到网络的容器可以通过任何端口相互通信。

EXPOSE指令指示容器侦听连接的端口。 因此，您应该为您的应用程序使用通用的传统端口。 例如，包含Apache Web服务器的映像将使用EXPOSE 80，而包含MongoDB的映像将使用EXPOSE 27017等.

对于外部访问，您的用户可以使用标志执行docker run，该标志指示如何将指定端口映射到他们选择的端口。 对于容器链接，Docker为从收件人容器返回源的路径提供环境变量.

示例：

```
FROM busybox as final
COPY --from=builder /app/src /opt/app/src
EXPOSE 8080
WORKDIR  /opt/app/
CMD ["./server"]
```

这样我们在`docker ps` 的时候就会发现

```
44af1f00f971        server_number1:1.0    "./server"               3 hours ago         Up 14 seconds      8080/tcp                             server_number1_1
b372192ef894        server_number2:1.0     "./server"             3 hours ago         Up 15 seconds      8080/tcp                             server_number2_1
9ffa14236ab1        nginx:latest        "/docker-entrypoint.…"   3 hours ago         Up 21 seconds       0.0.0.0:80->80/tcp                   server_nginx_1
7b73c43a97d3        redis:3.2           "docker-entrypoint.s…"   3 hours ago         Up 21 seconds       0.0.0.0:6379->6379/tcp               server_redis_1
8df25db59837        mysql:5.7           "docker-entrypoint.s…"   4 hours ago         Up 21 seconds       33060/tcp, 0.0.0.0:33306->3306/tcp   server_mysql_1
```

### ENV

```
ENV <key> <value>
ENV <key>=<value> ...
```

ENV指令将环境变量设置为值。 此值将在构建阶段中的所有后续指令的环境中，并且也可以在许多内联替换。

ENV指令有两种形式。 第一种形式ENV ，将单个变量设置为一个值。 第一个空格后面的整个字符串将被视为 - 包括空格字符。 该值将针对其他环境变量进行解释，因此如果未对其进行转义，则将删除引号字符。

第二种形式ENV = ...允许一次设置多个变量。 请注意，第二种形式在语法中使用等号（=），而第一种形式则不然。 与命令行解析一样，引号和反斜杠可用于在值内包含空格。

示例：

```
FROM golang:1.10.3 as builder
RUN yum install -y gcc \
    && yum install -y gcc-c++ kernel-devel make
ENV GOPATH /go
ENV PATH $PATH:$GOPATH/bin
```

为了使新软件更易于运行，您可以使用ENV更新容器安装的软件的PATH环境变量。例如： `ENV PATH /usr/local/nginx/bin:$PATH`确保`CMD ["nginx"]`正常运行。

ENV指令对于提供特定于您希望容纳的服务的必需环境变量也很有用，例如Postgres的PGDATA。 最后，ENV还可用于设置常用版本号，以便更容易维护版本迭代，如以下示例所示：

```
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

类似于在程序中使用常量变量（与硬编码值相反），此方法允许您更改单个ENV指令以自动神奇地修改容器中的软件版本。

每条ENV线都会创建一个新的中间层，就像RUN命令一样。 这意味着即使您在将来的图层中取消设置环境变量，它仍然会在此图层中保留，并且可以转储其值。 您可以通过创建如下所示的Dockerfile来测试它，然后构建它。

```
FROM alpine
ENV ADMIN_USER="mark"
RUN echo $ADMIN_USER > ./mark
RUN unset ADMIN_USER
```

`$ docker run --rm test sh -c 'echo $ADMIN_USER'`一样会输出mark

要防止这种情况，并且实际上取消设置环境变量，请使用带有shell命令的RUN命令，在单个图层中设置，使用和取消设置变量all。 您可以将命令与; 要么 ＆＆。 如果您使用第二种方法，并且其中一个命令失败，则docker构建也会失败。 这通常是一个好主意。 使用\\作为Linux Dockerfiles的行继续符可以提高可读性。 您还可以将所有命令放入shell脚本中，并使用RUN命令运行该shell脚本。

```
FROM alpine
RUN export ADMIN_USER="mark" \
    && echo $ADMIN_USER > ./mark \
    && unset ADMIN_USER
CMD sh
```

`docker run --rm test sh -c 'echo $ADMIN_USER'`不会输出mark

### ADD or COPY

#### ADD

-   `ADD [--chown=<user>:<group>] <src>... <dest>`
-   `ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]`

`--chown`功能仅用于Dockerfile构建linux容器.在windows容器上无效。

ADD指令从复制新文件，目录或远程文件URL，并将它们添加到路径的镜像文件系统中。 每个可能包含通配符，匹配将使用Go的filepath.Match规则完成。 例如：

是绝对路径，或相对于WORKDIR的路径，源将在目标容器中复制到该路径中。

```
ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
```

添加包含特殊字符（例如\[和\]）的文件或目录时，需要按照Golang规则转义这些路径，以防止它们被视为匹配模式。 例如，要添加名为arr \[0\] .txt的文件，请使用以下命令：

```
ADD arr[[]0].txt /mydir/    # copy a file named "arr[0].txt" to /mydir/
```

所有新文件和目录都是使用UID和GID为0创建的，除非可选的--chown标志指定给定用户名，组名或UID / GID组合以请求添加内容的特定所有权。 --chown标志的格式允许用户名和组名字符串或任意组合的直接整数UID和GID。 提供没有组名的用户名或没有GID的UID将使用与GID相同的数字UID。 如果提供了用户名或组名，则容器的根文件系统/ etc / passwd和/ etc / group文件将分别用于执行从名称到整数UID或GID的转换。 以下示例显示了--chown标志的有效定义：

```
ADD --chown=55:mygroup files* /somedir/
ADD --chown=bin files* /somedir/
ADD --chown=1 files* /somedir/
ADD --chown=10:11 files* /somedir/
```

如果容器根文件系统不包含/ etc / passwd或/ etc / group文件，并且在--chown标志中使用了用户名或组名，则构建将在ADD操作上失败。 使用数字ID不需要查找，也不依赖于容器根文件系统内容.

在是远程文件URL的情况下，目标将具有600的权限。如果正在检索的远程文件具有HTTP Last-Modified标头，则该标头的时间戳将用于设置目标上的mtime 文件。 但是，与ADD期间处理的任何其他文件一样，mtime将不包含在确定文件是否已更改且应更新缓存中。

ADD遵守以下规则：

-   路径必须位于构建的上下文中; 你不能添加../something / something，因为docker构建的第一步是将上下文目录（和子目录）发送到docker守护进程。
-   如果是URL且不以尾部斜杠结尾，则从URL下载文件并将其复制到。
-   如果是URL并且以尾部斜杠结尾，则从URL推断文件名，并将文件下载到`<dest> /<filename>`。 例如，`ADD http://example.com/foobar /`将创建文件`/foobar`。 URL必须具有非常重要的路径，以便在这种情况下可以发现适当的文件名（[example.com将不起作用）](https://link.juejin.cn/?target=http%3A%2F%2Fexample.com%25E5%25B0%2586%25E4%25B8%258D%25E8%25B5%25B7%25E4%25BD%259C%25E7%2594%25A8%25EF%25BC%2589 "http://example.com%E5%B0%86%E4%B8%8D%E8%B5%B7%E4%BD%9C%E7%94%A8%EF%BC%89")
-   如果是目录，则复制目录的全部内容，包括文件系统元数据。不复制目录本身，只复制其内容。
-   如果是可识别的压缩格式（identity，gzip，bzip2或xz）的本地tar存档，则将其解压缩为目录。 远程URL中的资源不会被解压缩。 复制或解压缩目录时，它与tar -x具有相同的行为，结果是：

```
1.无论在目的地路径上存在什么，
2.源树的内容，在逐个文件的基础上解决了有利于“2.”的冲突。
```

-   如果是任何其他类型的文件，则将其与元数据一起单独复制。 在这种情况下，如果以尾部斜杠/结尾，则将其视为目录，的内容将写入 / base（）。
-   如果直接或由于使用通配符指定了多个资源，则必须是目录，并且必须以斜杠/结尾。
-   如果不以尾部斜杠结束，则将其视为常规文件，的内容将写入。
-   如果不存在，则会在其路径中创建所有缺少的目录。

#### COPY

-   `COPY [--chown=<user>:<group>] <src>... <dest>`
-   `COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]`

`--chown`功能仅用于Dockerfile构建linux容器.在windows容器上无效。

COPY指令从复制新文件或目录，并将它们添加到路径的容器的文件系统中。

可以指定多个资源，但文件和目录的路径将被解释为相对于构建上下文的源。

每个可能包含通配符，匹配将使用Go的filepath.Match规则完成。 例如：

```
COPY hom* /mydir/        # adds all files starting with "hom"
COPY hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
```

是绝对路径，或相对于WORKDIR的路径，源将在目标容器中复制到该路径中。

```
COPY test relativeDir/   # adds "test" to `WORKDIR`/relativeDir/
COPY test /absoluteDir/  # adds "test" to /absoluteDir/
```

复制包含特殊字符（例如\[和\]）的文件或目录时，需要按照Golang规则转义这些路径，以防止它们被视为匹配模式。 例如，要复制名为arr \[0\] .txt的文件，请使用以下命令：

```
COPY arr[[]0].txt /mydir/    # copy a file named "arr[0].txt" to /mydir/
```

除非可选的--chown标志指定给定用户名，组名或UID / GID组合以请求复制内容的特定所有权，否则将使用UID和GID为0创建所有新文件和目录。 --chown标志的格式允许用户名和组名字符串或任意组合的直接整数UID和GID。 提供没有组名的用户名或没有GID的UID将使用与GID相同的数字UID。 如果提供了用户名或组名，则容器的根文件系统/ etc / passwd和/ etc / group文件将分别用于执行从名称到整数UID或GID的转换。 以下示例显示了--chown标志的有效定义：

```
COPY --chown=55:mygroup files* /somedir/
COPY --chown=bin files* /somedir/
COPY --chown=1 files* /somedir/
COPY --chown=10:11 files* /somedir/
```

如果容器根文件系统不包含/ etc / passwd或/ etc / group文件，并且在--chown标志中使用了用户名或组名，则构建将在COPY操作上失败。 使用数字ID不需要查找，也不依赖于容器根文件系统内容。

可选地，COPY接受一个标志--from = <name | index>，可用于将源位置设置为先前的构建阶段（使用FROM .. AS 创建），而不是由发送的构建上下文 用户。 该标志还接受为使用FROM指令启动的所有先前构建阶段分配的数字索引。 如果找不到具有指定名称的构建阶段，则尝试使用具有相同名称的图像。

COPY遵守以下规则：

-   路径必须位于构建的上下文中; 你不能COPY ../something / something，因为docker构建的第一步是将上下文目录（和子目录）发送到docker守护进程。
-   如果是目录，则复制目录的全部内容，包括文件系统元数据。不复制目录本身，只复制其内容。
-   如果是任何其他类型的文件，则将其与元数据一起单独复制。 在这种情况下，如果以尾部斜杠/结尾，则将其视为目录，的内容将写入 / base（）。
-   如果直接或由于使用通配符指定了多个资源，则必须是目录，并且必须以斜杠/结尾。
-   如果不以尾部斜杠结束，则将其视为常规文件，的内容将写入。
-   如果不存在，则会在其路径中创建所有缺少的目录。

尽管ADD和COPY在功能上相似，但一般来说，COPY是优选的。 那是因为它比ADD更透明。 COPY仅支持将本地文件基本复制到容器中，而ADD具有一些功能（如仅限本地的tar提取和远程URL支持），这些功能并不是很明显。因此，ADD的最佳用途是将本地tar文件自动提取到图像中，如`ADD rootfs.tar.xz /`中所示.

如果在Dockerfile中有多个步骤使用不同的文件，请针对每个文件，单独执行COPY指令，而不是放在一起执行COPY,这确保了每一步构建的缓存在当前步骤使用，如果特定的文件改变了。 示例：

```
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY . /tmp/
```

与放置在COPY之前相比，RUN步骤的缓存失效次数更少。 由于图像大小很重要，因此强烈建议不要使用ADD从远程URL获取包。 你应该使用curl或wget代替。 这样，您可以删除提取后不再需要的文件，也不必在图像中添加其他图层。 例如，你应该避免做以下事情：

```
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

替代为：

```
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

对于不需要ADD的tar自动提取功能的其他项目（文件，目录），应始终使用COPY。

### ENTRYPOINT

-   `ENTRYPOINT ["executable", "param1", "param2"]` (exec form, preferred)
-   `ENTRYPOINT command param1 param2`

ENTRYPOINT允许您配置将作为可执行文件运行的容器。 例如，以下将使用其默认内容启动nginx，侦听端口80： `docker run -i -t --rm -p 80:80 nginx`

exec形式的ENTRYPOINT中的命令行参数将会追加到`docker run <image>`所有元素的后面。并且会覆盖CMD指令指定的参数。这允许将参数传递给入口点。 `docker run <image> -d`会将-d参数传递给入口点。 您可以使用`docker run --entrypoint`标志覆盖ENTRYPOINT指令。

shell表单可以防止使用任何CMD或run命令行参数，但缺点是`ENTRYPOINT`将作为`/bin/sh -c`的子命令启动，它不传递信号。 这意味着可执行文件不是容器的`PID 1` - 并且不会收到Unix信号 - 因此您的可执行文件不会从`docker stop <container>`接收`SIGTERM`。

Dockerfile文件中只有最后一个ENTRYPOINT指令会起作用.

#### 示例

你可以使用exec 形式的ENTRYPOINT来设置相对稳定的默认的命令和参数，并使用任意形式的CMD指令来设置额外的默认的可能会变化的。

```
FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```

当你运行容器的时候，可以发现top是唯一的进程 `docker run -it --rm --name test top -H`

为了未来测试结果，你可使用`docker exec`

```
$ docker exec -it test ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  2.6  0.1  19752  2352 ?        Ss+  08:24   0:00 top -b -H
root         7  0.0  0.1  15572  2164 ?        R+   08:25   0:00 ps aux
```

并且您可以优雅地请求停止`top`使用`docker stop test`。

下面的示例展示了使用`ENTRYPOINT`来运行Apache在前端

```
FROM debian:stable
RUN apt-get update && apt-get install -y --force-yes apache2
EXPOSE 80 443
VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

#### 示例2

可为ENTRYPOINT指定一个字符串,他将在`/bin/sh -c`中执行，这种形式使用shell处理替换环境变量，并会忽略CMD或者docker run 命令行参数。为了正确有效的执行`docker stop`,需要记住以`exec`开头

```
FROM ubuntu
ENTRYPOINT exec top -b
```

当运行这个镜像的时候，可以看到`PID 1`的进程

```
$ docker run -it --rm --name test top
Mem: 1704520K used, 352148K free, 0K shrd, 0K buff, 140368121167873K cached
CPU:   5% usr   0% sys   0% nic  94% idle   0% io   0% irq   0% sirq
Load average: 0.08 0.03 0.05 2/98 6
  PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
    1     0 root     R     3164   0%   0% top -b
```

当执行docker stop命令时，可正常退出

```
$ /usr/bin/time docker stop test
test
real0m 0.20s
user0m 0.02s
sys0m 0.04s
```

如果你忘记以exec开头，如下:

```
FROM ubuntu
ENTRYPOINT top -b
CMD --ignored-param1

```

运行；

```
$ docker run -it --name test top --ignored-param2
Mem: 1704184K used, 352484K free, 0K shrd, 0K buff, 140621524238337K cached
CPU:   9% usr   2% sys   0% nic  88% idle   0% io   0% irq   0% sirq
Load average: 0.01 0.02 0.05 2/101 7
  PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
    1     0 root     S     3168   0%   0% /bin/sh -c top -b cmd cmd2
    7     1 root     R     3164   0%   0% top -b

```

可以看到top输出表明ENTRYPOINT指令并未作为PID1的进程运行。 这个时候运行docker stop，容器不会正常退出，stop指令会强制发出一个`SIGKILL`信号在数分钟以后。

```
$ docker exec -it test ps aux
PID   USER     COMMAND
    1 root     /bin/sh -c top -b cmd cmd2
    7 root     top -b
    8 root     ps aux
$ /usr/bin/time docker stop test
test
real0m 10.19s
user0m 0.04s
sys0m 0.03s
```

#### 理解CMD和ENTRYPOINT指令如何整合

这两个指令都定义了当运行一个容器的时候，应运行什么命令，在合同使用时需遵守以下规则：

-   Dockerfile应至少指定一个CMD或ENTRYPOINT命令。
-   使用容器作为可执行文件时，应定义ENTRYPOINT。
-   CMD应该用作为ENTRYPOINT命令定义默认参数或在容器中执行ad-hoc命令的方法。
-   使用备用参数运行容器时，将覆盖CMD。

下表显示了针对不同ENTRYPOINT / CMD组合执行的命令：

| null                           | No ENTRYPOINT                | ENTRYPOINT exec\_entry p1\_entry | ENTRYPOINT \[“exec\_entry”, “p1\_entry”\]          |
| ------------------------------ | ---------------------------- | -------------------------------- | -------------------------------------------------- |
| No CMD                         | error, not allowed           | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry                              |
| CMD \[“exec\_cmd”, “p1\_cmd”\] | exec\_cmd p1\_cmd            | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry exec\_cmd p1\_cmd            |
| CMD \[“p1\_cmd”, “p2\_cmd”\]   | p1\_cmd p2\_cmd              | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry p1\_cmd p2\_cmd              |
| CMD exec\_cmd p1\_cmd          | /bin/sh -c exec\_cmd p1\_cmd | /bin/sh -c exec\_entry p1\_entry | exec\_entry p1\_entry /bin/sh -c exec\_cmd p1\_cmd |

如果CMD是从基础镜像中定义，设置ENTRYPOINT将会重置CMD为一个空的值，在这种情况下，CMD需要指定一个值在该镜像定义的时候。

ENTRYPOINT的最佳使用是用来设置镜像的主命令。允许该映像像该命令一样运行（然后使用CMD作为默认标志）。

让我们以一个命令行工具s3cmd的镜像的例子开始：

```
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```

现在可以使用下面的命令运行来展示命令行的帮助信息： `docker run s3cmd`

或者使用正确的参数来执行命令： `docker run s3cmd ls s3://mybucket`

### VOLUME

`VOLUME ["/data"]`

VOLUME指令使用指定的名称创建一个挂载点，持有外部挂载的本地主机或者其他容器的卷。其值可以为一个JSON数组，`VOLUMN ["/var/log"]`,或者一个字符串，例如`VOLUMN /var/log`或者`VOLUMN /var/log /var/db`.

docker run命令使用基本映像中指定位置存在的任何数据初始化新创建的卷。 例如，请考虑以下Dockerfile片段：

```
FROM ubuntu
RUN mkdir /myvol
RUN echo "hello world" > /myvol/greeting
VOLUME /myvol
```

此Dockerfile会生成一个映像，该映像会导致docker run在`/myvol`上创建新的挂载点，并将greeting文件复制到新创建的卷中。

关于指定卷的说明: 特别注意以下几点关于Dockerfile文件中的卷

-   基于windows容器的卷：当使用windows类型的容器时，容器中的卷需是一下中的一种：

```
 1. 不存在或空目录
 2. C以外的驱动器
```

-   从Dockerfile中更改卷:如果任何构建步骤在声明后更改卷内的数据，那么这些更改将被丢弃。
-   JSON形式：该列表被解析为JSON数组。 您必须用双引号（“）而不是单引号（'）括起来。
-   主机目录在容器运行时声明：主机目录（mountpoint）本质上是依赖于主机的。 这是为了保持图像的可移植性，因为不能保证给定的主机目录在所有主机上都可用。 因此，您无法从Dockerfile中安装主机目录。 VOLUME指令不支持指定host-dir参数。 您必须在创建或运行容器时指定安装点。

VOLUME指令应用于公开由docker容器创建的任何数据库存储区域，配置存储或文件/文件夹。 强烈建议您将VOLUME用于图像的任何可变和/或用户可维修部分。

### USER

```
USER <user>[:<group>] or
USER <UID>[:<GID>]
```

USER指令用于设置用户名或者UID，可选的用户组或者GID.当运行镜像的时候，以便在运行映像时以及Dockerfile中跟随它的任何RUN，CMD和ENTRYPOINT指令时使用。

在Windows上，如果用户不是内置帐户，则必须先创建用户。 这可以使用作为Dockerfile一部分调用的net user命令来完成。

```
    FROM microsoft/windowsservercore
    # Create Windows user in the container
    RUN net user /add patrick
    # Set it for subsequent commands
    USER patrick
```

### WORKDIR

`WORKDIR /path/to/workdir`

WORKDIR指令为Dockerfile中的任何RUN，CMD，ENTRYPOINT，COPY和ADD指令设置工作目录。如果WORKDIR 目录不存在，即使它未在任何后续Dockerfile指令中使用，也将创建它。WORKDIR指令可以在Dockerfile中多次使用。 如果提供了相对路径，则它将相对于先前WORKDIR指令的路径。 例如：

```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

此Dockerfile中最终`pwd`命令的输出为`/a/b/c`。 WORKDIR指令可以解析先前使用ENV设置的环境变量。 您只能使用Dockerfile中显式设置的环境变量。 例如：

```
ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```

此Dockerfile中最后一个`pwd`命令的输出将是`/path/$DIRNAME` 为了清晰和可靠，您应该始终使用WORKDIR的绝对路径。 此外，您应该使用WORKDIR而不是像`RUN CD ...&& do-something`，这些指令难以阅读，故障排除和维护。

### ONBUILD

在当前Dockerfile构建完成后执行ONBUILD命令。 ONBUILD在从当前图像派生的任何子图像中执行。 将ONBUILD命令视为父Dockerfile为子Dockerfile提供的指令。

Docker构建在子Dockerfile中的任何命令之前执行ONBUILD命令。

ONBUILD对于将从给定图像构建的图像非常有用。 例如，您可以使用ONBUILD作为语言堆栈映像，在Dockerfile中构建使用该语言编写的任意用户软件，如Ruby的ONBUILD变体中所示。

从ONBUILD构建的图像应该获得一个单独的标记，例如：ruby：1.9-onbuild或ruby：2.0-onbuild。

将ADD或COPY放入ONBUILD时要小心。 如果新构建的上下文缺少正在添加的资源，则“onbuild”映像将发生灾难性故障。 如上所述，添加单独的标记有助于通过允许Dockerfile作者做出选择来缓解这种情况。


**参考资料**
[Dockerfile的最佳实践-掘金](https://juejin.cn/post/6844903922830671885)
[Best practices for writing Dockerfiles | Docker Docs](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)