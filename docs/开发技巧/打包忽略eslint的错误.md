# 打包忽略 eslint 的错误

> 有些时候我们进行项目打包时，通常情况下，我们需要先进行`eslint`校验。

一般的情况下，我们无需忽略`eslint`错误，而是应该修复`eslint`的错误。但是对于那些`非eslint`的`eslint`错误，我们可以采用下面的方式进行忽略。

## 在 webpack 下忽略

```js
module.exports = {
  // ...其他配置
  plugins: [
    // ...其他插件
    new webpack.LoaderOptionsPlugin({
      options: {
        eslint: {
          enable: true, // 启用 ESLint
          mode: 'file', // 检查文件而不是代码块
          configure: {
            rules: {
              // 配置您的 ESLint 规则
            },
          },
          loaderOptions: {
            eslintOptions: {
              emitWarning: true,
            },
          },
        },
      },
    }),
    // ...其他插件
  ],
  // ...其他配置
}
```

## 在 @craco/craco 下忽略

```js
module.exports = {
  eslint: {
    enable: true, // 启用 ESLint
    mode: 'file', // 检查文件而不是代码块
    configure: {
      rules: {
        // 配置您的 ESLint 规则
      },
    },
    loaderOptions: eslintOptions => {
      eslintOptions.emitWarning = true // 忽略警告
      return eslintOptions
    },
  },
  // ...其他配置
}
```

## 在 vite 下

```js
import { defineConfig } from 'vite'

export default defineConfig({
  // ...其他配置
  eslintOptions: {
    // 配置您的规则
    rules: {
      'no-warning-comments': 'warn',
    },
  },
  // ...其他配置
})
```
