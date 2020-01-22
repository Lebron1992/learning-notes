# 08 - 在 TypeScript 中使用第三方库

## JavaScript 库

在 TypeScript 中使用 JavaScript 库，例如 ![lodash](https://lodash.com/)，如果直接 import 会报错：`import _ from 'lodash';`。实际上代码是可以运行的，但是在 TypeScript 中会报错。

安装 ![@types/lodash](https://www.npmjs.com/package/@types/lodash) 之后，就不会报错：`npm install --save-dev @types/lodash`。

其他流行的第三方库，也会有对应的库 `@types/xxx`，可以在 Google 搜索 `xxx types` 就能找到。

## 访问全局变量

如果我们确定有一个全局变量 `GLOBAL`，然后直接在 TypeScript 中访问会报错。例如：

```typescript
console.log(GLOBAL);
```

为了消除这个错误，可以先声明这个全局变量：

```typescript
declare var GLOBAL: any;
```
