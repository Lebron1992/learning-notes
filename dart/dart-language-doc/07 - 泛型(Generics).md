# 07 - 泛型(Generics)

## 为什么使用泛型？（Why use generics?）

泛型通常是类型安全所必需的，但它们的好处不仅仅是允许代码运行：

- 正确地指定泛型类型可以产生更好的代码。
- 可以使用泛型来减少代码重复。

如果您希望列表只包含字符串，则可以将其声明为 `List<String>`。您的程序员同事和您的工具就可以检测到向列表分配非字符串可能是一个错误。以下是一个示例：

```dart
// 静态分析：错误
var names = <String>[];
names.addAll(['Seth', 'Kathy', 'Lars']);
names.add(42); // Error
```

使用泛型的另一个原因是为了减少代码重复。泛型允许在许多类型之间共享单个接口和实现，同时仍然利用静态分析。例如，假设创建了一个用于缓存对象的接口：

```dart
abstract class ObjectdCache {
  Object getByKey(String key);
  void setByKey(String key, Object value);
}
```

您发现您想要此接口的字符串版本，因此您创建了另一个接口：

```dart
abstract class StringCache {
  String getByKey(String key);
  void setByKey(String key, String value);
}
```

然后你决定要这个接口的数字版本...你明白了。

泛型类型可以省去创建所有这些接口的麻烦。您可以创建一个采用类型参数的单个接口：

```dart
abstract class Cache<T> {
  T getByKey(String key);
  void setByKey(String key, T value);
}
```

在这个代码中，T是替身类型。它是一个占位符，您可以将其视为开发人员稍后将定义的类型。

## 使用集合字面量（Using collection literals）

list、set 和 map 字面量可以参数化。参数化字面量与您已经看到的字面量一样，只是在左括号之前添加了 `<type>`（list 和 set）或 `<keyType，valueType>`（map）。以下是使用类型化字面量的示例：

```dart
var names = <String>['Seth', 'Kathy', 'Lars'];
var uniqueNames = <String>{'Seth', 'Kathy', 'Lars'};
var pages = <String, String>{
  'index.html': 'Homepage',
  'robots.txt': 'Hints for web robots',
  'humans.txt': 'We are people, not machines'
}
```

## 将参数化类型与构造函数一起使用（Using parameterized types with constructors）

要在使用构造函数时指定一个或多个类型，请将这些类型放在类名后面的尖括号（`<...>`）中。例如：

```dart
var nameSet = Set<String>.from(names);
```

以下代码创建一个 map，该 map 具有整数键和 View 类型值：

```dart
var views = Map<int, View>(); 
```

## 泛型集合及其包含的类型（Generic collections and the types they contain）

Dart 泛型类型是具体化的，这意味着它们在运行时携带类型信息。例如，您可以测试集合的类型：

```dart
var names = <String>[];
names.addAll(['Seth', 'Kathy', 'Lars']);
print(name is List<String>); // true
```

> **注意**：相比之下，Java 中的泛型使用擦除，这意味着在运行时删除泛型类型参数。在 Java 中，可以测试对象是否是 List，但不能测试它是否是 `List<String>`。

## 限制参数化类型（Restricting the parameterized type）

在实现泛型类型时，您可能希望限制可以作为参数提供的类型，以便参数必须是特定类型的子类型。您可以使用 `extends` 来完成此操作。

一个常见的用例是通过使类型成为 Object 的子类型（而不是默认的 `Object?`）来确保该类型不可为 `null`。

```dart
class Foot<T extends Object> {
  // ...
}
```

除了 `Object` 之外，还可以将 `extends` 与其他类型一起使用。以下是扩展 `SomeBaseClass` 的示例，以便 `SomeBaseClass` 的成员可以在 `T` 类型的对象上调用：

```dart
class Foo<T extends SomeBaseClass> {
  String toString => "Instance of 'Foot<$T>'";
}

class Extender extends SomeBaseClass { 
  // ...
}
```

可以使用 `SomeBaseClass` 或其任何子类型作为泛型参数：

```dart
var someBaseClassFoo = Foo<SomeBaseClass>();
var extender = Foot<Extender>();
```

也可以不指定泛型参数：

```dart
var foo = Foo();
print(foo); // Foo<SomeBaseClass> 的实例
```

指定任何非 SomeBaseClass 类型都会导致错误：

```dart
// 静态分析：错误
var foo = Foo<Object>();
```

## 使用泛型方法（Using generic methods）

方法也可以使用泛型：

```dart
T first<T>(List<T> ts) {
  // 做一些初步工作或错误检查，然后...
  T tmp = ts[0];
  // 做一些额外的检查或处理...
  return tmp;
}
```

`first` (<T>)上的泛型类型参数允许您在多个位置使用类型参数 `T`：

- 在方法的返回值（`T`）。
- 在参数类型（`List<T>`）。
- 在局部变量（`T tmp`）。
