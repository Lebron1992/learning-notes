### 控制流 (Control Flow)

#### For-in循环 (For-in Loop)

```swift
for index in 1...5 {
    print("\(index) times 5 is \(index * 5)")
}
// 1 times 5 is 5
// 2 times 5 is 10
// 3 times 5 is 15
// 4 times 5 is 20
// 5 times 5 is 25
```

如果不需要序列的每一个值，使用`_`代替变量的名字：

```swift
let base = 3
let power = 10
var answer = 1
for _ in 1...power {
	answer *= base
}
print("\(base) to the power of \(power) is \(answer)")
// Prints "3 to the power of 10 is 59049"
```

使用`for-in`遍历数组的所有元素：

```swift
let names = ["Anna", "Alex", "Brian", "Jack"]
for name in names {
    print("Hello, \(name)!")
}
// Hello, Anna!
// Hello, Alex!
// Hello, Brian!
// Hello, Jack!
```

使用`for-in`遍历字典的所有键值对：

```swift
let numberOfLegs = ["spider": 8, "ant": 6, "cat": 4]
for (animal, legCount) in numberOfLegs {
    print("\(animalName)s have \(legCount) legs")
}
// ants have 6 legs
// spiders have 8 legs
// cats have 4 legs
```

#### While循环 (While Loops)

Swift提供了两种While循环：

- `while`：执行循环体之前检查循环条件
- `repeat-while`：执行完循环体之后，再检查循环条件

##### While

`while`循环的通用形式：

```swift
while condiction {
	statements
}
```

##### Repeat-While

`repeat-while`循环的通用形式：

```swift
repeat {
	statements
} while condition
```

#### 条件语句

##### If

```swift
temperatureInFahrenheit = 40
if temperatureInFahrenheit <= 32 {
    print("It's very cold. Consider wearing a scarf.")
} else {
    print("It's not that cold. Wear a t-shirt.")
}
// Prints "It's not that cold. Wear a t-shirt."
```

将多个`if`语句串联在一起：

```swift
temperatureInFahrenheit = 90
if temperatureInFahrenheit <= 32 {
    print("It's very cold. Consider wearing a scarf.")
} else if temperatureInFahrenheit >= 86 {
    print("It's really warm. Don't forget to wear sunscreen.")
} else {
    print("It's not that cold. Wear a t-shirt.")
}
// Prints "It's really warm. Don't forget to wear sunscreen."
```

##### Switch

```swift
let someCharacter: Character = "z"
switch someCharacter {
case "a":
    print("The first letter of the alphabet")
case "z":
    print("The last letter of the alphabet")
default:
    print("Some other character")
}
// Prints "The last letter of the alphabet"
```

在Swift中，下面的写法是错误的：

```swift
let anotherCharacter: Character = "a"
switch anotherCharacter {
case "a": // Invalid, the case has an empty body
case "A":
    print("The letter A")
default:
    print("Not the letter A")
}
// This will report a compile-time error.
```

因为不同于C语言中的`switch`，这个`Switch`语句不能同时匹配`a`和`A`。如果想使用一个case同时满足`a`和`A`，需要写成下面这种形式：

```swift
let anotherCharacter: Character = "a"
switch anotherCharacter {
case "a", "A":
    print("The letter A")
default:
    print("Not the letter A")
}
// Prints "The letter A"
```

##### 间隔匹配 (Interval Matching)

```swift
let approximateCount = 62
let countedThings = "moons orbiting Saturn"
var naturalCount: String
switch approximateCount {
	case 0: naturalCount = "no"
	case 1..<5: naturalCount = "a few"
	case 5..<12: naturalCount = "several"
	case 12..<100: naturalCount = "dozens of"
	case 100..<1000: naturalCount = "hundreds of"
	default: naturalCount = "many"
}
print("There are \(naturalCount) \(countedThings).")
// Prints "There are dozens of moons orbiting Saturn."
```

#### 多元组 (Tuples)

```swift
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    print("(0, 0) is at the origin")
case (_, 0):
    print("(\(somePoint.0), 0) is on the x-axis")
case (0, _):
    print("(0, \(somePoint.1)) is on the y-axis")
case (-2...2, -2...2):
    print("(\(somePoint.0), \(somePoint.1)) is inside the box")
default:
    print("(\(somePoint.0), \(somePoint.1)) is outside of the box")
}
// Prints "(1, 1) is inside the box"
```

![coordinate](http://upload-images.jianshu.io/upload_images/2057254-ca44740de1e65752.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实际上`(0, 0)`可以满足这四个情况，然而只有第一个满足`(0, 0)`的情况才会被使用。其他后面的会被忽略。

##### 值绑定 (Value Bindings)

```swift
let anotherPoint = (2, 0)
switch anotherPoint {
case (let x, 0):
    print("on the x-axis with an x value of \(x)")
case (0, let y):
    print("on the y-axis with a y value of \(y)")
case let (x, y):
    print("somewhere else at (\(x), \(y))")
}
// Prints "on the x-axis with an x value of 2"
```

##### Where

在`switch`还可以使用`where`分句来添加额外的条件。

```swift
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
case let (x, y) where x == y:
    print("(\(x), \(y)) is on the line x == y")
case let (x, y) where x == -y:
    print("(\(x), \(y)) is on the line x == -y")
case let (x, y):
    print("(\(x), \(y)) is just some arbitrary point")
}
// Prints "(1, -1) is on the line x == -y"
```

##### 复合情况 (Compound Cases)

```swift
let someCharacter: Character = "e"
switch someCharacter {
case "a", "e", "i", "o", "u":
    print("\(someCharacter) is a vowel")
case "b", "c", "d", "f", "g", "h", "j", "k", "l", "m",
     "n", "p", "q", "r", "s", "t", "v", "w", "x", "y", "z":
    print("\(someCharacter) is a consonant")
default:
    print("\(someCharacter) is not a vowel or a consonant")
}
// Prints "e is a vowel"
```

还可以在复合情况使用值绑定：

```swift
let stillAnotherPoint = (9, 0)
switch stillAnotherPoint {
	case (let distance, 0), (0, let distance):
	print("On an axis, \(distance) from the origin")
	default:
	print("No on an axis")
}
```

#### 控制转换语句 (Control Transfer Statements)

控制转换语句可以改变代码的执行顺序。Swift有5个控制转换语句：

- continue
- break
- fallthrough
- return
- throw

##### Continue

`continue`告诉循环停止正在做的事，然后继续执行下一个遍历：

```swift
let puzzleInput = "great minds think alike"
var puzzleOutput = ""
let charactersToRemove: [Character] = ["a", "e", "i", "o", "u", " "]
for character in puzzleInput.characters {
    if charactersToRemove.contains(character) {
        continue
    } else {
        puzzleOutput.append(character)
    }
}
print(puzzleOutput)
// Prints "grtmndsthnklk"
```

##### Break

`break`可以让整个控制流立即停止。

###### 在循环语句中使用Break (Break in Loop Statement)

当在循环语句中使用时，`break`停止整个循环，然后跳到循环下面的代码。

###### 在Switch语句中使用Break (Break in Switch Statement)

当在Switch语句中使用时，`break`停止整个Switch语句，然后跳到Switch语句下面的代码。

##### Fallthrough

只要第一个匹配的情况执行完成，整个Swift的Switch语句就会停止。在C语言中，需要在每一个case最后面加上`break`来防止跳到下一个case。如果我们需要C语言风格的往下跳到下一个case的功能，可以使用`fallthrough`。

```swift
let integerToDescribe = 5
var description = "The number \(integerToDescribe) is"
switch integerToDescribe {
	case 2, 3, 5, 7, 11, 13, 17, 19:
	description += "a prime number, and also"
	fallthrough
	default:
	description += "an integer."
}
print(description)
// Prints "The number 5 is a prime number, and also an integer."
```

**注意：** `fallthrough`关键字不会检查下一个case的条件是否满足，而是直接执行下一个case的代码。

##### 有标签的语句 (Labeled Statements)

在Swift中，我们可以把循环和条件语句嵌套到其他循环和条件语句里面，来实现更复杂的控制流结构。当然，循环和条件语句可以各自使用`break`来提前终止自己的执行。所以，明确的说明想要终止哪一个循环或者条件语句非常重要。

为了达到这个目的，我们可以给循环和条件语句定义一个标签：

```swift
label name: while condition {
	statements
}
```

#### 提前退出 (Early Exit)

`guard`语句就像`if`语句一样，根据一个布尔值来决定是否执行里面的代码。我们可以使用`guard`语句来要求一个条件必须是`true`，然后`guard`语句后面的代码才能被执行。如果条件是`false`，那么会执行`else`分句的代码。

```swift
func greet(person: [String: String]) {
    guard let name = person["name"] else {
        return
    }
    
    print("Hello \(name)!")
    
    guard let location = person["location"] else {
        print("I hope the weather is nice near you.")
        return
    }
    
    print("I hope the weather is nice in \(location).")
}
 
greet(person: ["name": "John"])
// Prints "Hello John!"
// Prints "I hope the weather is nice near you."
greet(person: ["name": "Jane", "location": "Cupertino"])
// Prints "Hello Jane!"
// Prints "I hope the weather is nice in Cupertino."
```

使用`guard`语句可以提高代码的可读性，可以使我们避免包装很多个`else`代码块。

#### 检查API的可用性 (Checking API Availability)

```swift
if #available(iOS 10, macOS 10.12, *) {
    // Use iOS 10 APIs on iOS, and use macOS 10.12 APIs on macOS
} else {
    // Fall back to earlier iOS and macOS APIs
}
```

`*`不能省略，是用来指定其他任何平台。
