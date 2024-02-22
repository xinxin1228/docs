## 持久化缓存

缓存是一种应用非常广泛性能优化技术，在计算机领域几乎无处不在，例如：操作系统层面 CPU 高速缓存、磁盘缓存，网路世界中的 DNS 缓存、HTTP 缓存，以及业务应用中的数据库缓存、分布式缓存等等。

那自然而然的，我们也可以在 Webpack 使用各式各样的缓存技术，通过牺牲空间来提升构建过程的时间效率。

需要在 Webpack5 中设置 `cache.type = 'filesystem'` 即可开启：

```js
// webpack.config.js
module.exports = {
    //...
    cache: {
        type: 'filesystem'
    },
}
```

