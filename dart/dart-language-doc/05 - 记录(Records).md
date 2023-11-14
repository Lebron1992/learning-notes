# 05 - 记录(Records)

记录是一种匿名的、不可变的聚合类型。与其他集合类型一样，它们允许将多个对象绑定到一个对象中。与其他集合类型不同，记录是固定大小的、异构的和类型化的。

记录是真实的价值；可以将它们存储在变量中，嵌套它们，将它们传递给函数和从函数传递它们，并将它们存储到数据结构中，如列表(list)、映射(map)和集合(set)。

## 记录的语法（Record syntax）

记录表达式是由逗号分隔的命名字段或位置字段列表，用括号括起来：

```dart
var record = ('first', a: 2, b: true, 'last');
```

记录类型注释是用逗号分隔的类型列表，这些类型包含在括号中。可以使用记录类型注释来定义返回类型和参数类型。例如，以下 `(int, int)` 语句是记录类型注释：

```dart
(int, int) swap((int, int) record) {
  var (a, b) = record;
  return (b, a);
}
```

记录表达式和类型注释中的字段反映了参数和实参在函数中的工作方式。位置字段直接位于括号内：

```dart
// 变量声明中的类型注释
(String, int) record;

// 给记录赋值
record = ('A string', 123);
```

在记录的类型注释中，命名字段位于所有位置字段之后，位于类型和名称对的大括号分隔部分内。在记录表达式中，名称位于每个字段值之前：

```dart
// 变量声明中的类型注释
({int a, bool b}) record;

record = (a: 123, b: true);
```

记录类型中命名字段的名称是记录类型定义或其形状的一部分。具有不同名称的命名字段的两个记录具有不同的类型：

```dart
({int a, int b}) recordAB = (a: 1, b: 2);
({int x, int y}) recordXY = (x: 3, y: 4);

// 编译错误，这两个记录的类型不同
recordAB = recordXY;
```

在记录类型注释中，也可以命名位置字段，但这些名称仅用于文档，不会影响记录的类型：

```dart
(int a, int b) recordAB = (1, 2);
(int x, int y) recordXY = (3, 4);

// OK
recordAB = recordXY;
```

这类似于函数声明的位置参数可以有名称，但这些名称不会影响函数的签名。

## 记录字段（Record fields）

可以通过内置 getter 访问记录字段。记录是不可变的，因此字段没有 setter。

命名字段公开相同名称的 getter。位置字段公开名称为 `$<position>` 的getter，跳过命名字段：

```dart
var record = ('first', a: 2, b: true, 'last');

print(record.$1); // 'first'
print(record.a);  // 2
print(record.b);  // true
print(record.$2)  // 'last'
```

## 记录类型（Record types）

没有针对单个记录类型的类型声明。记录是根据其字段的类型进行结构类型化的。记录的形状（字段集、字段类型及其名称（如果有的话））唯一地决定了记录的类型。

记录中的每个字段都有自己的类型。同一记录中的字段类型可能不同。无论从记录中访问哪个字段，类型系统都知道每个字段的类型：

```dart
(num, Object) pair = (42, 'a');

var first = pair.$1;  // 静态类型 `num`，运行时类型 `int`
var second = pair.$2; // 静态类型 `Object`，运行时类型 `String`
```

## 记录相等性（Record equality）

如果两个记录具有相同的形状（字段集），并且它们对应的字段具有相同的值，则它们是相等的。由于命名字段顺序不是记录形状的一部分，因此命名字段的顺序不会影响相等性。

```dart
(int x, int y, int z) point = (1, 2, 3);
(int r, int g, int b) color = (1, 2, 3);

print(point == color); // true
```

```dart
({int x, int y, int z}) point = (x: 1, y: 2, z: 3);
({int r, int g, int b}) color = (r: 1, g: 2, b: 3);

print(point == color); // false，两个不相关的类型
```

记录根据其字段的结构自动定义 `hashCode` 和 `==` 方法。

## 多个值返回（Multiple returns）

记录允许函数返回捆绑在一起的多个值。要从返回中检索记录值，使用模式匹配将这些值分解为局部变量。

```dart
(String, int) userInfo(Map<String, dynamic> json) {
  return (json['name'] as String, json['age'] as int);
}

final json = <String, dynamic>{
  'name': 'Dash',
  'age': 10,
  'color': 'blue',
}

// 使用记录模式解构：
var (name, age) = userInfo(json);

// 相当于：
var info = userInfo(json);
var name = info.$1;
var age  = info.$2;
```
