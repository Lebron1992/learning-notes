# 01 - 介绍(Introduction)

## Hello World

每个应用程序都需要一个顶层的 `main()` 函数，从它开始执行。

```dart
void main() {
  print('Hello, World!');
}
```

## 变量（Variables）

即使在类型安全的 Dart 代码中，可以使用 `var` 而无需显式地指定其类型来声明大多数变量。由于类型推断，这些变量的类型由其初始值决定：

```dart
var name = 'Voyager I';
var year = 1977;
var antennaDiameter = 3.7;
var flybyObjects = ['Jupiter', 'Saturn', 'Uranus', 'Neptune'];
var image = {
  'tags': ['saturn'],
  'url': '//path/to/saturn.jpg'
};
```

## 控制流语句（Control flow statements）

Dart 支持常见的控制流语句：

```dart
if (year >= 2001) {
  print('21st century');
} else if (year >= 1901) {
  print('20th century');
}

for (final object in flybyObjects) {
  print(object);
}

for (int month = 1; month <= 12; month++) {
  print(month);
}

while (year < 2016) {
  year += 1;
}
```

## 函数（Functions）

建议指定每个函数的参数和返回值的类型：

```dart
int fibonacci(int n) {
  if (n == 0 || n == 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

var result = fibonacci(20);
```

简写 `=>` 语法对于包含单个语句的函数非常方便。当将匿名函数作为参数传递时，此语法特别有用：

```dart
flybyObjects.where((name) => name.contains('turn')).forEach(print);
```

除了显示一个匿名函数 `where()` 外，这段代码还显示了可以使用一个函数作为参数：`print()` 函数是 `forEach()` 的一个参数。

## 注释（Comments）

Dart 注释通常以 `//` 开头。

```dart
// 单行注释.

/// 这是文档注释，用来注释 libraries, 
/// classes 和他们的成员。

/* 这种注释也支持。*/
```

## 导入（Imports）

要访问在其他库中定义的 API，请使用 `import`。

```dart
// 导入核心库
import 'dart:math';

// 从外部 package 导入库
import 'package:test/test.dart';

// 导入文件
import 'path/to/my_other_file.dart';
```

## 类（Classes）

下面是一个具有三个属性、两个构造函数和一个方法的类的示例。其中一个属性不能直接设置，所以它是使用 `getter` 方法（而不是变量）定义的。

```dart
class Spacecraft {
  String name;
  DateTime? launchDate;

  // 只读属性
  int? get launchYear => launchDate.year;

  // 构造函数，使用语法糖为成员赋值
  Spacecraft(this.name, this.launchDate) {
    // 其他初始化代码
  }

  // 转发到默认构造函数的已命名构造函数。
  Spacecraft.unlaunched(String name) : this(name, null);

  // 方法
  void describe() {
    print('Spacecraft: $name');
    var launchDate = this.launchDate;
    if (launchDate != null) {
      int years = DateTime.now().difference(launchDate).inDays ~/ 365;
      print('Launched: $launchYear ($years years ago)');
    } else {
      print('Unlaunched');
    }
  }
}
```

可以像这样使用 `Spacecraft`：

```dart
var voyager = Spacecraft('Voyager I', DateTime(1977, 9, 5));
voyager.describe();

var voyager3 = Spacecraft.unlaunched('Voyager III');
voyager3.describe();
```

## 枚举（Enums）

枚举是一种枚举预定义的值或实例集的方式，可以确保不存在该类型的任何其他实例。
以下是一个简单枚举的示例，它定义了预定义行星类型的简单列表：

```dart
enum PlanetType { terrestrial, gas, ice }
```

这里是一个描述行星的类的增强枚举声明的例子，它有一组定义的恒定实例，即我们太阳系的行星。

```dart
enum Planet {
  mercury(planetType: PlanetType.terrestrial, moons: 0, hasRings: false),
  venus(planetType: PlanetType.terrestrial, moons: 0, hasRings: false),
  // ···
  uranus(planetType: PlanetType.ice, moons: 27, hasRings: true),
  neptune(planetType: PlanetType.ice, moons: 14, hasRings: true);

  const Planet({required this.planetType, required this.moons, required this.hasRings});

  // 所有实例变量都是 final
  final PlanetType planetType;
  final int moons;
  final bool hasRings;

  // 枚举也支持 getter 和其他方法
  bool get isGiant => planetType == PlanetType.gas || planetType == PlanetType.ice;
}
```

可以这样使用 `Planet`：

```dart
final yourPlanet = Planet.earth;
if (!yourPlanet.isGiant) {
  print('Your planet is not a "giant planet".')
}
```

## 继承（Inheritance）

Dart 支持单继承：

```dart
class Orbiter extends Spacecraft {
  double altitude;

  Orbiter(super.name, DateTime super.launchDate, this.altitude);
}
```

## Mixins

Mixins 是在多个类层次结构中重用代码的一种方式。以下是一个 mixin 声明：

```dart
mixin Piloted {
  int astronauts = 1;

  void describeCrew() {
    print('Number of astronauts: $astronauts');
  }
}
```

要将 mixin 的功能添加到类中，只需使用 mixin 扩展该类即可。

```dart
class PilotedCraft extends Spacecraft with Piloted {
  // ...
}
```

`PilotedCraft` 现在就拥有了 `astronauts` 属性及 `describeCrew()` 函数。

## 接口和抽象类（Interfaces and abstract classes）

所有 classes 都隐式定义了一个 interface。因此，你可以实现任何类。

```dart
class MockSpaceship implements Spacecraft {
  // ...
}
```

可以创建一个抽象类，由具体类进行扩展（或实现）。抽象类可以包含抽象方法（具有空实体）。

```dart
abstract class Describable {
  void describe();

  void describeWithEmphasis() {
    print('=========');
    describe();
    print('=========');
  }
}
```

任何扩展 `Describeable` 的类都有 `describeWithEmphasis()` 方法，该方法调用扩展类对 `describe()` 的实现。

## 异步（Async）

通过使用 `async` 和 `await`，可以避免 callback 嵌套，并使代码可读性更强。

```dart
const oneSecond = Duration(seconds: 1);

Future<void> printWithDelay(String message) async {
  await Future.delayed(oneSecond);
  print(message);
}
```

上面的代码相当于：

```dart
Future<void> printWithDelay(String message) {
  return Future.delayed(oneSecond).then((_) {
    print(message);
  });
}
```

如下一个示例所示，`async` 和 `await` 使异步代码易于阅读。

```dart
Future<void> createDescriptions(Iterable<String> objects) async {
  for (final object in objects) {
    try {
      var file = File('$object.text');
      if (await file.exists()) {
        var modified = await file.lastModified();
        print('File for $object already exists. It was modified on $modified.');
        continue;
      }
      await file.create();
      await file.writeAsString('Start describing $object in this file.');
    } on IOException catch (e) {
      print('Cannot create description for $object: $e');
    }
  }
}
```

还可以使用 `async*`，它提供了一种构建流的良好、可读的方式。

```dart
Stream<String> report(Spacecraft craft, Iterable<String> objects) async* {
  for (final object in objects) {
    await Future.delayed(oneSecond);
    yield '${craft.name} flies by $object';
  }
}
```

## 异常（Exceptions）

使用 `throw` 引发异常。

```dart
if (astronauts == 0) {
  throw StateError('No astronauts.');
}
```

要捕获异常，使用带有 `on` 或 `catch`（或两者都有）的 `try` 语句：

```dart
Future<void> describeFlybyObjects(List<String> flybyObjects) async {
  try {
    for (final object in flybyObjects) {
      var description = await File('$object.txt').readAsString();
      print(description);
    }
  } on IOException catch (e) {
    print('Could not describe object: $e');
  } finally {
    flybyObjects.clear();
  }
}
```

上面的代码是异步的；`try` 同时适用于同步代码和异步函数中的代码。

## 重要概念

- 放置在变量中的所有内容都是一个 object，每个 object 都是 class 的实例。number、function 和 `null` 都是 objects。除了 `null`，所有 object 都继承自 `Object` 类。

- 虽然 Dart 是强类型的，但类型注释是可选的，因为Dart可以推断类型。在 `var number = 101` 中，`number` 被推断为 `int` 类型。

- 如果启用 null safety，变量的值不能是 `null`，除非明确说明它可以 `null`。通过在变量的类型末尾加一个问号（`?`）来使其可以为 `null`。例如，`int?` 类型的变量可能是一个整数，也可能是 `null`。如果你知道一个表达式的计算结果永远不会为 `null`，但 Dart 报错，可以添加 `!` 断言它不是 `null`（如果是 `null` 则抛出异常）。例如：`int x = nullableButNotNullInt!`

- 当想明确表示允许任何类型时，使用类型 `Object?`、`Object`，或者如果您必须将类型检查推迟到运行时——特殊类型 dynamic。

- Dart 支持泛型类型，如 `List<int>`（整数数组）或 `List<Object>`（任何类型的对象数组）。

- Dart 支持顶级函数（如 `main()`），以及绑定到类或对象的函数（分别是静态方法和实例方法）。也可以在函数（嵌套函数或本地函数）中创建函数。

- Dart 支持顶层变量，以及绑定到类或对象的变量（静态变量和实例变量）。实例变量有时被称为字段或属性。

- 与 Java 不同，Dart 没有关键字 `public`、`protected` 和 `private`。如果标识符以下划线（`_`）开头，则它是其库的私有标识符。

- 标识符可以以字母或下划线（`_`）开头，后跟这些字符和数字的任意组合。

- Dart 既有表达式（有运行时值），也有语句（没有值）。例如，条件表达式 `condition ? expr1 : expr2` 的值为 `expr1` 或 `expr2`。没有值的是 `if-else` 语句。语句通常包含一个或多个表达式，但表达式不能直接包含语句。

- Dart 工具可以报告两种问题：警告和错误。警告只是表示您的代码可能无法工作，但它们不会阻止您的程序执行。错误可以是编译时错误，也可以是运行时错误。编译时错误会阻止代码执行；运行时错误导致在代码执行时引发异常。
