### 基础知识 (The Basics)

#### 常量和变量 (Constants and Variables)

##### 声明常量和变量 (Declaring Constants and Variables)

使用`let`来声明常量，用`var`来声明变量。例如下面的常量和变量用来跟踪用户的输入的密码次数：

```swift
let maximumNumberOfLoginAttemps = 10
var currentLoginAttemps = 0
```

当然，我们还可以用一行代码中声明多个常量或者变量，并且用逗号隔开：

```swift
var x = 0.0, y = 0.0, z = 0.0
```
**注意：** 如果在代码中，你存储的一个值永远不会变，请一定使用let来声明；后续需要改变的值用var声明。

##### 类型注释 (Type Annotations)

当我们定义一个常量或者变量的时候，可以给一个具体的类型。例如：

```swift
var welcomeMessage: String
welcomeMessage = "Hello"
```

同样，我们可以用一行代码注释多个变量的类型：

```swift
var red, green, blue: Double
```

在大多数情况下，我们不必注释变量的类型，因为Swift能根据常量或者变量的值来推断类型。

##### 命名常量和变量 (Naming Constants and Variables)

常量和变量名几乎可以包含任意字符，包括Unicode：

```swift
let π = 3.14159
let 你好 = "你好世界"
let 🐶🐮 = "dogcow"
```

常量和变量的命名规则和OC一样，不能包括空格、数学符号、箭头、私有（或非法）的Unicode代码、线条和方块图字符；也不能以数字开头，但数字可以出现在后面的其他位置。

##### 打印常量和变量 (Printing Constants and Variables)

直接使用`print(_:separator:terminator:)`方法来打印：

```swift
print(welcomeMessage)
// Prints "Hello"
```

`print(_:separator:terminator:)`方法是一个全局方法，`separator` 和 `terminator`参数有默认值，所以在使用这个方法时可以省略。

当我们要拼接一个变量到字符串，并且打印出来，可以这样写：

```swift
print("\(welcomeMessage), Lebron James!")
// Prints "Hello, Lebron James!"
```

#### 注释 (Comments)

##### 单行注释

```swift
// This is a comment.
```

##### 多行注释

```swift
/* This is also a comment
 but is written over multiple lines. */
```

##### 内嵌多行注释

```swift
/* This is the start of the first multiline comment.
 /* This is the second, nested multiline comment. */
 This is the end of the first multiline comment. */
```

#### 分号 (Semicolons)

不同于其他语言，Swift在每行代码后面不需要加分号。

#### 整数 (Integers)

Swift提供了8、16、32和64位的整数。这些整数的命名和C语言类似，例如8位无符号整数是`UInt8`，32位有符号整数是`Int32`。向其他Swift类型一样，这些整数类型的名称都是大写字母开头的。

##### 整数边界 (Integer Bounds)

我们可以使用`min`和`max`属性来访问各个整数类型的最小值和最大值。

```swift
let minValue = UInt8.min  // UInt8的最小值是0
let maxValue = UInt8.max  // UInt8的最大值是255
```

##### Int

在多数情况下，我们不必在代码中指定整数的取值范围。Swift提供了另外一个整数类型`Int`，它的取值范围取决于当前运行平台：

- 在32-bit的平台上，`Int`相当于`Int32`
- 在64-bit的平台上，`Int`相当于`Int64`

**注意：**当你真的特别需要无符号的整数类型时才使用`Unit`，否则一般情况下是推荐使用`Int`，即使你知道你要存储的值是非负数。一致使用`Int`能增强代码的互动性，可以避免不同数字类型的转换。

##### UInt

Swift也提供了不带符号的整型，`UInt`，他的取值范围也取决于当前运行平台：

- 在32-bit的平台上，`UInt`相当于`UInt32`
- 在64-bit的平台上，`UInt`相当于`UInt64`

#### 浮点型 (Floating-Point Numbers)

浮点型的变量存储的值范围比整数型更广。Swift提供了两种有符号的浮点型类型：

- `Double`：代表64-bit浮点型
- `Float`：代表32-bit浮点型

**注意：**`Double`的精度可以达到15为小数，而Float只能达到6位小数。选用哪一种类型取决于你的实际情况。一般情况下，我们直接使用`Double`。

#### 类型安全与类型推断 (Type Safety and Type Inference)

细心的同学可以发现，在上面定义的一些变量，我们无需在最前面写变量的类型，而是直接用`let`或者`var`。因为Swift可以直接根据你赋给那个变量的值来推断变量的类型。例如：

```swift
let meaningOfLife = 42 // meaningOfLife被推断为Int类型，很明显，我们从42字面上就能看出是一个整数
let pi = 3.14159 // pi被推断为Double类型
```

在推断浮点型数字的时候，Swift会选择推断为Double，而不是Float。

如果一个整数和一个小数相加，以算式的方式复制给一个变量，这个变量会被推断为Double类型：

```swift
let anotherPi = 3 + 0.14159 // anotherPi是Double类型
```

#### 数值类型的字面值 (Numeric Literals)

整数类型的字面值应该这样写：

- 十进制数，没有前缀
- 二进制数，需要加`0b`前缀
- 八进制数，需要加`0o`前缀
- 十六进制数，需要加`0x`前缀

按照上面的规则，十进制数17可以写成：

```swift
let decimalInteger = 17
let binaryInteger = 0b10001       // 17 in binary notation
let octalInteger = 0o21           // 17 in octal notation
let hexadecimalInteger = 0x11     // 17 in hexadecimal notation
```

浮点型的字面值也可以写成10进制(没有前缀)，或者16进制(`0o`前缀)。10进制的浮点型还需要一个可选的指数`e`；16进制的也需要一个指数`p`。

十进制的浮点型有一个指数`e`，这个浮点型的值就等于基数乘以10<sup>e</sup>：

- `1.25e2`就是 1.25 x 10<sup>2</sup>，或者125.0。
- `1.25e-2`就是 1.25 x 10<sup>-2</sup>，或者0.0125。

十六进制的浮点型有一个指数`p`，这个浮点型的值就等于基数乘以2<sup>p</sup>：

- `0xFp2` 就是 15 x 2<sup>2</sup>，或者是60.0。
- `0xFp-2` 就是 15 x 2<sup>-2</sup>，或者是3.75。

下面这些变量的字面值都代表着十进制的`12.1875`：

```swift
let decimalDouble = 12.1875
let exponentDouble = 1.21875e1
let hexadecimalDouble = 0xC.3p0
```

数值类型的字面值可以包含其他格式化的字符使得这个数更易读。整型和浮点型都可以加上额外的`0`和包含`_`来增强可读性。这些额外增加的字符不会改变这个数的值。

```swift
let paddedDouble = 000123.456
let oneMillion = 1_000_000
let justOverOneMillion = 1_000_000.000_000_1
```

#### 数字类型转换 (Numeric Type Conversion)

##### 整型转换 (Integer Conversion)

不同整数类型的取值范围是不同的。例如`Int8`只能存储-128~127的数值；而`UInt8`只能存储0~225。如果一个数超过了这个整数类型的范围，将会报错：

```swift
let cannotBeNegative: UInt8 = -1
// UInt8 cannot store negative numbers, and so this will report an error
let tooBig: Int8 = Int8.max + 1
// Int8 cannot store a number larger than its maximum value,
// and so this will also report an error
```

类型转换的格式方法如下：

```swift
let twoThousand: UInt16 = 2_000
let one: UInt8 = 1
let twoThousandAndOne = twoThousand + UInt16(one)
```

因为`one`是`UInt8`类型，而`twoThousand`是`UInt16`类型，所以他们不能直接相加，需要把`one`转换成`UInt16`类型，然后才可以相加。

##### 整型和浮点类型转换 (Integer and Floating-Point Conversion)

整型和浮点型之间的转换必须显示转换：

```swift
let three = 3
let pointOneFourOneFiveNine: 0.14159
let pi = Double(three) + pointOneFourFiveNine
// pi 等于 3.14159, 并且被推断为Double类型
```

浮点型转为整型也必须是显示的：

```swift
let integerPi = Int(pi)
// integerPi 等于 3， 并且被推断为Int类型
```

当浮点型转为整型时，浮点型的小数部分总是被省略掉，只取整数部分。也就是说4.75会被转为4，-3.9被转为-3。

#### 类型别名 (Type Aliases)

使用`typealias`关键字来定义一个别名。当我们想用一个更清晰更适合的类型来表达一个变量的时候，使用别名是非常有用的。例如我们从外部资源操作一个特定大小的数据时：

```swift
typealias AudioSample = UInt16
```

定义好这个别名之后，你就可以像使用`UInt16`一样地使用`AudioSample`：

```swift
var maxAmplitudeFound = AudioSample.min
// maxAmplitudeFound is now 0
```

因为`AudioSample`是一个别名，所以`AudioSample.min`实际上就是`UInt16.min`。

其实别名可以这么理解，就是为代码的可读性更高，更贴合项目实际，例如我们经常见的`TimeInterval`，实际上是一个`Double`类型。


#### 布尔值 (Booleans)

Swift提供了两个布尔类型的常量：`true`和`false`：

```swift
let orangesAreOrange = true
let turnipsAreDelicious = false
```

`orangesAreOrange`和`turnipsAreDelicious`被推断为`Bool`类型。

Swift是不允许其他不是布尔类型的值替代`Bool`的。例如下面这个例子就会报错：

```swift
let i = 1
if i {
    // 这个例子不能编译通过，会报错
}
```

如果改为下面这个例子就可以编译通过：

```swift
let i = 1
if i == 1 {
    
}
```

`i == 1` 的结果是`Bool`类型，所以第二个例子是正确的。


#### 多元组 (Tuples)

一个多元组包含了多个元素，多元组里面的元素可以是任何类型，并且可以是不一样的类型。例如：

```swift
let http404Error = (404, "Not Found")
```
`http404Error`这个多元组描述了HTTP的一个状态码，里面包含了一个`Int`和`String`，所以这个多元组的类型是`(Int, String)`。

我们可以拆分多元组里面的元素，例如：

```swift
let (statusCode, statusMessage) = http404Error
print("The status code is \(statusCode)")
// Prints "The status code is 404"
print("The status message is \(statusMessage)")
// Prints "The status message is Not Found"
```

如果你只需要访问其中的一个元素，可以使用`_`来忽略其他元素：

```swift
let (justTheStatusCode, _) = http404Error
print("The status code is \(justTheStatusCode)")
// Prints "The status code is 404"
```

另外，我们还可以使用下标来访问里面的元素：

```swift
print("The status code is \(http404Error.0)")
// Prints "The status code is 404"
print("The status message is \(http404Error.1)")
// Prints "The status message is Not Found"
```

给多元组的元素命名：

```swift
let http200Status = (statusCode: 200, description: "OK")
```

一旦里面的元素有了名字，就可以使用名字来访问：

```swift
print("The status code is \(http200Status.statusCode)")
// Prints "The status code is 200"
print("The status message is \(http200Status.description)")
// Prints "The status message is OK"
```

多元组作为方法的返回值是特别有用的，我们在方法里可以返回更多的信息。


#### 可选类型 (Optionals)

可选类型是Swift新增的一个类型，在OC或者C中是没有的。当一个值有可能为空时，就是用可选类型。一个可选类型代表着两种可能：1）有一个特定的值，并且可以把这个可选值解包得到他真正的值；2）没有值，也就是`nil`。

下面举个例子，把`String`转换为`Int`:

```swift
let possibleNumber = "123"
let convertedNumber = Int(possibleNumber)
convertedNumber被推断为Int?类型，或者说是 optional Int
```

因为`Int(possibleNumber)`在转换的时候，假如`possibleNumber`的字面值不是数字，那就会转换不成功，最后返回`nil`给`convertedNumber`。所以`convertedNumber`的值有两种可能，要么就转成功，返回具体值，要么就转换不成功，返回`nil`。所以convertedNumber是`Int?`类型。这个`?`的意思是变量的值可能会是`nil`。

##### nil

我们可以把`nil`赋值给一个可选类型：

```swift
var serverResponseCode: Int? = 404
// serverResponseCode contains an actual Int value of 404
serverResponseCode = nil
// serverResponseCode now contains no value
```

**注意：** `nil`不能赋值给非可选类型的常量和变量。所以如果在代码中，如果变量有可能为`nil`，尽量声明为可选类型。

如果在声明可选类型变量的时候，没有给一个默认值，那么这个变量默认为`nil`：

```swift
var surveyAnswer: Stirng?
// surveyAnswer 自动设置为nil
```

**注意：**Swift的`nil`和OC中的`nil`是不同的。在OC中，`nil`是一个指针，指向一个不存在的对象；而在Swift中，`nil`不是指针，只是表示一个特定类型的值是空的。任何可选类型都可以设置为`nil`，不仅仅是对象类型。

##### If语句和强制解包 (If Statements and Forced Unwrapping)

使用`==`或者`!=`来判断一个可选类型的变量是否等于`nil`：

```swift
if covnertedNumber != nil {
    print("convertedNumber contains some integer value.")
}
```

当我们能确定可选类型变量确实包含了一个值，可以使用`!`来解包这个可选类型变量：

```swift
if convertedNumber != nil {
    print("convertedNumber has an integer value of \(convertedNumber!).")
}
// Prints "convertedNumber has an integer value of 123."
```

**注意：** 在使用`!`解包时，一定要保证可选类型的变量有值，不能为`nil`，否则在运行的时候会报错。

##### 可选绑定 (Optional Binding)

使用可选绑定来确认一个可选类型是否有值，如果有值，那么就赋值给一个局部的变量或者常量。可选绑定可以配合`if`和`while`语句来检查一个可选类型变量是否有值：

```swift
if let constantName = someOptional {
	// do something...
}
```

我们可以使用可选绑定重写之前的这个例子：

```swift
// 之前的例子
if convertedNumber != nil {
    print("convertedNumber has an integer value of \(convertedNumber!).")
}

// 使用可选绑定重写，改为
if let actualNumber = Int(possibleNumber) {
    print("\"\(possibleNumber)\" has an integer value of \(actualNumber)")
}
else {
    print("\"\(possibleNumber)\" could not be converted to an integer")
}
// Prints ""123" has an integer value of 123"
```

代码可以这么解读：如果`Int(possibleNumber)`返回的值是一个非`nil`的值，那么就把这个值赋值给`actualNumber`，然后`actualNumber`就可以作为局部变量在if的第一个分支语句里使用。因为`actualNumber`已经确定有非`nil`值，所以在打印那句代码无需用`!`解包。

在if语句中，我们可以包含多个可选绑定和布尔条件，并且以`,`隔开。只要任意一个可选绑定或者任意一个布尔条件的结果为false，那么整个if语句的值将会被判断为false：

```swift
if let firstNumber = Int("4"), let secondNumber = Int("42"), firstNumber < secondNumber && secondNumber < 100 {
    print("\(firstNumber) < \(secondNumber) < 100")
}
// Prints "4 < 42 < 100"

if let firstNumber = Int("4") {
    if let secondNumber = Int("42") {
        if firstNumber < secondNumber && secondNumber < 100 {
            print("\(firstNumber) < \(secondNumber) < 100")
        }
    }
}
// Prints "4 < 42 < 100"
```

第一种写法更为简短。

##### 隐式解包可选类型 (Implicitly Unwrapped Optionals)

有时候，我们能确定一个可选类型被第一次赋值后，总会有值。那我们就可以把这种可选类型声明为隐式解包可选类型。在要设置为可选的类型名称后面加上`!`，例如`String!`。

其实隐式解包可选类型实际上也是一个正常的可选类型，但是可以像非可选类型一样使用，访问它的值时不用解包。

```swift
let possibleString: String? = "一个可选类型的字符串"
let forcedString: String = possibleString! // 需要感叹号解包

let assumedString: String! = "一个隐式解包可选类型的字符串"
let implicitString: String = assumedString // 无需感叹号解包
```

**注意：** 如果一个隐式解包可选类型是`nil`，然后你访问他的解包后的值，将会在运行的时候报错。

像正常的可选类型一样，你可以使用if检查是否有值：

```swift
if assumedString != nil {
    print(assumedString)
}
// Prints "An implicitly unwrapped optional string."
```

也可以使用可选绑定：

```swift
if let definiteString = assumedString {
    print(definiteString)
}
// Prints "An implicitly unwrapped optional string."
```

**注意：** 当一个变量有可能为`nil`时，千万不要使用隐式解包可选绑定。

#### 错误处理 (Error Handling)

当一个方法遇到错误条件时，会抛出一个错误。方法的调用者就能抓取这个错误，然后进行处理：

```swift
func canThrowAnError() throws {
	// this function may or may not throw an error
}
```

在声明方法时，使用`throws`关键字来说明这个方法会抛出异常。当调用一个会抛出异常的方法时，使用`try`关键字。

```swift
do {
	try canThrowAnError()
	// no error was thrown
}
catch {
	// an error was thrown
}
```

可以使用多个`catch`来抓取不同的异常：

```swift
func makeASandwich() throws {
    // ...
}
 
do {
    try makeASandwich()
    eatASandwich()
} catch SandwichError.outOfCleanDishes {
    washDishes()
} catch SandwichError.missingIngredients(let ingredients) {
    buyGroceries(ingredients)
}
```

在这个例子中，如果没有干净的盘子可用或者缺少某种做三明治的原料时，`makeASandwich()`将会抛出异常。如果没有异常，就会执行`eatASandwich()`。如果抓取的错误匹配`SandwichError.outOfCleanDishes`，就会执行`washDishes()`，如果抓取的错误匹配`SandwichError.missingIngredients`，就会执行` buyGroceries(_:) `。

#### 断言 (Assertions)

##### 调试断言 (Debugging with Assertions)

在某些情况下，我们需要让某些变量满足特定的条件之后，才让代码继续执行。如果不满足，代码会运行错误。我们可以在代码中使用断言来阻止代码执行，然后调试错误的原因。

```swift
let age = -3
assert(age >= 0, "人的年龄不能小于0")
```

如果`age >= 0`是`true`，那么代码继续执行，如果是`false`，那么程序将会停止，并且打印`人的年龄不能小于0`。

**注意：** 在当优化优化编译时，断言会被禁用，例如在Xcode中使用一个程序的默认发布配置建立新版本的时候。

##### 何时使用断言 (When to Use Assertions)

当一个判断条件有可能为false时，可以使用断言。下面这些情况可以使用断言：

- 当一个整数下标被传给一个自定义的下标实现时，但是下标的值可能会太小或太大。
- 当一个值作为参数传给一个方法，但是一个非法的值不能让这个方法完成它的任务。
- 一个可选类型的值目前是`nil`，但是后面的代码要求这个可选类型要有一个非`nil`的值。
