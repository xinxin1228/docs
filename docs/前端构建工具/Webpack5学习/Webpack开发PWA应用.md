> 本节完整的完整代码地址：https://github.com/xinxin1228/webpack-learning/tree/main/07-pwa%E5%BC%80%E5%8F%91

`PWA` 全称 `Progressive Web Apps` (渐进式 Web 应用)，原始定义很复杂，可以简单理解为 **一系列将网页如同独立 APP 般安装到本地的技术集合**，借此，我们即可以保留普通网页轻量级、可链接(SEO 友好)、低门槛（只要有浏览器就能访问）等优秀特点，又同时具备独立 APP 离线运行、可安装等优势。

实现上，PWA 与普通 Web 应用的开发方法大致相同，都是用 CSS、JS、HTML 定义应用的样式、逻辑、结构，两者主要区别在于，PWA 需要用一些新技术实现离线与安装功能：

- `ServiceWorker`： 可以理解为一种介于网页与服务器之间的本地代理，主要实现 PWA 应用的离线运行功能。例如 `ServiceWorker` 可以将页面静态资源缓存到本地，用户再次运行页面访问这些资源时，`ServiceWorker` 可拦截这些请求并直接返回缓存副本，即使此时用户处于离线状态也能正常使用页面。
- `manifest` 文件：描述 PWA 应用信息的 `JSON` 格式文件，用于实现本地安装功能，通常包含应用名、图标、URL 等内容，例如：

```json
// manifest.json
{
  "icons": [
    {
      "src": "/icon_120x120.0ce9b3dd087d6df6e196cacebf79eccf.png",
      "sizes": "120x120",
      "type": "image/png"
    }
  ],
  "name": "My Progressive Web App",
  "short_name": "MyPWA",
  "display": "standalone",
  "start_url": ".",
  "description": "My awesome Progressive Web App!"
}
```

我们可以选择自行开发、维护 `ServiceWorker` 及 `manifest` 文件 ，也可以简单点使用 Google 开源的 Workbox 套件自动生成 PWA 应用的壳，首先安装依赖：

```shell
$ pnpm add workbox-webpack-plugin webpack-pwa-manifest -D
```

其中：

- `workbox-webpack-plugin`：用于自动生成 `ServiceWorker` 代码的 Webpack 插件；
- `webpack-pwa-mainifest`：根据 Webpack 编译结果，自动生成 PWA Manifest 文件的 Webpack 插件。

之后，在 `webpack.config.js` 配置文件中注册插件：

```js
// webpack.config.js
module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, 'public/index.html'),
    }),
    // 自动生成 Manifest 文件
    new WebpackPwaManifest({
      name: 'My Progressive Web App',
      short_name: 'MyPWA',
      description: 'My awesome Progressive Web App!',
      publicPath: '.',
      icons: [
        {
          // 桌面图标，注意这里只支持 PNG、JPG、BMP 格式
          src: path.resolve('public/logo.png'),
          sizes: [150],
        },
      ],
    }),
    // 自动生成 ServiceWorker 文件
    new GenerateSW({
      clientsClaim: true,
      skipWaiting: true,
    }),
  ],
}

```

然后在 `src/index.js` 中注册：

```js
// src/index.js
if ('serviceWorker' in navigator) {
  window.addEventListener('load', async () => {
    console.log('page load...')
    let res = await navigator.serviceWorker.register('/service-worker.js')
    console.log(res, 'serviceWorker res')
    if (res) {
      console.log('register success!')
    } else {
      console.log('register fail!')
    }
  })
}
```

