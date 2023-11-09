# 04 - 内置类型(Built-in types)

Dart语言支持以下类型：

- Numbers(`int`, `double`)
- Strings(`String`)
- Booleans(`bool`)
- Records(`(value1, value2)`)
- Lists(`List`, 也称为数组)
- Sets(`Set`)
- Maps(`Map`)
- Runes(`Runes`, 通常替换为 `characters` API)
- Symbols(`Symbol`)
- `null`(`Null`)

其他一些类型：

`Object`：除 `Null` 之外的所有 Dart 类的父类。
`Enum`：所有枚举的父类。
`Future` 和 `Stream`：用于异步支持。
`Iterable`：用于 `for-in` 循环和同步生成器功能。
`Never`：表示表达式永远无法成功完成求值。最常用于总是抛出异常的函数。
`dynamic`：表示要禁用静态检查。通常应该使用 `Object` 或者 `Object?`。
`void`：表示从不使用的值。经常用作返回类型。

## Numbers

**int**

不大于64位的整数值，具体取决于平台。在本机平台上，值可以在 `-2^63` 到 `2^63 - 1` 之间。在 web 上，整数值表示为 JavaScript 数字（没有小数部分的64位浮点值），可以是`-2^53` 到 `2^53 - 1` 之间。

**double**

64位（双精度）浮点数。

`int` 和 `double` 都是 `num` 的子类型。`num `类型包括`+`、`-`、`/` 和 `*` 等基本运算符，也有 `abs()`、`ceil()` 和 `floor()` 等方法。（位运算符，如 `>>`，在 `int` 类中定义。）

整数是没有小数点的数字。以下是定义整数文字的一些示例：

```dart
var x = 1;
var hex = 0xDEADBEEF;
```

如果一个数字包含一个小数，它就是一个 `double`。以下是一些示例：

```dart
var y = 1.1;
var exponents = 1.42e5;
```

也可以将变量声明为 `num`。如果这样做，变量可以同时具有 `int` 值和 `double` 值。

```dart
num x = 1;
x += 2.5;
```

必要时，整型文字会自动转换为 `double`：

```dart
double z = 1; // 相当于 double z = 1.0;
```

以下是如何将字符串转换为数字，反之亦然：

```dart
// String -> int
var one = int.parse('1');

// String -> double
var onePointOne = double.parse('1.1');
assert(onePointOne == 1.1);

// int -> String
String oneAsString = 1.toString();
assert(oneAsString == '1');

// double -> String
String piAsString = 3.14159.toStringAsFixed(2);
asset(piAsString == '3.14');
```

`int` 类型指定传统的逐位移位（`<<`，`>>`，`>>>`）、补码（`~`）、AND（`&`）、OR（`|`）和XOR（`^`）运算符，这些运算符对于操作和屏蔽位字段中的标志非常有用。例如：

```dart
assert((3 << 1) == 6); // 0011 << 1 == 0110
assert((3 | 4) == 7);  // 0011 | 0100 == 0111
assert((3 & 4) == 0);  // 0011 & 0100 == 0000
```

文字数字是编译时常数。许多算术表达式也是编译时常数，只要它们的操作数是计算为数字的编译时常数即可。

```dart
const msPerSecond = 1000;
const secondsUntilRetry = 5;
const msUntilRetry = secondsUntilRetry * msPerSecond;
```

## String

`String` 包含一系列 UTF-16 编码单元。可以使用单引号或双引号创建字符串：

```dart
var s1 = 'Single quotes work well for string literals.';
var s2 = "Double quotes work just as well.";
var s3 = 'It\'s easy to escape the string delimiter.';
var s4 = "It's even easier to use the other delimiter.";
```

可以使用 `${expression}` 将表达式的值放入字符串中。如果表达式是标识符，则可以省略 `{}`。为了获得与对象对应的字符串，调用对象的 `toString()` 方法。

```dart
var s = 'string interpolation';

assert('Dart has $s, which is very handy.' ==
    'Dart has string interpolation, '
        'which is very handy.');
assert('That deserves all caps. '
        '${s.toUpperCase()} is very handy!' ==
    'That deserves all caps. '
        'STRING INTERPOLATION is very handy!');
```

> **注意**：`==` 运算符测试两个对象是否相等。如果两个字符串包含相同的编码单元序列，则它们是等效的。

可以使用相邻的字符串文字或 `+` 运算符连接字符串：

```dart
var s1 = 'String '
    'concatenation'
    " works even over line breaks.";
assert(s1 == 'String concatenation works even over line breaks.');

var s2 = 'The + operator ' + 'works, as well.';
assert(s2 == 'The + operator works, as well.');
```

要创建多行字符串，请使用带有单引号或双引号的三引号：

```dart
var s1 = '''
You can create
multi-line strings link this one.
''';

var s2 = """This is also a 
multi-line string.""";
```

可以通过在“原始”字符串前面加 `r` 来创建它：

```dart
var s = r'In a raw string, not event \n gets special treatment.';
```

只要任何插值表达式是计算结果为 null 或数值、字符串或布尔值的编译时常量，则文字字符串就是编译时常量。

```dart
// 这些可以放在常量字符串
const aConstNum = 0;
const aConstBool = true;
const aConstString = 'a constant string';

// 这些不可以放在常量字符串
const aNum = 0;
const aBool = true;
const aString = 'a string';
const aConstList = [1, 2, 3];

const validConstString = '$aConstNum $aConstBool $aConstString';
// const invalidConstString = '$aNum $aBool $aString $aConstList';
```

## Booleans

为了表示布尔值，Dart 有一个名为 `bool` 的类型。只有两个对象具有布尔类型：`true` 和 `false`，它们都是编译时常量。

Dart 的类型安全性意味着您不能使用 `if(nonbooleanValue)` 或 `assert(nonbooleanValue)` 之类的代码。相反，显式检查值，如下所示：

```dart
// 检查字符串是否为空
var fullName = '';
assert(fullName.isEmpty);

// 检查 0
var hitPoints = 0;
assert(hitPoints <= 0);

// 检查 null
var unicorn = null;
assert(unicorn == null);

// 检查 NaN
var iMeantToDoThis = 0 / 0;
assert(iMeantToDoThis.isNaN);
```

## 符文和字形簇 (Runes and grapheme clusters)

在 Dart 中，符文公开了字符串的 Unicode 代码点。您可以使用 [characters package](https://pub.dev/packages/characters) 来查看或操作用户感知的字符。

Unicode 为世界上所有书写系统中使用的每个字母、数字和符号定义了一个唯一的数值。因为 Dart 字符串是 UTF-16 编码单元的序列，所以在字符串中表达 Unicode 代码点需要特殊的语法。表示 Unicode 代码点的常用方法是 `\uXXXX`，其中 `XXXX` 是一个4位数的十六进制值。例如，♥ 是 `\u2665`。要指定多于或少于4个十六进制数字，请将该值放在大括号中。例如，大笑的表情符号(😆) 是 `\u{1f606}`。

如果需要读取或写入单个 Unicode 字符，请使用 characters package 在 String 上定义的 `characters` getter。返回的`Characters` 对象是字符串作为字形簇序列。下面是使用 characters API 的示例：

```dart
import 'package:characters/characters.dart';

void main() {
  var hi = 'Hi 🇩🇰';
  print(hi);
  print('The end of the string: ${hi.substring(hi.length - 1)}');
  print('The last character: ${hi.character.last}');
}
```

输出如下：

```
$ dart run bin/main.dart
Hi 🇩🇰
The end of the string: ???
The last character: 🇩🇰
```

## 符号（Symbols）

`Symbol` 对象表示 Dart 程序中声明的运算符或标识符。可能永远不需要使用 Symbol，但它们对于按名称引用标识符的 API 来说是非常宝贵的，因为缩小（minification）会更改标识符名称，但不会更改标识符符号。

要获取标识符的符号，使用 `#` 后面跟着标识符：

```dart
#radix
#bar
```

符号文字是编译时常量。
