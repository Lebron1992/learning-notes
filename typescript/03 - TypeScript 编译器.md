# 03 - TypeScript 编译器

## 编译单个 `ts` 文件

命令：`tsc test.ts`

## 使用 Watch Mode

命令：`tsc test.ts -w`

## 编译整个项目或者多个 `ts` 文件

1. 在项目的根目录执行 `tsc --init`，生成 `tsconfig.json` 配置文件
2. 直接执行 `tsc` 就可以编译所有的 `ts` 文件；或者执行 `tsc -w` 使用 Watch Mode。

## 包含和移除某些文件

在 `tsconfig.json` 文件的 json 根节点下添加 `exclude` 可以在编译时跳过某些文件：

```
"exclude": [
  "node_modules",
  "**/*.dev.ts"
]
```

这样包含在数组中的文件将不会被编译。

如果我们只需要编译某些特定的文件，则可以添加 `include`。例如：

```
"include": [
  "app.ts"
]
```

**注意：** 添加了 `include` 之后，只有包含在 `include` 数组中的文件才会被编译，同时会跳过 `exclude` 的文件。

## 设置编译 Target

浏览 `tsconfig.json` 文件，可以看到 `compilerOptions` 节点下有一个字段 `target`，它就是用于指定 TypeScript 代码最终编译成哪个版本的 JavaScript 代码。

目前我们一般设置为 `es5`。

## `noEmitOnError`

如果想在 ts 文件编译出错时不生成对应的 js 文件，可以在 `compilerOptions` 节点下添加 `"noEmitOnError": true`，这样可以避免一些不必要的错误。

## 更多编译设置

- ![tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)

- ![Compiler Options](https://www.typescriptlang.org/docs/handbook/compiler-options.html)
