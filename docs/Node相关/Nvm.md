> `node` 版本管理器，也就是说：一个 `nvm` 可以管理多个 `node` 版本（包含 `npm` 与 `npx`），可以方便快捷的 安装、切换 不同版本的 `node`。

> 首先最重要的是：**一定要卸载已安装的NodeJS，否则会发生冲突**。 

## 安装`nvm`

### 安装方式一：`Mac HomeBrew` 安装

如果安装没问题，按照 `安装方式三` 中除安装部分外，配置好当前解释器需要的 `nvm` 配置。

```shell
brew install nvm
```

### 安装方式二：命令安装

下面的安装路径，在 [nvm 官方文档](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fnvm-sh%2Fnvm%2Fblob%2Fmaster%2FREADME.md) 中有。

```shell
# 复制代码
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

# 或
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

附带：（下面后面找到的替代方案，安装源都换成国内的，尤其是在服务器上进行安装时，网络不行的情况下，可以使用下面这个）

```shell
# 安装
$ bash -c "$(curl -fsSL https://gitee.com/RubyKids/nvm-cn/raw/master/install.sh)"

# 卸载
$ bash -c "$(curl -fsSL https://gitee.com/RubyKids/nvm-cn/raw/master/uninstall.sh)"
```

## 查看`nvm`版本

```shell
nvm -v
```



## 使用`nvm`

### 安装

- 安装最新稳定版

  ``` shell
  nvm install stable
  ```

- 安装指定版本

  ```shell
  nvm install <version>
  
  # 安装v16版本
  nvm install v16
  nvm install 16 # 默认安装16的最新版本
  nvm install V14.0.0 # 安装指定的14.0.0版本
  ```

### 删除

```shell
nvm uninstall <version>
```

### 切换版本

```shell
# 临时版本 - 只在当前窗口生效指定版本
$ nvm use <version>

# 永久版本 - 所有窗口生效指定版本 需要关闭重新打开终端
$ nvm alias default <version>
```

- 列出所有安装的版本

  ```shell
  nvm list
  
  # 或
  
  nvm ls
  ```

- 列出所有远程服务器的版本（官方 `node version list`）

  ```shell
  nvm ls-remote
  ```

- 显示当前的版本

  ```shell
  nvm current
  ```

- 给不同的版本号添加别名

  ```shell
  nvm alias <name> <version>
  ```

- 删除已定义的别名

  ```shell
  nvm unalias <name>
  ```

  
