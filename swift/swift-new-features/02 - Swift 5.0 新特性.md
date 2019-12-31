## 声明
> 此篇文章大部分类容翻译自 [Hacking with Swift](https://www.hackingwithswift.com) 的 [What’s new in Swift 5.0](https://www.hackingwithswift.com/articles/126/whats-new-in-swift-5-0) ，需要看原文的可以直接点击链接。另外原文作者还创建了 [playground](https://github.com/twostraws/whats-new-in-swift-5-0)，方便读者学习。翻译这篇文章的主要目的是加深对文章内容的学习；以后看到有比较好的文章，也会通过这种方式来学习。还记得当时是因为美国同事的推荐，知道了 [Hacking with Swift](https://www.hackingwithswift.com) 这个网站。从那以后自己也从这里学到了不少东西。

## 原始字符串
 [SE-0200](https://github.com/apple/swift-evolution/blob/master/proposals/0200-raw-string-escaping.md) 添加了创建原始字符串的功能，其中反斜杠和引号被解释为文字符号，而不是转义字符或字符串终止符。这使得许多用例更加容易，尤其是正则表达式会很受益。

要使用原始字符串，请在字符串两边前放置一个或多个 `#` 符号，如下所示：

```swift
let rain = #"The "rain" in "Spain" falls mainly on the Spaniards."#
```

字符串开始和结束的 `#` 符号成为字符串分隔符的一部分，因此 Swift 就可以理解 rain 和 spain 的两边的独立引号应被视为文本，而不是字符串的结尾。

原始字符串也允许使用反斜杠：

```swift
let keypaths = #"Swift keypaths such as \Person.name hold uninvoked references to properties."#
```

它将反斜杠视为字符串中的文字字符，而不是转义字符。这就意味着字符串插值的工作方式不同：

```swift
let answer = 42
let dontpanic = #"The answer to life, the universe, and everything is \#(answer)."#
```

这里我们使用 `\#(answer)`来实现字符串插值，而以前我们所熟知的`\(answer)`将被解释为字符串中的字符，因此当您希望在原始字符串中使用字符串插值时，必须添加额外的`#`字符。

Swift 原始字符串的一个有趣的特性是在开头和结尾使用 `#` 符号，您可以使用多个`#`符号。如下所示：

```swift
let str = ##"My dog said "woof"#gooddog"##
```

请注意，末尾的`#`个数必须与开头的`#`个数匹配。

原始字符串与 Swift 的多行字符串系统完全兼容–只需在开始使用 `#"""`，然后结尾处使用`"""#`，如下所示：

```swift
let multiline = #"""
	The answer to life,
	the universe,
	and everything is \#(answer).
	"""#
```

在正则表达式中，尽量不使用大量反斜杠是非常有好处的。例如，编写一个简单的 regex 来查找键路径，如`\Person.name`，用于如下所示：

```swift
let regex1 = "\\\\[A-Z]+[A-Za-z]+\\.[a-z]+"
```

由于有了原始字符串，我们可以用一半的反斜杠来写同样的东西：

```swift
let regex2 = #"\\[A-Z]+[A-Za-z]+\.[a-z]+"#
```

## `Result` 类型

[SE-0235](https://github.com/apple/swift-evolution/blob/master/proposals/0235-add-result.md)在标准库中引入了一个 `Result` 类型，使我们能够更简单、更清晰地处理复杂代码（如异步API）中的错误。

Swift的 `Result` 类型是一个枚举，有两种情况：`success`和`failure`。两者都是使用泛型实现的，这样它们就可以有关联值，但是`failure`必须是遵循`Error`类型的。

为了演示`Result`，我们可以编写一个连接到服务器的函数来计算有多少未读消息在等待用户。在这个示例代码中，我们只有一个可能的错误，即请求的URL字符串不是有效的URL:

```swift
enum NetworkError: Error {
    case badURL
}
```

连接到服务器的函数将接受一个URL字符串作为其第一个参数，并接受一个 completion handler 作为其第二个参数。completion handler 本身将接受一个 `Result`，其中 success case 将存储一个整数，而 failure case 将是某种网络错误。我们实际上不会在这里连接到服务器，但是使用 completion handler 可以让我们模拟异步代码。代码如下：

```swift
import Foundation

func fetchUnreadCount1(from urlString: String, completionHandler: @escaping (Result<Int, NetworkError>) -> Void)  {
    guard let url = URL(string: urlString) else {
        completionHandler(.failure(.badURL))
        return
    }

    // complicated networking code here
    print("Fetching \(url.absoluteString)...")
    completionHandler(.success(5))
}
```

要使用上面的代码，我们需要检查 `Result` 中的值，以查看调用是否成功，如下所示：

```swift
fetchUnreadCount1(from: "https://www.hackingwithswift.com") { result in
    switch result {
    case .success(let count):
        print("\(count) unread messages.")
    case .failure(let error):
        print(error.localizedDescription)
    }
}
```

在开始在自己的代码中使用 `Result` 之前，还有三件事情您应该知道:

- 第一： `Result` 有一个`get()`方法，该方法返回成功的值（如果存在），或者抛出错误。这允许您将 `Result` 转换为常规的抛出调用，如下所示：

```swift
fetchUnreadCount1(from: "https://www.hackingwithswift.com") { result in
    if let count = try? result.get() {
        print("\(count) unread messages.")
    }
}
```

- 第二： `Result` 有一个初始化器，它接受一个抛出闭包：如果成功，则闭包返回一个用于 success case的值，否则抛出用于 failure case 的错误。例如：

```swift
let result = Result { try String(contentsOfFile: someFile) }
```

- 第三：与其使用您创建的特定错误枚举，倒不如使用常规 `Error`协议。事实上，Swift Evolution 建议说，“在使用 `Result`时，大多数情况都将使用 `Swift.Error` 作为 `Error`类型参数。” 因此，不使用 `Result<Int, NetworkError>` 而是使用 `Result<Int, Error>`。虽然这意味着您将失去类型化抛出的安全性，但您将获得抛出各种不同错误枚举的能力——您更喜欢哪种错误枚举取决于您的编码风格。

## 自定义字符串插值

 [SE-0228](https://github.com/apple/swift-evolution/blob/master/proposals/0228-fix-expressiblebystringinterpolation.md) 大幅改进了 Swift 的字符串插值系统，使其更高效、更灵活，并创造了一系列以前不可能实现的新功能。

在其最基本的形式中，新的字符串插值系统允许我们控制对象在字符串中的显示方式。Swift具有有助于调试structs 的默认行为，因为它会打印结构名及其所有属性。但是，如果您使用的是 classes（没有这种行为），或者希望格式化输出，以便它可以对用户更友好，那么您可以使用新的字符串插值系统。

例如，如果我们有一个一下结构体：

```swift
struct User {
    var name: String
    var age: Int
}
```

如果我们想为它添加一个特殊的字符串插值，以便整齐地打印 users，我们将扩展 `String.StringInterpolation`并添加 `appendInterpolation()` 方法。Swift已经内置了其中的几个，并且使用了插值类型。在本例中，`User`需要找出要调用的方法。

在本例中，我们将添加一个将用户的名称和年龄放入单个字符串的实现，然后调用一个内置的`appendInterpolation()`方法将其添加到字符串中，如下所示：

```swift
extension String.StringInterpolation {
    mutating func appendInterpolation(_ value: User) {
        appendInterpolation("My name is \(value.name) and I'm \(value.age)")
    }
}
```

现在我们可以创建一个用户并打印出他们的数据：

```swift
let user = User(name: "Guybrush Threepwood", age: 33)
print("User details: \(user)")
// User details: My name is Guybrush Threepwood and I'm 33
```

而如果使用字符串插值，将会打印 `User details: User(name: "Guybrush Threepwood", age: 33)`。当然，这个功能与实现 `CustomStringConvertible` 协议没有什么不同，所以让我们继续讨论更高级的用法。

您的自定义插值方法可以根据需要使用任意多个参数。例如，我们可以使用各种样式添加一个插值来打印数字，如下所示：

```swift
extension String.StringInterpolation {
    mutating func appendInterpolation(_ number: Int, style: NumberFormatter.Style) {
        let formatter = NumberFormatter()
        formatter.numberStyle = style

        if let result = formatter.string(from: number as NSNumber) {
            appendLiteral(result)
        }
    }
}
```

`NumberFormatter` 类有许多样式，包括货币（$72.83）、序数（1st, 12th）和拼写（five, forty-three）。所以，我们可以创建一个随机数，并把它拼成这样的字符串：

```swift
let number = Int.random(in: 0...100)
let lucky = "The lucky number this week is \(number, style: .spellOut)."
print(lucky)
```

您可以根据需要多次调用`appendLiteral()`，甚至在必要时根本不调用。例如，我们可以添加一个字符串插值来多次重复一个字符串，如下所示：

```swift
extension String.StringInterpolation {
    mutating func appendInterpolation(repeat str: String, _ count: Int) {
        for _ in 0 ..< count {
            appendLiteral(str)
        }
    }
}

print("Baby shark \(repeat: "doo ", 6)")
```

而且，由于这些只是常规方法，所以您可以使用Swift的全部功能。例如，我们可以添加一个将字符串数组连接在一起的插值，但如果该数组为空，则执行一个返回字符串的闭包：

```swift
extension String.StringInterpolation {
    mutating func appendInterpolation(_ values: [String], empty defaultValue: @autoclosure () -> String) {
        if values.count == 0 {
            appendLiteral(defaultValue())
        } else {
            appendLiteral(values.joined(separator: ", "))
        }
    }
}

let names = ["Harry", "Ron", "Hermione"]
print("List of students: \(names, empty: "No one").")
```

使用 `@autoclosure` 意味着我们可以使用简单的值或调用复杂的函数作为 `defaultValue`，但是除非`values.count`为零，否则不会用到默认值。

结合了`ExpressibleByStringLiteral`和`ExpressibleByStringInterpolation`协议，现在可以使用字符串插值创建整个类型，如果我们添加`CustomStringConvertible`，甚至可以让这些类型按我们想要的方式打印为字符串。

要实现这一目标，我们需要满足一些特定的标准：

- 我们创建的任何类型都应该遵循`ExpressibleByStringLiteral`、`ExpressibleByStringInterpolation`和`CustomStringConvertible`。只有当您想自定义打印类型的方式时，才需要使用后者。
- 在类型内部需要是一个名为`StringInterpolation`的嵌套 Struct，该Struct遵循`StringInterpolationProtocol`。
- 嵌套Struct需要有一个初始化器，它接受两个整数，大致告诉我们它可以需要多少数据。
- 它还需要实现`appendLiteral()`方法，以及一个或多个`appendInterpolation()`方法。
- 您的主类型需要有两个初始化器，允许从字符串文本和字符串插值创建它。

我们可以将所有这些放到一个示例类型中，该类型可以从各种常见的元素构造HTML。嵌套的`StringInterpolation` Struct中的 “scratchpad” 将是一个字符串：每次添加一个新的文本或插值时，我们都会将其附加到字符串中。为了帮助您理解我们要演示的例子，我在各种append方法中添加了一些`print()`调用:

```swift
struct HTMLComponent: ExpressibleByStringLiteral, ExpressibleByStringInterpolation, CustomStringConvertible {
    struct StringInterpolation: StringInterpolationProtocol {
        // 开始时是一个空字符串
        var output = ""

        // 分配足够的内存来存储两倍的文本
        init(literalCapacity: Int, interpolationCount: Int) {
            output.reserveCapacity(literalCapacity * 2)
        }

        // 一段硬编码的文本
        mutating func appendLiteral(_ literal: String) {
            print("Appending \(literal)")
            output.append(literal)
        }

        // 一个 Twitter 用户名，并且作为一个链接
        mutating func appendInterpolation(twitter: String) {
            print("Appending \(twitter)")
            output.append("<a href=\"https://twitter/\(twitter)\">@\(twitter)</a>")
        }

        // 邮件地址
        mutating func appendInterpolation(email: String) {
            print("Appending \(email)")
            output.append("<a href=\"mailto:\(email)\">\(email)</a>")
        }
    }

    // 完成后的文本
    let description: String

    // 用一个字符串创建实例
    init(stringLiteral value: String) {
        description = value
    }

    // 用一个 StringInterpolation 创建实例
    init(stringInterpolation: StringInterpolation) {
        description = stringInterpolation.output
    }
}
```

我们现在可以使用如下字符串插值创建和使用`HTMLComponent`的实例：

```swift
let text: HTMLComponent = "You should follow me on Twitter \(twitter: "twostraws"), or you can email me at \(email: "paul@hackingwithswift.com")."
 print(text)

// 打印结果如下
Appending You should follow me on Twitter 
Appending twostraws
Appending , or you can email me at 
Appending paul@hackingwithswift.com
Appending .
You should follow me on Twitter <a href="https://twitter/twostraws">@twostraws</a>, or you can email me at <a href="mailto:paul@hackingwithswift.com">paul@hackingwithswift.com</a>.
```

由于`print()`调用分散在内部，您可以看到字符串插值功能的确切工作方式，每个部分触发一个方法调用，并添加到字符串中。

## 动态可调用类型

 [SE-0216](https://github.com/apple/swift-evolution/blob/master/proposals/0216-dynamic-callable.md) 为Swift添加一个新的 `@dynamiccallable` 属性，使其能够将类型标记为可直接调用。它是语法糖，而不是编译器的功劳，有效地转把下面这段代码：

```swift
let result = random(numberOfZeroes: 3)
```

转换为：

```swift
let result = random.dynamicallyCall(withKeywordArguments: ["numberOfZeroes": 3])
```

`@dynamicallable`是`@dynamicmemberlookup`的自然扩展，其作用相同：使Swift代码更容易与动态语言（如Python和Javascript）一起工作。

要将此功能添加到您自己的类型中，您需要添加`@dynamiccalable`属性以及以下一种或两种方法：

```swift
func dynamicallyCall(withArguments args: [Int]) -> Double
func dynamicallyCall(withKeywordArguments args: KeyValuePairs<String, Int>) -> Double
```

第一个用于调用没有参数标签的类型（例如 `a(b, c)`），第二个用于有标签（例如`a(b: cat, c: dog)`）。

`@dynamicallable`对于其方法接受和返回的数据类型非常灵活，允许您从Swift的所有类型安全性中获益，同时仍有一些高级使用空间。因此，对于第一个方法（没有参数标签），您可以使用任何遵循`ExpressibleByArrayLiteral`的类型，如`Array`、`Set`；对于第二个方法（带有参数标签），您可以使用任何遵循`ExpressibleByDictionaryLiteral`的类型，如`Dictionary`和`keyValuePair`。

除了接受各种输入之外，还可以为各种输出提供多个重载——可以返回字符串、整数，等等。只要 Swift 能分辨出使用哪一种，你就可以随心所欲地进行混合和匹配。

让我们来看一个例子。首先，这里有一个`RandomNumberGenerator` Struct，它根据传入的输入生成介于0和某个最大值之间的数字：

```swift
struct RandomNumberGenerator {
    func generate(numberOfZeroes: Int) -> Double {
        let maximum = pow(10, Double(numberOfZeroes))
        return Double.random(in: 0...maximum)
    }
}
```

要将其转换为`@dynamiccallable`，我们将代码改为：

```swift
@dynamicCallable
struct RandomNumberGenerator {
    func dynamicallyCall(withKeywordArguments args: KeyValuePairs<String, Int>) -> Double {
        let numberOfZeroes = Double(args.first?.value ?? 0)
        let maximum = pow(10, numberOfZeroes)
        return Double.random(in: 0...maximum)
    }
}
```

该方法可以用任意数量的参数调用，或者可能是零，因此我们仔细地读取第一个值，并使用 `??` 来确保有一个合理的默认值。

我们现在可以创建`RandomNumberGenerator`的实例，并像函数一样调用它：

```swift
let random = RandomNumberGenerator()
let result = random(numberOfZeroes: 0)
```

如果您使用了`dynamicallyCall(withArguments:)`，或者同时使用，因为您可以将两者都作为一种类型，那么您可以编写以下代码：

```swift
@dynamicCallable
struct RandomNumberGenerator {
    func dynamicallyCall(withArguments args: [Int]) -> Double {
        let numberOfZeroes = Double(args[0])
        let maximum = pow(10, numberOfZeroes)
        return Double.random(in: 0...maximum)
    }
}

let random = RandomNumberGenerator()
let result = random(0)
```

使用`@dynamiccalable`时需要注意一些重要的规则：

- 您可以将它应用于structs, enums, classes, 和 protocols。
- 如果实现了`withKeywordArguments:`，而不实现`withArguments:`，你仍然可以在没有参数标签的情况下调用您的类型-您将只能从键中获得空字符串。
- 如果将`withKeywordArguments:`或`withArguments:`的实现标记为`throws`，则调用该类型也会`throws`。
- 不能将`@dynamiccalable`添加到扩展中，只能添加类型的定义中。
- 您仍然可以向类型中添加其他方法和属性，并正常使用它们。

也许更重要的是，不支持方法解析，这意味着我们必须直接调用类型（例如`random(numberOfZeroes: 5)`），而不是对类型调用特定的方法（例如`random.generate(numberOfZeroes: 5)`）。已经有一些关于使用方法签名添加后者的讨论，例如：

```swift
func dynamicallyCallMethod(named: String, withKeywordArguments: KeyValuePairs<String, Int>)
```

如果这在未来的Swift版本中成为可能，它可能会为测试模拟打开一些非常有趣的可能性。

与此同时，`@dynamiccalable`不太可能广受欢迎，但对于少数希望与Python、Javascript和其他语言进行交互的人来说，它非常重要。

## 处理将来的枚举cases

 [SE-0192](https://github.com/apple/swift-evolution/blob/master/proposals/0192-non-exhaustive-enums.md) 添加区分固定枚举和将来可能更改的枚举的功能。

Swift 的一个安全特性是它要求所有switch语句都是详尽的——它们必须覆盖所有情况。虽然从安全的角度来看，这很好地工作，但在将来添加新的case时，它会导致兼容性问题：系统框架可能会发送您不需要的不同东西，或者您所依赖的代码可能会添加新的case并导致编译中断，因为您的switch语句不再是详尽的了。

有了`@unknown`属性，我们现在可以区分两种细微不同的情况：1）“对于所有其他情况，应该运行此默认情况，因为我不想单独处理它们，”；2）“我想单独处理所有情况，但如果将来出现任何情况，请使用此选项，而不是导致错误。”

例如，有下列的枚举：

```swift
enum PasswordError: Error {
    case short
    case obvious
    case simple
}
```

我们可以使用`switch`来处理这些情况：

```swift
func showOld(error: PasswordError) {
    switch error {
    case .short:
        print("Your password was too short.")
    case .obvious:
        print("Your password was too obvious.")
    default:
        print("Your password was too simple.")
    }
}
```

它显式地处理了`short`和`obvious`两种情况，但将第三种情况`simple`放到的`default`中。

如果将来我们在枚举中添加一个名为old的case，对于以前使用过的密码，我们的`default` case将自动被调用，即使它的打印出来的消息提示并不是真正有意义——密码可能不是太简单。

Swift不能警告我们关于这个代码，因为它在技术上是正确的，所以这个错误很容易被忽略。幸运的是，新的`@unknown`属性完美地修复了它——它只能在`default` case下使用，并且设计为在将来出现新case时运行。

```swift
func showNew(error: PasswordError) {
    switch error {
    case .short:
        print("Your password was too short.")
    case .obvious:
        print("Your password was too obvious.")
    @unknown default:
        print("Your password wasn't suitable.")
    }
}
```

上面的代码现在将发出警告，因为switch不再是详尽的 –— Swift希望我们明确地处理每个case。有帮助的是，这只是一个警告，这正是使该属性如此有用的原因：如果一个框架在将来添加了一个新的case，您将收到关于它的警告，但它不会破坏源代码。

## 扁平化由`try?`产生的嵌套可选类型

 [SE-0230](https://github.com/apple/swift-evolution/blob/master/proposals/0230-flatten-optional-try.md) 修改了`try?`的工作方式，使嵌套可选类型扁平成为常规可选类型。这使得它的工作方式与可选链接和条件类型转换相同。

下面是一个实际的例子，演示了这种变化：

```swift
struct User {
    var id: Int

    init?(id: Int) {
        if id < 1 {
            return nil
        }
        self.id = id
    }

    func getMessages() throws -> String {
        // complicated code here
        return "No messages"
    }
}

let user = User(id: 1)
let messages = try? user?.getMessages()
```

`User` struct有一个可失败的初始化器，因为我们希望确保人们使用有效的ID创建用户。理论上，`getMessages()`方法将包含一些复杂的代码，以便为用户获取所有消息的列表，因此它被标记为 `throws`；我已使它返回一个固定的字符串，以便对代码进行编译。

关键行是最后一行：因为 `user`是可选类型的，所以它使用可选链接，并且因为`getMessages()`可以抛出错误，所以它使用 `try?`要将抛出方法转换为可选方法，我们最终得到一个嵌套的可选类型。在swift 4.2及更早版本中，这会使 `message` 成为 `String??` 类型（可选的可选的S tring）。但从 Swift 5.0开始，如果值已经是可选的，`try?`则不会将其包装在可选值中，因此 `messages` 只是一个`String?`。

这个新特性与可选链接和条件类型转换的现有行为匹配。也就是说，如果需要，可以在一行代码中使用可选的链接很多次，但最终不会得到多个嵌套可选类型。类似地，如果把可选链接和 `as?`结合起来使用， 你最终还是只能一个层次的可选类型，因为这通常是你想要的。

## 检查整数倍数

 [SE-0225](https://github.com/apple/swift-evolution/blob/master/proposals/0225-binaryinteger-iseven-isodd-ismultiple.md) 为 integers 添加了 `isMultiple(of:`方法，允许我们用求余数运算`%`更清楚的方法检查一个数是否是另一个数的倍数。例如：

```swift
let rowNumber = 4

if rowNumber.isMultiple(of: 2) {
    print("Even")
} else {
    print("Odd")
}
```

当然，我们可以使用 `if rowNumber % 2 == 0` 来编写相同的检查，但您必须承认这种写法不是太清晰。

## 使用`compactMapValues()`转换和解包字典的值

 [SE-0218](https://github.com/apple/swift-evolution/blob/master/proposals/0218-introduce-compact-map-values.md) 向字典中添加了新的`compactMapValues()`方法，将数组中的`compactMap()`方法（转换元素，解包结果，并移除所有为nil的结果）与字典中的`mapValues()`方法（保留键不变，但转换对应的值）结合在一起。

举个例子，这里有一个跑步参赛者的字典，以及他们在几秒钟内完成比赛的时间。如果未完成，标记为“DNF”：

```swift
let times = [
    "Hudson": "38",
    "Clarke": "42",
    "Robinson": "35",
    "Hartis": "DNF"
]
```

我们可以使用`compactMapValues()`创建一个新字典，名称和时间为整数，删除一个 DNF 人员：

```swift
let finishers1 = times.compactMapValues { Int($0) }
```

或者，您可以直接将 `Int` 的初始化器传递给`compactMapValues()`，如下所示：

```swift
let finishers2 = times.compactMapValues(Int.init)
```

您还可以使用`compactMapValues()`解包可选类型并放弃nil值，而不执行任何的转换，如：

```swift
let people = [
    "Paul": 38,
    "Sophie": 8,
    "Charlotte": 5,
    "William": nil
]

let knownAges = people.compactMapValues { $0 }
```
