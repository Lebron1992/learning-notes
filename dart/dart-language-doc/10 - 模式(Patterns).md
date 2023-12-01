# 10 - 模式(Patterns)

模式是 Dart 语言中的一个语法类别，与语句和表达式一样。一个模式表示一组值的形状，这些值可以与实际值相匹配。

## 模式能干什么（What patterns do）

通常来说，模式可以匹配值，也可以解构值，或者两者都有，这取决于模式的上下文和形状。

首先，模式匹配允许您检查给定值是否：

- 有一定的形状。
- 是一个常数。
- 等于其他东西。
- 具有特定类型。

然后，模式解构为您提供了一种方便的声明性语法，可以将该值分解为其组成部分。同样的模式还可以让您将变量绑定到流程中的部分或全部部分。

## 匹配（Matching）

模式总是根据一个值进行测试，以确定该值是否具有您期望的形式。换句话说，您正在检查值是否与模式匹配。

匹配的组成取决于您使用的模式。例如，如果值等于模式的常量，则常量模式匹配：

```dart
switch (number) {
  // 如果 `1 == number` 常量模式匹配
  case 1:
    print('one');
}
```

许多模式都使用子模式，有时分别称为外部模式和内部模式。模式在其子模式上递归匹配。例如，任何集合类型模式的各个字段都可以是变量模式或常量模式：

```dart
const a = 'a';
const b = 'b';
switch (obj) {
  // 如果 `obj` 是一个有两个值的列表，并且这两个值与常量子模式 `a` 和 `b` 匹配，
  // 那么列表模式 [a, b] 匹配 `obj`。
  case [a, b]:
    print('$a, $b');
}
```

要忽略匹配值的部分，可以使用通配符模式作为占位符。在列表模式的情况下，可以使用rest元素。

## 解构（Destructuring）

当对象和模式匹配时，模式可以访问对象的数据，并将其分部分提取。换句话说，模式会解构对象：

```dart
var numList = [1, 2, 3];
// 列表模式 [a, b, c] 从 `numList` 解构三个值，并把值赋给新的变量
var [a, b, c] = numList;
print(a + b + c);
```

您可以在解构模式中嵌套任何类型的模式。例如，此 case 模式匹配并解构第一个元素为 `a` 或 `b` 的双元素列表：

```dart
switch (list) {
  case ['a' | 'b', var c]:
    print(c);
}
```

## 模式可以出现的位置（Places patterns can appear）

在 Dart 语言中，您可以在以下几个位置使用模式：

- 局部变量声明和赋值
- for 和 for-in 循环
- if-case 和 switch-case
- 集合字面量中的控制流

### 变量声明（Variable declaration）

您可以在 Dart 允许本地变量声明的任何地方使用模式变量声明。该模式与声明右侧的值匹配。一旦匹配，它就会解构并将其绑定到新的局部变量：

```dart
// 声明新的变量 a, b 和 c
var (a, [b, c]) = ('str', [1, 2]);
```

### 变量赋值（Variable assignment）

变量赋值模式位于赋值的左侧。首先，它解构匹配的对象。然后，它将值分配给现有变量，而不是绑定新变量。

使用变量赋值模式交换两个变量的值，而不声明第三个临时变量：

```dart
var (a, b) = ('left', 'right');
(b, a) = (a, b); // 交换
print('$a $b'); // right left
```

### 转换语句和表达式（Switch statements and expressions）

每个 case 子句都包含一个模式。这适用于 switch 语句和表达式，以及 if-case 语句。你可以在一个案例中使用任何类型的模式。

case 模式是可以反驳的。它们允许控制流：

- 匹配并解构正在转换的对象。
- 如果对象不匹配，则继续执行。

模式在一个 case 中解构的值变成了局部变量。它们的范围仅在该 case 的主体范围内。

```dart
switch (obj) {
  // `1 == obj` 时匹配
  case 1:
    print('one');

  // 当 obj 在 `first` 和 `last` 之间时匹配
  case >= first && <= last:
  print('in range');

  // 如果 `obj` 是一个刚好有两个值的记录（record），
  // 那么把这两个值分别赋值给 `a` 和 `b`
  case (var a, var b):
    print('a = $a, b = $b');
}
```

逻辑或模式对于在 switch 表达式或语句中让多个 case 共享一个主体非常有用：

```dart
var isPrimary = switch (color) {
  Color.red || Color.yellow || Color.blue => true,
  _ => false
}
```

switch 语句可以让多个 case 共享一个主体，而无需使用逻辑或模式，但它们对于允许多个 case 分享一个保护仍然非常有用：

```dart
switch (shape) {
  case Square(size: var s) || Circle(size: var s) when s > 0:
    print('Non-empty symmetric shape');
}
```

保护(guard)语句将任意条件作为 case 的一部分进行评估，如果条件为false，则不退出 switch（就像在用例主体中使用 `if` 语句一样）。

```dart
switch (pair) {
  case (int a, int b):
  if (a > b) print('First element greater');
  case (int a, int b) when a > b:
    // 如果是 false, 不打印任何东西，而是继续处理下一个 case
    print('First element greater');
  case (int a, int b):
    print('First element not greater');
}
```

### for 和 for-in 循环（For and for-in loops）

可以在循环中使用 for 和 for-in 的模式来迭代和解构集合中的值。

此示例在 for-in 循环中使用对象解构来解构 `<Map>.entries` 返回的 ` MapEntry` 对象：

```dart
Map<String, int> hist = {
  'a': 23,
  'b': 100
}

for (var MapEntry(key: key, value: count) in hist.entries) {
  print('$key occurred $count times');
}
```

对象模式检查 `hist.entries` 是否具有命名类型 `MapEntry`，然后递归到命名字段子模式 `key` 和 `value` 中。它在每次迭代中调用 `MapEntry` 上的`key` getter 和 `value` getter，并将结果分别绑定到局部变量 `key` 和 `count`。

将 getter 调用的结果绑定到同名变量是一种常见的用例，因此对象模式也可以从变量子模式中推断 getter 名称。这允许您将变量模式从像 `key: key` 这样的冗余模式简化为 `:key`:

```dart
for (var MapEntry(:key, value: count) in hist.entries) {
  print('$key occurred $count times');
}
```

## 模式的用例（Use cases for patterns）

### 解构多个返回值（Destructuring multiple returns）

记录允许从单个函数调用中聚合和返回多个值。模式添加了将记录的字段直接解构到局部变量中的功能，与函数调用内联。

而不是为每个记录字段单独声明新的局部变量，如下所示：

```dart
var info = userInfo(json);
var name = info.$1;
var age = info.$2;
```

您可以使用变量声明或赋值模式将函数返回的记录的字段解构为局部变量，并将记录模式作为其子模式：

```dart
var (name, age) = userInfo(json);
```

### 解构类的实例（Destructuring class instances）

对象模式与命名对象类型匹配，允许您使用对象的类已经公开的 getter 来解构它们的数据。

要解构类的实例，请使用命名类型，然后使用括号中要解构的属性：

```dart
final Foo myFoo = Foo(one: 'one', two: 2);
var Foo(:one, :two) = myFoo;
print('one $one, two $two');
```

### 代数数据类型（Algebraic data types）

对象解构和 switch case 有助于以代数数据类型风格编写代码。在以下情况下使用此方法：

- 您有一个相关类型的族。
- 您有一个操作，每个类型都需要特定的行为。
- 您希望将该行为分组在一个位置，而不是将其分布在所有不同的类型定义中。

与其将操作实现为每种类型的实例方法，不如将操作的变体保留在 switch 子类型的单个函数中：

```dart
sealed class Shape {}

class Square implements Shape {
  final double length;
  Square(this.length);
}

class Circle implements Shape {
  final double radius;
  Circle(this.radius);
}

double calculateArea(Shape shape) => switch (shape) {
  Square(length: var l) => l * l,
  Circle(radius: var r) => math.pi * r * r
}
```

### 验证传入的JSON（Validating incoming JSON）

map 和 list 模式可以很好地解构 JSON 数据中的键值对：

```dart
var json = {
  'user': ['Lily', 13]
};

var {'user': [name, age]} = json;
```

如果您知道 JSON 数据具有您期望的结构，那么前面的示例是可以的。但数据通常来自外部来源，比如通过网络。您需要首先对其进行验证以确认其结构。

没有模式，验证是冗长的：

```dart
if (json is Map<String, Object?> &&
    json.length == 1 &&
    json.containsKey('user')) {
  var user = json['user'];
  if (user is List<Object> &&
      user.length == 2 &&
      user[0] is String &&
      user[1] is int) {
    var name = user[0] as String;
    var age = user[1] as int;
    print('User $name is $age years old.');
  }
}
```

一个 case 模式可以实现相同的验证。模式提供了一种更具声明性、更不冗长的验证 JSON 的方法：

```dart
if (json case {'user': [String name, int age]}) {
  print('User $name is $age years old.');
}
```

此 case 模式同时验证：

- `json` 是一个 `map`，因为它必须首先匹配外部 map 模式才能继续。
  - 而且，由于它是一个 map，它还确认 `json` 不是 `null`。
- `json` 包含一个键 `user`。
- 键 `user`与两个值的列表配对。
- 列表值的类型为 `String` 和 `int`。
- 保存这些值的新局部变量是 `String` 和 `int`。
