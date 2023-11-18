# 08 - Typedefs

类型别名，通常称为 `typedef`，它是用关键字 `typedef` 声明的，是引用类型的一种简洁方式。下面是一个声明和使用名为 `IntList` 的类型别名的示例：

```dart
typedef IntList = List<int>;
IntList il = [1, 2, 3];
```

类型别名可以具有类型参数：

```dart
typedef ListMapper<X> = Map<X, List<X>>;
Map<String, List<String>> m1 = {}; // 冗长的
ListMapper<String> m2 = {}; // 更简洁
```

在大多数情况下，我们建议使用内联函数类型，而不使用 `typedef`。但是，在函数中使用 typedef 仍然很有用：

```dart
typedef Compare<T> = int Function(T a, T b);

int sort(int a, int b) => a - b

void main() {
  assert(sort is Compare<int>); // True
}
```
