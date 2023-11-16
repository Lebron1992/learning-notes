# 06 - 集合(Collections)

## 列表（Lists）

也许在几乎所有编程语言中，最常见的集合是数组或有序的对象组。在 Dart 中，数组是 `List` 对象。

Dart 列表字面量由逗号分隔的表达式或值列表表示，该列表用方括号（`[]`）括起来。下面是一个简单的列表：

```dart
var list = [1, 2, 3];
```

> **注意**：Dart 推断列表的类型为 `List＜int＞`。如果尝试将非整数对象添加到此列表中，分析器或运行时会引发错误。

可以在 Dart 集合字面量的最后一项后面添加逗号。这个尾随逗号不会影响集合，但它可以帮助防止复制粘贴错误。

```dart
var list = [
  'Car',
  'Boat',
  'Plane',
]
```

列表使用从零开始的索引，其中0是第一个值的索引，`list.length - 1` 是最后一个值的索引。可以使用 `.length` 属性获取列表的长度，并使用下标运算符（`[]`）访问列表的值：

```dart
var list = [1, 2, 3];
assert(list.length == 3);
assert(list[1] == 2);

list[1] = 1;
assert(list[1] == 1);
```

要创建一个编译时常量列表，请在列表字面量之前添加 `const`：

```dartgi
var constantList = const [1, 2, 3];
// constantList[1] = 1; // 导致错误
```

## 集合（Sets）

Dart中的集合是一组无序并且唯一的值。在 Dart 中的类型是 `Set`。

下面是一个简单的Dart集合，使用字面量创建：

```dart
var halogens = {'fluorine', 'chlorine', 'bromine', 'iodine', 'astatine'};
```

> **注意**：Dart 推断 `halogens` 的类型为`Set＜String＞`。如果试图向集合中添加错误类型的值，分析器或运行时会引发错误。

若要创建空集，使用前面带有类型参数的 `{}`，或将 `{}` 赋给 `Set` 类型的变量：

```dart
var names = <String>{};

// Set<String> names = {}; // 这样也可以
// var names = {}; // 创建 Map，而不是 Set
```

使用 `add()` 或 `addAll()` 方法将项添加到集合：

```dart
var elements = <String>{};
elements.add('fluorine');
elements.allAll(halogens);
```

使用 `.length` 获取集合中的项数：

```dart
var elements = <String>{};
elements.add('fluorine');
elements.allAll(halogens);
assert(elements.length == 5);
```

若要创建一个编译时常量集合，请在该集字面量前添加 `const`：

```dart
final constantSet = const {
  'fluorine',
  'chlorine',
  'bromine',
  'iodine',
  'astatine',
}
// constantSet.add('helium'); // 报错，常量不能继续添加。
```

## 字典（Maps）

通常，map 是一个关联键和值的对象。键和值都可以是任何类型的对象。每个键只能出现一次，但可以多次使用相同的值。

以下是使用字面量创建的几个简单的 map：

```dart
var gifts = {
  // Key: Value
  'first': 'partridge',
  'second': 'turtledoves',
  'fifth': 'golden rings'
}

var nobleGases = {
  2: 'helium',
  10: 'neon',
  18: 'argon',
}
```

> **注意**：Dart 推断 `gifts` 的类型为 `Map<String, String>`，而 `nobleGases` 的类型为 `Map<int, String`。如果试图将错误类型的值添加到 map 中，分析器或运行时会引发错误。

可以使用 `Map` 构造函数创建相同的对象：

```dart
var gifts = Map<String, String>();
gifts['first'] = 'partridge';
gifts['second'] = 'turtledoves';
gifts['fifth'] = 'golden rings';

var nobleGases = Map<int, String>();
nobleGases[2] = 'helium';
nobleGases[10] = 'neon';
nobleGases[18] = 'argon';
```

> **注意**：如果你来自 `C#` 或 `Java` 这样的语言，可能会认为应该是 `new Map()`，而不仅仅是 `Map()`。在 Dart 中，`new` 关键字是可选的。

使用下标赋值运算符（`[]=`）将新的键值对添加到现有 map 中：

```dart
var gifts = {'first': 'partridge'};
gifts['fourth'] = 'calling birds'; // 添加键值对
```

使用下标运算符（`[]`）从 map 中检索值：

```dart
var gifts = {'first': 'partridge'};
assert(gifts['first'] == 'partridge');
```

如果你寻找一个不在 map 中的键，你会得到 `null`：

```dart
var gifts = {'first': 'partridge'};
assert(gifts['fifth'] == null);
```

使用 `.length` 获取 map 中键值对的数量：

```dart
var gifts = {'first': 'partridge'};
gifts['fourth'] = 'calling birds';
assert(gifts.length == 2);
```

要创建一个编译时常量的 map，请在 map 字面量之前添加 `const`：

```dart
final constantMap = const {
  2: 'helium',
  10: 'neon',
  18: 'argon',
}

// constantMap[2] = 'Helium'; // 报错
```

## 运算符（Operators）

### 扩展运算符（Spread operators）

Dart 的 list、map 和 set 字面量中支持的扩展运算符（`...`）和 null 感知扩展运算符（`...?`）。扩展运算符提供了一种将多个值插入集合的简洁方法。

例如，可以使用扩展运算符将一个列表的所有值插入另一个列表：

```dart
var list = [1, 2, 3];
var list2 = [0, ...list];
assert(list2.length == 4);
```

如果扩展运算符右侧的表达式可能为 `null`，则可以使用 null 感知扩展运算符来避免异常：

```dart
var list2 = [0, ...?list];
assert(list2.length == 1);
```

## 控制流运算符（Control-flow operators）

Dart 提供了 collection if 和 collection for，用于list、map 和 set 字面量。可以使用这些运算符来使用条件（`if`）和重复（`for`）来构建集合。

下面是一个使用 collection if 创建包含三个或四个项目的列表的示例：

```dart
var nav = ['Home', 'Furniture', 'Plants', if (promoActive) 'Outlet'];
```

Dart在集合字面量还支持 if case：

```dart
var nav = ['Home', 'Furniture', 'Plants', if (login case 'Manager') 'Inventory'];
```

以下是使用 collection for 的示例：

```dart
var listOfInts = [1, 2, 3];
var listOfStrings = ['#0', for (var i in listOfInt) '#$i'];
assert(listOfString[1] == '#1');
```
