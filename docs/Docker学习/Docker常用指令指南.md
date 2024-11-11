## docker命令行的基本使用

`docker` + 管理的对象（比如容器，镜像） + 具体操作（比如创建，启动，停止，删除）

例如

- `docker image pull nginx` 拉取一个叫`nginx`的`docker image`镜像
- `docker container stop web` 停止一个叫`web`的`docker container`容器

## 镜像操作：

- `docker images` 或 `docker image ls`：列出所有本地镜像。
- `docker rmi` 或 `docker image rm`：删除镜像。
- `docker build`：从Dockerfile构建镜像。
- `docker tag`：为镜像添加标签。
- `docker pull`：从Docker Hub或其他注册服务器拉取镜像。
- `docker push`：将镜像推送到Docker Hub或其他注册服务器。
- `docker image prune -f`：删除所有的空悬镜像，即那些没有被任何容器引用的中间层镜像，如果不加 `-f` 参数，Docker 会询问用户是否确认删除。

## 容器操作：

- `docker run`或` docker container run`：运行一个新的容器。
- `docker start`：启动已停止的容器。
- `docker stop`或` docker container stop`：停止正在运行的容器。
- `docker rm` 或 `docker container rm`：删除容器。
- `docker exec`：在运行中的容器中执行命令。
- `docker logs` 或 `docker container logs`：获取容器的日志输出，摁下 `Ctrl + c` 退出。
- `docker logs -f <id>` ：实时跟随的容器的所有新产生的日志。
- `docker ps` 或 `docker container ls`：列出当前正在运行的容器。
- `docker ps -a`或`docker container ls -a`：列出当前所有的容器，包含已暂停的。
- `docker inspect` 或 `docker container inspect`：查看容器详细信息。
- `docker system prune -f`：删除所有已停止的容器，删除所有的构建缓存，用于清理（prune）系统中不必要的资源以释放磁盘空间。`-f` 或 `--force` 参数是强制执行的意思，不询问用户确认。

## 数据卷管理：

- `docker volume create`：创建新的数据卷。
- `docker volume ls`：列出所有数据卷。
- `docker volume rm` 或 `docker volume prune`：删除数据卷。

## 网络管理：

- `docker network create`：创建新的网络。
- `docker network ls`：列出所有网络。
- `docker network connect`：将容器连接到网络。
- `docker network disconnect`：将容器从网络断开。

## 系统维护与清理：

- `docker system df`：查看磁盘使用情况。
- `docker system prune`：清理未使用的资源（镜像、容器等）。

## 定义和运行多容器：

- `docker compose up [-d]` 根据**当前目录** `compose.yaml` 或 `docker-compose.yaml` 文件启动并在后台运行服务。
- `docker-compose down`：关闭和移除当前目录下的服务。
- `docker-compose start`：启动已经在停止状态的服务（不新建容器）。
- `docker-compose stop`：停止正在运行服务的容器。
- `docker-compose restart`：重启所有服务的容器。
- `docker-compose logs [-f]`：查看服务的输出日志，`-f` 参数表示实时输出（跟随）日志。

## 其他：

- `docker login`：登录到Docker注册服务器。
- `docker logout`：从Docker注册服务器登出。

