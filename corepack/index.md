# Corepack

[Corepack](https://nodejs.org/api/corepack.html) 是一个实验性工具，用于帮助管理您的包管理器的版本。它为每个支持的包管理器（Yarn:yarn/yarnpkg, pnpm:pnpm/pnpx）暴露了二进制代理，当被调用时，将识别当前项目配置的任何包管理器，如果需要，将其下载，并最终运行它。

此功能简化了两个核心工作流程：
- 它简化了新贡献者的入职流程，因为他们不再需要遵循特定于系统的安装过程，只需使用您希望他们使用的包管理器。
- 它确保您团队中的每个人都将使用您希望他们使用的确切包管理器版本，而无需每次更新时手动同步。

## Workflows

### Enable the feature

由于其实验性状态，Corepack 目前需要明确启用才能生效。要做到这一点，请运行 `corepack enable`，这将会在您的环境中设置与 node 二进制文件相邻的符号链接（并在必要时覆盖现有的符号链接）。

### Configuring a package

Corepack 代理将会在您当前的目录层次结构中找到最近的 `package.json` 文件，以提取其 `packageManager` 属性。

如果该值对应于受支持的包管理器，Corepack 将确保所有对相关二进制文件的调用都针对请求的版本运行，如有需要，将按需下载，并在无法成功检索时中止。

指定包管理器:
  
```bash
corepack use pnpm@7.x # sets the latest 7.x version in the package.json
corepack use yarn@* # sets the latest version in the package.json 
```

### Running npm install -g yarn doesn't work

npm 在进行全局安装时会防止意外覆盖 Corepack 的二进制文件。为避免此问题，请考虑以下选项之一：

- 不要运行此命令；Corepack 会提供包管理器的二进制文件，并确保所请求的版本始终可用，因此不需要显式安装包管理器。
- 在 `npm install` 中添加 `--force` 标志；这将告诉 npm 可以覆盖二进制文件，但您将在此过程中删除 Corepack 的二进制文件。（运行 `corepack enable` 以将它们添加回来。）

```bash
corepack enable
corepack yarn install
```
