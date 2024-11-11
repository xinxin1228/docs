## Dockerfile介绍

`Dockerfile` 是用于**构建** `Docker` **镜像**的文本文件，其中包含了一系列的指令和配置。`Docker` 镜像是用于运行 `Docker` 容器的轻量级、可移植的打包格式。`Dockerfile` 用于定义 `Docker` 镜像的内容、环境和运行时配置。

### Dockerfile 的作用和用途：

1. **定义基础镜像：** `Dockerfile` 允许你选择一个已有的基础镜像，如官方的 `Ubuntu`、`Alpine`、`Node.js` 等，作为构建的起点。

2. **指定构建步骤：** `Dockerfile` 中包含一系列指令，每个指令代表构建步骤中的一个操作，例如安装软件包、拷贝文件、设置环境变量等。

3. **创建自定义应用镜像：** 使用 `Dockerfile`，你可以定义自己的应用程序所需的环境和依赖，并将其打包成一个独立的、可运行的镜像。

4. **提高可重复性：** `Dockerfile` 提供了一种文本化的描述方式，能够使镜像的构建过程更加可重复，从而确保在不同环境中相同的 `Dockerfile` 会产生相同的镜像。

### Dockerfile 示例：

写一个`Dockerfile`示例，运行一个项目 前端是`react`编写的 后端是`node`编写的，文件结构类似于，后端项目在根目录上 前端项目在 文件夹`/client`中，运行的时候需要分别安装依赖，并且需要先讲前端进行打包，进入`client`目录 运行 `npm run build`，然后在根目录运行 `npm run start`  设置内部端口 `3000` 对外暴露端口`5173` 。

项目文件结构如下：

```shell
├── # 根目录，后端项目
│   ├── client # 前端项目
│   ├── package.json
```

编写的`Dockerfile`如下：

```dockerfile
# 使用 Node.js 做为基础镜像
FROM node:20.11.0-buster-slim

# 设置工作区
WORKDIR /app

# 复制后端代码到工作区
COPY . /app

# 安装前端和后端依赖，并构建前端代码，安装后端依赖 这里使用 --prefix 避免重复进入client和退出client
RUN npm install --prefix client && npm run build --prefix client && npm install

# 暴露内部端口
EXPOSE 3000

# 设置环境变量，指定项目运行的端口
ENV PORT=3000

# 启动项目
CMD ["npm", "run", "start"]
```

#### 构建镜像：

```shell
# 从当前目录下的 Dockerfile 构建一个名为 my-app 的 Docker 镜像
$ docker build -t my-app .
```

#### 运行容器：

```shell
# 后台运行该镜像的容器 并且对外暴露5173端口
$ dcoker container run -p 5173:3000 -d my-app
```

## 基础镜像的选择

- 官方镜像优于非官方的镜像，如果没有官方镜像，则尽量选择Dockerfile开源的

- 固定版本tag而不是每次都使用latest

- 尽量选择体积小的镜像

## Run命令

`RUN` 主要用于在`Image`里执行指令，比如安装软件，下载文件等。

```shell
$ apt-get update
$ apt-get install wget
$ wget https://github.com/ipinfo/cli/releases/download/ipinfo-2.0.1/ipinfo_2.0.1_linux_amd64.tar.gz
$ tar zxf ipinfo_2.0.1_linux_amd64.tar.gz
$ mv ipinfo_2.0.1_linux_amd64 /usr/bin/ipinfo
$ rm -rf ipinfo_2.0.1_linux_amd64.tar.gz
```

`Dockerfile` 的写法如下：

```dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y wget
RUN wget https://github.com/ipinfo/cli/releases/download/ipinfo-2.0.1/ipinfo_2.0.1_linux_amd64.tar.gz
RUN tar zxf ipinfo_2.0.1_linux_amd64.tar.gz
RUN mv ipinfo_2.0.1_linux_amd64 /usr/bin/ipinfo
RUN rm -rf ipinfo_2.0.1_linux_amd64.tar.gz
```

镜像的大小和分层

```shell
$ docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
ipinfo       latest    97bb429363fb   4 minutes ago   138MB
ubuntu       21.04     478aa0080b60   4 days ago      74.1MB
$ docker image history 97b
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
97bb429363fb   4 minutes ago   RUN /bin/sh -c rm -rf ipinfo_2.0.1_linux_amd…   0B        buildkit.dockerfile.v0
<missing>      4 minutes ago   RUN /bin/sh -c mv ipinfo_2.0.1_linux_amd64 /…   9.36MB    buildkit.dockerfile.v0
<missing>      4 minutes ago   RUN /bin/sh -c tar zxf ipinfo_2.0.1_linux_am…   9.36MB    buildkit.dockerfile.v0
<missing>      4 minutes ago   RUN /bin/sh -c wget https://github.com/ipinf…   4.85MB    buildkit.dockerfile.v0
<missing>      4 minutes ago   RUN /bin/sh -c apt-get install -y wget # bui…   7.58MB    buildkit.dockerfile.v0
<missing>      4 minutes ago   RUN /bin/sh -c apt-get update # buildkit        33MB      buildkit.dockerfile.v0
<missing>      4 days ago      /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      4 days ago      /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B
<missing>      4 days ago      /bin/sh -c [ -z "$(apt-get indextargets)" ]     0B
<missing>      4 days ago      /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   811B
<missing>      4 days ago      /bin/sh -c #(nop) ADD file:d6b6ba642344138dc…   74.1MB
```

每一行的`RUN`命令都会产生一层`image layer`, 导致镜像的臃肿。

### 改进版Dockerfile

```dockerfile
FROM ubuntu:20.04
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/ipinfo/cli/releases/download/ipinfo-2.0.1/ipinfo_2.0.1_linux_amd64.tar.gz && \
    tar zxf ipinfo_2.0.1_linux_amd64.tar.gz && \
    mv ipinfo_2.0.1_linux_amd64 /usr/bin/ipinfo && \
    rm -rf ipinfo_2.0.1_linux_amd64.tar.gz
```

## (ADD,COPY,WORKDIR)命令

往镜像里复制文件有两种方式，`COPY` 和 `ADD` , 我们来看一下两者的不同。

### 复制普通文件

`COPY` 和 `ADD` 都可以把local的一个文件复制到镜像里，如果目标目录不存在，则会自动创建

```dockerfile
FROM python:3.9.5-alpine3.13
COPY hello.py /app/hello.py
```

比如把本地的 `hello.py` 复制到 `/app` 目录下。 `/app`这个`folder`不存在，则会自动创建

### 复制压缩文件

`ADD` 比 `COPY`高级一点的地方就是，如果复制的是一个`gzip`等压缩文件时，`ADD`会帮助我们自动去解压缩文件。

```dockerfile
FROM python:3.9.5-alpine3.13
ADD hello.tar.gz /app/
```

### 如何选择

因此在 `COPY` 和 `ADD` 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 `COPY` 指令，仅在需要自动解压缩的场合使用 `ADD`。

### WORKDIR命令

当执行了`WORKDIR /app`后，该命令有以下效果：

1. **构建时**：从该指令之后的 `COPY`、`ADD` 或者 `RUN` 指令将相对于 `WORKDIR` 设置的路径来执行。例如，如果设置了 `WORKDIR /app`，然后执行 `COPY . .`，则会将Dockerfile所在目录下的所有内容复制到镜像内的 `/app` 目录下。
2. **运行时**：当基于这个镜像创建并启动一个容器时，容器的默认工作目录也会是 `WORKDIR` 指定的目录。这意味着如果你通过 `docker run -it image_name sh` 进入容器并执行 shell，那么你的初始位置将是 `WORKDIR` 设定的路径。

简而言之，`WORKDIR` 提供了一个在构建和运行阶段都适用的基础目录，方便了文件操作和程序运行环境的统一管理。多个 `WORKDIR` 指令可以连续使用，后续的 `WORKDIR` 将基于前一个 `WORKDIR` 的基础上进行相对路径变更。

## 构建参数和环境变量 (ARG vs ENV)

### ENV

**用法：**

```dockerfile
FROM ubuntu:20.04
ENV VERSION=2.0.1
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/ipinfo/cli/releases/download/ipinfo-${VERSION}/ipinfo_${VERSION}_linux_amd64.tar.gz && \
    tar zxf ipinfo_${VERSION}_linux_amd64.tar.gz && \
    mv ipinfo_${VERSION}_linux_amd64 /usr/bin/ipinfo && \
    rm -rf ipinfo_${VERSION}_linux_amd64.tar.gz
```

**作用：**

- `ENV` 指令用于设置镜像构建过程中以及容器运行时的环境变量。
- 使用 `ENV` 设置的环境变量会被添加到镜像中，并且在基于该镜像创建的所有容器中默认存在。
- 一旦镜像构建完成，通过 `ENV` 设置的环境变量值不可更改（除非在运行容器时使用 `-e` 或 `--env-file` 参数覆盖）。
- 主要用于持久化配置镜像和容器运行环境。

### ARG

**使用：**

```dockerfile
FROM ubuntu:20.04
ARG VERSION=2.0.1
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/ipinfo/cli/releases/download/ipinfo-${VERSION}/ipinfo_${VERSION}_linux_amd64.tar.gz && \
    tar zxf ipinfo_${VERSION}_linux_amd64.tar.gz && \
    mv ipinfo_${VERSION}_linux_amd64 /usr/bin/ipinfo && \
    rm -rf ipinfo_${VERSION}_linux_amd64.tar.gz
```

**作用：**

- `ARG` 指令同样用于定义环境变量，但它主要用于镜像构建阶段，为构建过程提供可变参数。
- 在构建镜像时，可以通过 `docker build --build-arg <variable>=<value>` 的方式从命令行传入 `ARG` 变量的具体值。
- 构建完成后，`ARG` 定义的环境变量不会保留在镜像中。这意味着当你运行容器时，不能访问到在构建时用 `ARG` 设置的环境变量，除非在Dockerfile中将这些变量值通过 `ENV` 指令保存下来。
- `ARG` 常用于动态指定依赖版本、API密钥等敏感信息，这些信息仅在构建时需要，不应该存储在最终镜像内。
- 在镜像构建阶段临时使用的可变参数，有助于灵活控制构建流程并保护敏感信息不被泄露

## 容器启动命令 CMD

`CMD`可以用来设置容器启动时默认会执行的命令。

- 容器启动时默认执行的命令
- 如果`docker container run`启动容器时指定了其它命令，则CMD命令会被忽略
- 如果定义了多个`CMD`，只有**最后一个**会被执行。

## 容器启动命令 ENTRYPOINT

- `CMD` 设置的命令，可以在 `docker container run` 时传入其它命令，覆盖掉 `CMD` 的命令，但是 `ENTRYPOINT` 所设置的命令是一定会被执行的。
- `ENTRYPOINT` 和 `CMD` 可以联合使用，`ENTRYPOINT` 设置执行的命令，CMD传递参数

在本地编写两个 `Dockerfile` 文件，一个启动命令采用 `CMD` 方式，另一个启动命令采用 `ENTRYPOINT`。

<!-- tabs:start -->

#### **Dockerfile.cmd**

```dockerfile
FROM ubuntu:18.04

CMD [ "echo", "Hello docker" ]
```

构建镜像：

```shell
$ docker build -f Dockerfile.cmd -t demo-cmd .
```

在这个命令中：

- `-f Dockerfile.cmd ` 表示使用名为 `Dockerfile.cmd ` 的文件作为构建镜像的 Dockerfile。
- `-t demo-cmd` 是指定了构建完成后镜像的名称。
- `.` 表示构建上下文是当前目录，Docker 会查找 `-f` 参数指定的 Dockerfile 并在此目录及其子目录中查找需要的文件。

执行创建容器，不指定运行时的命令，则会默认执行CMD所定义的命令，打印出 `hello docker`。

```shell
$ docker container run -it --rm demo-cmd
hello docker
```

在这个命令中：

- `-it`: 用于在启动容器时指定容器运行的交互模式。
- `--rm`: 这个标志告诉 Docker 在容器退出后自动删除（remove）容器。这对于一次性执行任务或测试场景非常有用，避免产生大量不再需要的停止状态的容器。

但是如果我们 `docker container run` 的时候指定命令，则该命令会覆盖掉 CMD 的命令:

```shell
$ docker run -it --rm demo-cmd echo "hello world"
hello world
```



#### **Dockerfile.entrypoint**

```dockerfile
FROM ubuntu:18.04

ENTRYPOINT [ "echo", "Hello docker" ]
```

构建镜像：

```shell
$ docker build -f Dockerfile.entrypoint -t demo-entrypoint .
```

在这个命令中：

- `-f Dockerfile.entrypoint ` 表示使用名为 `Dockerfile.entrypoint ` 的文件作为构建镜像的 Dockerfile。
- `-t demo-entrypoint` 是指定了构建完成后镜像的名称。
- `.` 表示构建上下文是当前目录，Docker 会查找 `-f` 参数指定的 Dockerfile 并在此目录及其子目录中查找需要的文件。

执行创建容器，不指定运行时的命令，则会默认执行 ENTRYPOINT 所定义的命令，打印出 `hello docker`。

```shell
$ docker container run -it --rm demo-entrypoint
hello docker
```

在这个命令中：

- `-it`: 用于在启动容器时指定容器运行的交互模式。
- `--rm`: 这个标志告诉 Docker 在容器退出后自动删除（remove）容器。这对于一次性执行任务或测试场景非常有用，避免产生大量不再需要的停止状态的容器。

但是如果我们 `docker container run` 的时候指定命令，ENTRYPOINT的容器里ENTRYPOINT所定义的命令则无法覆盖，一定会执行。

```shell
$ docker run -it --rm demo-cmd echo "hello world"
hello docker echo hello world
```



<!-- tabs:end -->

## 编写 .Dockerignore

 `.dockerignore`  文件的作用是定义在构建 Docker 镜像时，哪些文件或目录不应该包含在构建上下文中，也就是说，这些被忽略的文件不会被发送到 Docker 守护进程去构建镜像。

`.dockerignore` 的用法与 `.gitignore` 类似。

```.dockerignore
node_modules
dist
.git
.env*
```

上面的例子中，`.dockerignore`文件指定了：

- 忽略 `node_modules` 目录下的所有内容
- 忽略 `dist` 目录下的所有内容
- 忽略`.git`目录下的所有内容
- 忽略所有以`.env`开头的文件
