# 基本运算符 (Basic Operators)

## 术语 (Terminology)

运算符有一元、二元和三元：

- 一元运算符只操作一个操作数(例如`-a`)。一元运算符写在操作数的前面或者后面，例如`!b`和`c!`，并且运算符和操作数不能有空格。
- 二元运算符操作两个操作数(例如`2 + 3`)。
- 三元运算符操作三个操作数。像C语言一样，Swift只有一个三元运算符，`a ? b : c`。

## 赋值运算符 (Assignment Operator)

赋值运算符用来初始化变量的值，或者更新变量的值

```swift
let b = 10
var a = 5
a = b
// a现在等于10
```

如果赋值运算符右边是一个有多个元素的多元组，多元组的元素可以一次性分解成多个常量或者变量：

```swift
let (x, y) = (1, 2)
// x等于1，y等于2
```

不同于C和OC，Swift中的赋值运算符本身不返回任何值。下面这个语句是非法的：

```swift
if x = y {
	// 这是非法的，因为x=y不返回任何值
}
```

## 算术运算符 (Arithmetic Operators)

Swift支持下面这四个标准的算术运算符：

- 加 (`+`)
- 减 (`-`)
- 乘 (`*`)
- 除 (`/`)

```swift
1 + 2       // 等于 3
5 - 3       // 等于 2
2 * 3       // 等于 6
10.0 / 2.5  // 等于 4.0

```

另外，`+`可以用来拼接`String`字符串：

```swift
"hello, " + "world"  // 等于 "hello, world"
```

### 求余运算符 (Remainder Operator)

```swift
9 % 4    // 等于 1
```

为了得出`a % b`的结果，`%`这个求余运算符计算下面的等式，并返回`remainder`：

```swift
a = (b * some multiplier(另外一个乘数)) + 余数
```

`some multiplier`是`a`和`b`的最大公约数。

上面的计算方法同样适用于负数：

```swift
-9 % 4 // 等于 -1
```

把`-9`和`4`代入那个公式：

`-9` = (`4` * `-2`) + `-1`

所以返回`-1`。

其实`b`前面的符号可以忽略，也就意味着`a % b`和`a % -b`的返回值是相等的。

### 一元减号运算符 (Unary Minus Operator)

```swift
let three = 3
let minusThree = -3 		// minusThree等于-3
let plusThree = -minusThree // plusThree等于3
```

### 一元加号运算符 (Unary Plus Operator)

```swift
let minusSix = -6
let alsoMinusSix = +minusSix // alsoMinusSix等于6
```

一元加号运算符不会改变操作数的值。但是为了保持对称，在代码中负数使用一元加号运算符，正数也加上一元加号运算符。

## 复合赋值运算符 (Compound Assignment Operators)

```swift
var a = 1
a += 2
// a 等于 3
```

`a += 2`其实是`a = a + 2`的简写。

**注意：** 复合赋值运算符不会返回任何值，例如：我们不能这样写：`let b = a += 2`。

## 比较运算符 (Comparison Operators)

Swift支持全部C语言的标准运算符：

- 等于 (`a == b`)
- 不等于 (`a != b`)
- 大于 (`a > b`)
- 小于 (`a < b`)
- 大于或等于 (`a >= b`)
- 小于或等于 (`a <= b`)

**注意：** Swift还额外提供了恒等运算符 (*identity operator*) (`===`和`!==`)，用来判断两个对象是否引用同一个实例。

所有的比较运算符都会返回一个`Bool`值：

```swift
1 == 1   // true 因为 1 等于 1
2 != 1   // true 因为 2 不等于 1
2 > 1    // true 因为 2 大于 1
1 < 2    // true 因为 1 小于 2
1 >= 1   // true 因为 1 大于或等于 1
2 <= 1   // false 因为 2 不是小于或等于 1
```

比较运算符通常用于条件语句，例如`if`语句：

```swift
let name = "world"
if name == "world" {
    print("hello, world")
} else {
    print("I'm sorry \(name), but I don't recognize you")
}
// Prints "hello, world", because name is indeed equal to "world".
```

还可以用来比较多元组，只要多元组的元素个数相同，并且每个元素都是可比较的。例如，`Int`和`String`都是可以比较的，也就意味着多元组`(Int, String)`也是可以比较的。相反，`Bool`是不能比较的，所以元素中含有布尔值的多元组是不能比较的。

多元组在比较的时候，是从左到右一个一个比较，直到发现两个参与比较的值不相等，那么这两个不相等的元素的比较结果就作为两个多元组的比较结果。如果每个元素都各自相等，那么这个多元组就相等：

```swift
(1, "zebra") < (2, "apple")   // true 因为 1 is less than 2; "zebra" 和 "apple" 不用在比较
(3, "apple") < (3, "bird")    // true 因为 3 等于 3, 并且 "apple" 小于 "bird"
(4, "dog") == (4, "dog")      // true 因为 4 等于 4, 并且 "dog" 等于 "dog"
```

在上面的例子中，按照从左到右一个个比较的规则，因为`1`小于`2`，所以`(1, "zebra")`小于`(2, "apple") `，因为已经发现了两个不相等的元素，后面的元素将不再比较，尽管`zebra`大于`apple`。然而，当发现两个元素相等时，将会继续往下比较。

**注意：** Swift标准库只支持小于7个元素的多元组进行比较。如果多元组的元素大于或等于7个，我们必须自己实现比较运算符。

## 三元运算符 (Ternary Conditional Operator)

三元运算符是有三部分的比较特别的运算符，格式：`question ? answer1: answer2`。是下面这个代码的简写：

```swift
if question {
    answer1
}
else {
    answer2
}
```

例如下面这个例子：

```swift
let contentHeight = 40
let hasHeader = true
let rowHeight = contentHeight + (hasHeader ? 50 : 20)
// rowHeight 等于 90
```

我们要避免使用三元运算符把多个判断条件组合在一起，因为这会造成代码很难读懂。


## 空合并运算符 (Nil-Coalescing Opeartor)

使用空合并运算符(`a ?? b`)来解包一个可选类型变量，如果a有值，返回`a!`，如果为`nil`，返回默认值`b`。`a`必须是一个可选类型，`b`的类型必须`a`包含的值类型相同。

空合并运算符其实是下面这个代码的简写：

```swift
a != nil ? a! : b
```

下面举个例子：

```swift
let defaultColor = "red"
var userDefinedColorName: String? // 默认是nil

var colorNameToUse = userDefinedColorName ?? defaultColor
// userDefinedColorName 为 nil，所以 colorNameToUse被设置为默认值 "red"
```

## 范围运算符 (Range Operators)

### 闭合范围运算符 (Closed Range Operator)

闭合范围运算符(`a...b`)定义了一个从`a`到`b`的范围，并且包括`a`和`b`，`a`的值不能大于`b`。

当你想使用一个范围的每一个值时，使用闭合运算符非常有用。例如：

```swift
for index in 1...5 {
	print("\(index) 乘以 5 等于 \(index * 5)")
}
// 1 乘以 5 等于 5
// 2 乘以 5 等于 10
// 3 乘以 5 等于 15
// 4 乘以 5 等于 20
// 5 乘以 5 等于 25
```

### 半闭合运算符 (Half-Open Range Operator)

半闭合运算符(`a..<b`)定义了一个从`a`到`b`的范围，但是不包括`b`，`a`的值不能大于`b`。如果`a`等于`b`，那么这个范围就是空的。

在使用下标以0开始的的列表，例如数组，但是不包含数组的长度，使用半闭合运算符非常合适：

```swift
let names = ["Anna", "Alex", "Brian", "Jack"]
let count = names.count
for i in 0..<count {
    print("Person \(i + 1) 是 \(names[i])")
}
// Person 1 是 Anna
// Person 2 是 called Alex
// Person 3 是 called Brian
// Person 4 是 called Jack
```

## 逻辑运算符 (Logical Operators)

Swift支持这三个标准的逻辑运算符：

- 逻辑非 (`!a`)
- 逻辑与 (`a && b`)
- 逻辑或 (`a || b`)

### 逻辑非运算符 (Logical NOT Operator)

逻辑非让一个布尔值取反，`true`变为`false`，`false`变为`true`。

```swift
let allowedEntry = false
if !allowedEntry {
    print("拒绝访问")
}
// Prints "拒绝访问"
```

### 逻辑与运算符 (Logical AND Operator)

逻辑与运算符(`a && b`)两边的逻辑语句都为`true`时，整个语句才是`true`。如果有其中一个为`false`，整个语句就为`false`。实际上，只要第一个语句是`false`，第二个语句就不会在计算了，因为第二个语句无论是`true`还是`false`都不可能把最终的结果改为`true`。

```swift
let enteredDoorCode = true
let passedRetinaScan = false
if enteredDoorCode && passedRetinaScan {
    print("Welcome!")
} else {
    print("ACCESS DENIED")
}
// Prints "ACCESS DENIED"
```

### 逻辑或运算符 (Logical OR Operator)

逻辑或运算符两边的逻辑语句，只要有一个是`true`，那整个语句就是`true`。如果第一个语句是`true`，第二个就不会再计算。

```swift
let hasDoorKey = false
let knowsOverridePassword = true
if hasDoorKey || knowsOverridePassword {
    print("Welcome!")
} else {
    print("ACCESS DENIED")
}
// Prints "Welcome!"
```

### 组合逻辑运算符 (Combining Logical Operators)

```swift
if enteredDoorCode && passedRetinaScan || hasDoorKey || knowsOverridePassword {
    print("Welcome!")
} else {
    print("ACCESS DENIED")
}
// Prints "Welcome!"
```

在这个例子中，使用多个`&&`和`||`运算符来创建一个更长的复合语句。但是`&&`和`||`运算符还是仅仅操作两个逻辑语句。上面的代码可以这么读：如果我们输入了正确的门密码并且通过了视网膜扫描，或者有门钥匙，或者我们知道紧急覆盖密码，那么就允许进入。

### 明确的括号 (Explicit Parentheses)

在恰当的位置加上括号，增强代码的易读性：

```swift
if (enteredDoorCode && passedRetinaScan) || hasDoorKey || knowsOverridePassword {
    print("Welcome!")
} else {
    print("ACCESS DENIED")
}
// Prints "Welcome!"
```
