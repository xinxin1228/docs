默认情况下，容器内创建的所有文件都存储在可写容器层上。这意味着：

当该容器不再存在时，数据不会持久存在，并且如果另一个进程需要数据，则可能很难从容器中获取数据。
容器的可写层与运行容器的主机紧密耦合。无法轻松地将数据移动到其他地方。

Docker 有两个选项让容器在主机上存储文件，以便即使在容器停止后文件也能保留：卷存储（`volume`）和绑定挂载（`bind`）。

卷存储在主机文件系统的一部分中，*由 Docker 管理*（在 Linux 上的存储位置是 `/var/lib/docker/volumes/`）。非 Docker 进程不应修改文件系统的这一部分。卷是在 Docker 中保存数据的最佳方式。

绑定安装可以存储在主机系统上的任何位置。它们甚至可能是重要的系统文件或目录。Docker 主机或 Docker 容器上的非 Docker 进程可以随时修改它们。

## Volume

### 概述

卷是持久保存 Docker 容器生成和使用的数据的首选机制。虽然绑定挂载取决于主机的目录结构和操作系统，但卷完全由 Docker 管理。与绑定挂载相比，卷有几个优点：

- 卷比绑定安装更容易备份或迁移。
- 可以使用 `Docker CLI` 命令或 `Docker API` 管理卷。
- 卷适用于 `Linux` 和 `Windows` 容器。
- 卷可以在多个容器之间更安全地共享。
- 卷驱动程序允许将卷存储在远程主机或云提供商上、加密卷的内容或添加其他功能。
- 新卷的内容可以由容器预先填充。
- `Docker Desktop` 上的卷比 `Mac` 和 `Windows` 主机上的绑定挂载具有更高的性能。

此外，卷通常是比将数据保留在容器的可写层中更好的选择，因为卷不会增加使用它的容器的大小，并且卷的内容存在于给定容器的生命周期之外。

### 在命令行中使用

> 使用 `-v` 或 `--mount` 标志。

一般来说，`--mount`更加明确和详细。最大的区别在于`-v`语法将所有选项组合在一个字段中，而`--mount` 语法则将它们分开。

#### `-v` 参数

`-v` 或 `--volume` 参数的基本格式如下：

```shell
$ docker run -v <主机路径>:<容器路径>[:<模式>] <image>
```

- `<主机路径>`：这是主机上的绝对路径，指定要挂载到容器内的主机目录或数据卷。
- `<容器路径>`：这是容器内部的路径，挂载点。
- `[:<模式>]`：可选参数，用于指定挂载权限模式（默认为读写权限，即 `rw`）。也可以指定为 `ro` 表示只读。

例如：

```shell
$ docker run -v $(pwd):/container/directory ubuntu:latest
```

在此情况下，主机上的当前命令行所在目录将被挂载到容器的 `/container/directory`。如果主机路径不存在，Docker 会自动创建一个空目录。

使用 `-v` 参数创建匿名卷时，只需指定容器内的挂载路径即可，宿主机的路径会被忽略或者由 Docker 自动生成一个唯一的名字和路径。

匿名卷的基本用法如下：

```shell
$ docker run -v <容器内路径>[:<模式>] <image>
```

这里省略了宿主机路径部分，仅指定了容器内的挂载点：

```shell
$ docker run -v /container/path/to/mount ubuntu:latest
```

在这个例子中，Docker 会创建一个新的匿名数据卷，并将其挂载到容器内部的 `/container/path/to/mount` 路径下。

#### `--mount` 参数

`--mount` 参数提供了更详细的挂载选项，并且采用键值对的方式指定挂载细节：

```shell
$ docker run --mount type=<类型>,source=<主机路径>,target=<容器路径>[,<选项列表>] <image>
```

- `type`：挂载类型，常见的是 `bind`（用于挂载主机目录）和 `volume`（用于挂载数据卷）。
- `source`：对应 `-v` 的 `<主机路径>`。可以指定为`source` 或`src`。
- `target`：对应 `-v` 的 `<容器路径>`。可以指定为`destination`、`dst`、 或`target`。
- `<选项列表>`：可选的额外挂载选项，例如 `readonly` 表示只读，还可以指定 `consistency` 属性等。

#### 区别总结：

1. `-v` 参数比较简洁，适合简单的主机目录挂载或数据卷挂载操作。
2. `--mount` 参数更加灵活和明确，它可以提供更多挂载选项，并且强制要求主机路径必须存在。
3. 当需要指定挂载选项如 `readonly` 或者使用特殊类型的挂载时（比如 tmpfs、nfs 等），必须使用 `--mount` 参数。
4. `-v` 和 `--mount` 在功能上是等效的，都可以实现数据卷的挂载，只是语法和灵活性不同。

因此，在选择使用 `-v` 或 `--mount` 时，可以根据具体需求和场景来决定。在新版 Docker 中，推荐优先使用 `--mount` 语法，因为它更具扩展性和易读性。

#### 案例说明

启动一个带有卷的容器

如果您使用尚不存在的卷启动容器，Docker 会为您创建该卷。以下示例将卷安装`myvol2`到 `/app/`容器中。

<!-- tabs:start -->

#### **--mount**

```shell
$ docker run -d \
--name mount-container \
--mount type=volume,source=myvol2,target=/app \
nginx:latest
```

#### **-v**

```shell
$ docker run -d \
--name volume-container \
-v myvol2:/app \
nginx:latest
```

<!-- tabs:end -->

### 在 Docker Compose 中使用

<!-- tabs:start -->

#### **内部创建**

第一次运行 `docker compose up` 时会创建一个卷，之后再次运行时会重复使用相同的卷。

```yaml
services:
  frontend:
    image: node:20-alpine
    volumes:
      - myapp:/home/node/app
volumes:
  myapp:
```

#### **外部创建**

直接在 Compose 外部创建一个卷`docker volume create myapp`，然后在内部引用它，`compose.yaml`如下所示：

```yaml
services:
  frontend:
    image: node:20-alpine
    volumes:
      - myapp:/home/node/app
volumes:
  myapp:
    external: true # 表示使用的是外部的 而不是内部创建
```

<!-- tabs:end -->

## Bind mount

### 概述

与卷相比，绑定安装的功能有限。使用绑定挂载时，主机上的文件或目录将挂载到容器中。文件或目录由其在主机上的完整路径引用。该文件或目录不需要已存在于 Docker 主机上。如果尚不存在，则会根据需要创建它。

### 在命令行中使用

假设我们要将当前目录挂载在容器的 `/app` 中，可以使用 `$(pwd)` 在 `Linux` 和 `macOS` 上表示当前的工作目录。

> 注意：`--mount` 下面的示例 `-v` 产生相同的结果。除非在运行第一个容器后删除容器，否则您无法同时运行它们。

<!-- tabs:start -->

#### **--mount**

```shell
$ docker run -d \
--name mount-container \
--mount type=bind,source=$(pwd),target=/app \
nginx:latest
```

#### **-v**

```shell
$ docker run -d \
--name mount-container \
-v $(pwd):/app \
nginx:latest
```

<!-- tabs:end -->

### 在 Docker Compose 中使用

<!-- tabs:start -->

#### **写法一**

```yaml
services:
  frontend:
    image: node:20-alpine
    volumes:
      - ./:/home/node/app
```

#### **写法二**

```yaml
services:
  frontend:
    image: node:20-alpine
    volumes:
      - type: bind
        source: ./
        target: /home/node/app
```



<!-- tabs:end -->