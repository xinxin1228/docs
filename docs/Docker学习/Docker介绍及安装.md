## Docker介绍

`Docker` 是一个开源的应用容器引擎，它通过操作系统级别的虚拟化技术实现了轻量级的容器（`container`）解决方案。`Docker` 允许开发者打包应用及其依赖环境（如库、配置文件等）到一个可移植的容器镜像中，这样构建好的应用可以在任何安装了 `Docker` 的环境下运行，而无需关心底层操作系统的差异。

以下是一些关于 `Docker` 的关键特性：

1. **容器化**：`Docker` 容器不是传统的虚拟机，而是共享主机内核的隔离进程空间，因此容器相比于虚拟机更加轻量化、启动更快且资源占用更少。
2. **镜像**：`Docker` 镜像是不可变的，基于分层存储机制，包含应用程序运行所需的所有内容，包括代码、运行时、系统工具、库和设置等。
3. **仓库**：`Docker Hub` 是官方提供的公共镜像仓库，用户也可以搭建私有仓库，如 `Docker Registry`，用于存储和分发自定义的 `Docker` 镜像。
4. **标准化部署**：通过 `Dockerfile` 可以描述如何构建一个 `Docker` 镜像，并可以使用这个文件在任何地方重现相同的部署环境。
5. **资源隔离与限制**：`Docker` 使用 `Linux` 内核的功能如 `namespaces`、`cgroups` 以及其他安全增强功能来实现容器间的资源隔离与配额管理。
6. **便携性**：`Docker` 容器可以在开发、测试、生产环境中无缝迁移，大大简化了 `DevOps` 流程。
7. **生态系统**：围绕 `Docker` 已经形成了庞大的生态系统，包括各种工具、插件和服务，支持自动化运维、持续集成/持续部署(`CI/CD`)等多种应用场景。

需要注意的是：

容器（`container`）是指的一种技术，而`Docker`只是一个容器技术的实现。因此：

`容器 != Docker`

`Docker`只是容器技术的一种实现，目前可以是说一种非常成功的实现。

## 镜像与容器的关系

## image镜像

- Docker image是一个 `read-only` 文件
- 这个文件包含文件系统，源码，库文件，依赖，工具等一些运行application所需要的文件
- 可以理解成一个模板
- `docker image`具有分层的概念

## container容器

- “一个运行中的docker image”
- 实质是复制`image`并在`image`最上层加上一层 `read-write` 的层 （称之为 `container layer` ,容器层）
- 基于同一个`image`可以创建多个`container`

## Docker安装

进入[Docker官网](https://www.docker.com/)后，点击下载即可安装。

`Windows`用户如果没有安装WSL2的话，请选择 `Hyper-V backend and Windows containers`。