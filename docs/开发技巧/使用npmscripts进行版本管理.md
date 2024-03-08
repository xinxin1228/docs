每次构建完的代码都应该有新的版本号，修改版本号直接使用 `npm` 内置的 `version` 自命令即可，可以在 `package.json` 中添加如下 `scripts`：

```json
{
  // ...
  "scripts": {
    "release:patch": "npm version patch && git push && git push --tags",
    "release:minor": "npm version minor && git push && git push --tags",
    "release:major": "npm version major && git push && git push --tags"
  }
}
```

在上面的 `package.json` 中，定义了三个脚本：

- `npm run release:patch` 会将版本号自动递增补丁版本（例如从 1.0.0 变为 1.0.1）。
- `npm run release:minor` 会将版本号自动递增次要版本（例如从 1.0.0 变为 1.1.0）。
- `npm run release:major` 会将版本号自动递增主版本（例如从 1.0.0 变为 2.0.0）。

`npm version` 命令不仅会更新 `package.json` 中的版本号，还会自动创建一个新的 `git` 版本标签，并提交相关改动到 `git` 仓库。此外，还可以根据需要在此基础上添加更多自动化步骤，比如发布到 `npm registry` 等。
