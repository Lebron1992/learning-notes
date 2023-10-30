# 03 - 库和导入(Libraries & imports)

每个 Dart 文件都是一个库，即使它没有使用[library](https://dart.dev/language/libraries#library-directive)指令。

## 使用库（Using libraries）

使用 `import` 指定如何在另一个库的作用域中使用一个库中的命名空间。

例如，Dart web应用程序通常使用[dart:html](https://api.dart.dev/stable/dart-html)库，可以像这样导入该库：

```dart
import 'dart:html';
```

`import` 的唯一参数是库的 URI。对于内置库，URI 具有特殊的 `dart:` 前缀。对于其他库，可以使用文件系统路径或 `package:`。`package:` 指定由包管理器（如pub工具）提供的库。例如：

```dart
import 'pakage:test/test.dart';
```

## 指定库前缀（Specifying a library prefix）

如果导入两个具有冲突标识符的库，则可以为其中一个或两个库指定前缀。例如，如果 library1 和 library2 都有 Element 类，那么可能有如下代码：

```dart
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;

// lib1 的 Element
Element element1 = Element();

// lib2 的 Element
lib2.Element element2 = lib2.Element();
```

## 导入库的一部分（Importing only part of a library）

如果只想使用库的一部分，可以选择性地导入库。例如：

```dart
// 只导入 foo
import 'package:lib1/lib1.dart' show foo;

// 除了 foo，其他都导入
import 'package:lib2/lib2.dart' hide foo;
```

## 延迟加载库

延迟加载允许 web 应用程序在需要库时按需加载库。以下是一些可能使用延迟加载的情况：

- 减少 web 应用程序的初始启动时间。
- 执行 A/B 测试——例如，尝试算法的替代实现。
- 加载很少使用的功能，如可选屏幕和对话框。

> **只有 `dart compile js` 支持延迟加载**

若要延迟加载库，必须首先使用 `deferred as` 导入它。

```dart
import 'package:greetings/hello.dart' deferred as hello;
```

当需要库时，使用库的标识符调用`loadLibrary()`。

```dart
Future<void> greet() async {
  await hello.loadLibrary();
  hello.printGreeting();
}
```

在前面的代码中，`await` 关键字会暂停执行，直到库加载成功为止。

可以多次调用 `loadLibrary()`，但库只加载一次。

使用延迟加载时，请记住以下几点：
- 延迟库的常量不是导入文件中的常量。在加载延迟库之前，这些常量是不存在的。
- 不能在导入文件中使用延迟库中的类型。可以考虑将接口类型移动到由延迟库和导入文件导入的库中。
- Dart 将 `loadLibrary()` 隐式插入到使用 `deferred as namespace` 定义的命名空间中。`loadLibrary()` 函数返回一个 `Future`。

## `library` 指令

要指定库级（library-level）文档注释或元数据注释，将它们附加到文件开头的 `library` 声明中。

```dart
@TestOn('browser')
library;
```























