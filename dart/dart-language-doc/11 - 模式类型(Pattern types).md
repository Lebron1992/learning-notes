# 11 - 模式类型(Pattern types)

**模式优先级**

与运算符优先级类似，模式求值也遵循优先级规则。可以先使用带括号的模式计算优先级较低的模式。

本文档按优先级升序列出了模式类型：

- logical-or 模式的优先级低于 logical-and，logical-and 模式的优先级高于关系(relational)模式。
- post-fix 一元模式（强制转换、null 检查和 null 断言）具有相同的优先级。
- 其余主要的模式具有最高的优先级。集合类型（record, list 和 map）和 Object 模式包含其他数据，因此首先作为外部模式进行计算。

## 逻辑或（logical-or）

`subpattern1 || subpattern2`

logical-or 模式用 `||` 分隔子模式，如果任何分支匹配，则进行匹配。从左到右对分支求值。一旦一个分支匹配，就不计算其余分支。

```dart
var isPrimary = switch (color) {
  Color.red || Color.yellow || Color.blue => true,
  _ => false
};
```

logical-or 模式中的子模式可以绑定变量，但分支必须定义同一组变量，因为模式匹配时只计算一个分支。

## 逻辑和（logical-and）

`subpattern1 && subpattern2`

由 `&&` 分隔的一对模式只有在两个子模式匹配时才匹配。如果左分支不匹配，则不计算右分支。

logical-and 模式中的子模式可以绑定变量，但每个子模式中的变量不能重叠，因为如果模式匹配，它们都将被绑定：

```dart
switch ((1, 2)) {
  // 错误：两个子模式都尝试绑定 `b`
  case (var a, var b) && (var b, var c): // ...
}
```

## 关系（Relational）

`== expression`

`< expression`

关系模式使用任意等式或关系运算符将匹配的值与给定的常量进行比较：`==`、`!=`、`<`、`>`、`<=` 和 `>=`。

当对具有常量的匹配值调用适当的运算符时，模式匹配，返回 `true`。

关系模式对于数字范围的匹配非常有用，尤其是与逻辑和模式相结合时：

```dart
String assciiCharType(int char) {
  const space = 32;
  const zero = 48;
  const nine = 57;

  return switch (char) {
    < space => 'control',
    == space => 'space',
    > space && < zero => 'punctuation',
    >= zero && <= nine => 'digit',
    _ => ''
  }
}
```

## 强制转换（Cast）

`foo as String`

强制转换模式允许您在解构函数的中间插入类型强制转换，然后再将值传递给另一个子模式：

```dart
(num, Object) record = (1, 's');
var (i as int, s as String) = record;
```

如果值不是指定的类型，则强制转换模式将抛出错误。与 null 断言模式一样，这允许您强制断言某个解构函数值的预期类型。

## Null 检查（Null-check）

`subpattern?`

如果值不为 null，则首先匹配 null 检查模式，然后根据相同的值匹配内部模式。它们允许您绑定一个变量，该变量的类型是所匹配的可为 null 值的不可为 null 的基类型。

要将 null 值视为匹配失败而不抛出，请使用 null 检查模式。

```dart
String? maybeString = 'nullable with base type String';
switch (maybeString) {
  case var s?:
  // `s` 是不可为 null 的 String 类型
}
```

若要在值为 `null` 时进行匹配，请使用null常量模式。

## Null 断言（Null-assert）











































































































































































































































