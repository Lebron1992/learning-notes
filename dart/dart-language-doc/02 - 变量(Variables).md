# 02 - 变量(Variables)

下面是一个创建变量并对其进行初始化的示例：

```dart
var name = 'Bob';
```

变量存储的是引用。名为 `name` 的变量包含对值为 `“Bob”` 的 `String` 对象的引用。

`name` 变量的类型被推断为 `String`，但可以通过指定它来更改该类型。如果对象不限于单个类型，请指定 `Object` 类型（如有必要，可指定动态类型）。

```dart
Object name = 'Bob';
```

另一种选择是显式声明将被推断的类型：

```dart
String name = 'Bob';
```

> 注意：通常建议对局部变量使用 `var`，而不是类型注释。

##  Null 安全（Null safety）

Dart 语言强制执行 sound null safety。

Null safety 可防止由于意外访问设置为 `null` 的变量而导致的错误。当访问属性或对计算结果为 `null` 的表达式调用方法时，会发生错误。此规则的一个例外是 null 支持属性或方法，如 `toString()` 或者 `hashCode`。使用 null safety。，Dart 编译器可以在编译时检测到这些潜在的错误。

Null safety 引入了三个关键变化：

1. 为变量、参数或其他相关组件指定类型时，可以控制该类型是否允许 `null`。若启用，请添加 `?` 到类型声明的末尾。

```dart
// 可以为 null 的类型
String? name

// 不能为 null 的类型
String name
```

2. 必须先初始化变量，然后才能使用它们。可为 `null` 的变量默认为 `null`，因此它们在默认情况下进行初始化。Dart不会为非 null 的类型设置初始值。它强制您设置一个初始值。Dart 不允许您观察未初始化的变量。

3. 不能访问具有可为 `null` 类型的表达式的属性或调用方法。同样的异常也适用于 `null` 支持的属性或方法，如 `hashCode` 或 `toString()`。

Sound null safety 将潜在的运行时错误更改为编辑时错误。Null safety 标志非 null 变量，如果它已经：

- 未使用非 null 值进行初始化。
- 分配了 `null` 空值。

## 默认值（Default value）

具有可为 `null` 类型的未初始化变量的初始值为 `null`。即使是具有数字类型的变量最初也是 `null`，因为数字和 Dart 中的其他所有变量一样，都是 objects。

```dart
int? lineCount;
assert(lineCount == null);
```

> 生产代码忽略 `assert()` 调用。在开发过程中，如果条件为false，`assert(condition)` 将抛出异常。

使用 null safety 时，必须先初始化非 null 的变量的值，然后才能使用它们：

```dart
int lineCount = 0;
```

不必在声明局部变量的地方初始化它，但在使用它之前，必须为它赋值。例如，以下代码是有效的，因为 Dart 可以在传递给 `print()` 时检测到 `lineCount` 为非 null：

```dart
int lineCount;

if (weLikeToCount) {
  lineCount = countLines();
} else {
  lineCount = 0;
}

print(lineCount);
```

顶层变量和类变量被延迟初始化；初始化代码在第一次使用变量时运行。

## 延迟变量（Late variables）

`late` 修饰符有两个用例：

- 声明一个在声明后初始化的非 null 的变量。
- 懒散地初始化变量。

Dart 的控制流分析通常可以检测非 null 的变量在使用前何时设置为非 null 值，但有时分析会失败。两种常见的情况是顶层变量和实例变量：Dart通常无法确定它们是否已赋值。

如果确信某个变量在使用前已设置，但 Dart 编译不通过，则可以通过将该变量标记为 `late` 来修复错误：

```dart
late String description;

void main() {
  description = 'Feijoada!';
  print(description);
}
```

> 如果未能初始化 `late` 变量，则在使用该变量时会发生运行时错误。

当将变量标记为 `late`，但在其声明时对其进行初始化时，初始化器将在第一次使用该变量时运行。这种惰性初始化在以下几种情况下很方便：

- 可能不需要该变量，并且初始化该变量的成本很高。
- 正在初始化一个实例变量，其构造函数需要访问 `this`。

在以下示例中，如果从未使用过 `temperature` 变量，则永远不会调用昂贵的 `readThermometer()` 函数：

```dart
late String temperature = readThermometer();
```

## final 和 const（Final and const）

如果从未打算更改变量，使用 `final` 或 `const` 来代替 `var` 或添加到类型中。`final` 变量只能设置一次；`const` 变量是一个编译时常量。（`const` 变量是隐式的的 `final` 变量。）

> 实例变量可以是 `final`，但不能是 `const`。

以下是创建和设置 `final` 变量的示例：

```dart
final name = 'Bob';
final String nickname = 'Bobby';
```

不能更改 `final` 的值：

```dart
// 错误: `final` 变量只能设置一次
name = 'Alice'; 
```

对于要成为**编译时常量**的变量，请使用 `const`。如果 `const` 变量处于类级别，将其标记为 `static const`。在声明变量时，将值设置为编译时常量，如数字或字符串文字、常量变量或对常量进行算术运算的结果：

```dart
const bar = 1000000;
const double atm = 1.01325 * bar;
```

`const` 关键字不仅仅用于声明常量变量。还可以使用它来创建常数值，以及声明创建常数值的构造函数。任何变量都可以有一个常数值。

```dart
var foo = const [];
final bar = const [];
const baz = []; // 等同于 `const []`
```

可以从 `const` 声明的初始化表达式中省略 `const`，就像上面的 `baz` 一样。

可以更改非 final、非 const 变量的值，即使它曾经具有常量值：

```dart
foo = [1, 2, 3]; // 之前的值是 `const []`
```

不能更改常量变量的值：

```dart
baz = [42]; // baz 是一个常量变量，不能重新赋值
```

可以定义使用类型检查和强制转换（`is` 和 `as`）、collection if 和spread运算符（`...`和`...?`）的常量：

```dart
const Object i = 3; // 一个值为 int 的 Object 常量变量
const list = [i as int]; // 类型转换
const map = { if (i is int) i: 'int' }; // is 和 collection if
const set = { if (list is List<int>) ...list }; // spread 运算符
```

> 虽然 `final` 对象无法修改，但其字段可以更改。相比之下，`const` 对象及其字段不能更改：它们是不可变的。
