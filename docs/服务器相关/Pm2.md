## pm2运行script脚本命令

```shell
# 运行脚本启动项目并且命名为app1
pm2 start "npm run dev" --name app1
```

## 重启

### 重启应用

```shell
pm2 restart app
```

### 重启全部

```shell
pm2 restart all
```

## 停止

### 停止应用

```shell
pm2 stop app
```

### 停止全部应用

```shell
pm2 stop all
```

## 删除

### 删除应用

```shell
pm2 delete app
```

### 删除全部应用

```shell
pm2 delete all
```

## 显示所有应用

```shell
pm2 list
```

