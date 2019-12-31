# 21 - 扩展 (Extensions)

扩展可以在已经存在的类、结构、枚举或者协议上添加新的功能。Swift的扩展类似于OC的分类(不同于OC的是，Swift的扩展没有名字)。

Swift的扩展可以：

- 添加实例计算属性和类型计算属性
- 添加实例方法和类型方法
- 提供新的初始化器
- 定义下标
- 定义并使用新的嵌套类型
- 让一个已经存在的类遵循一个协议

**注意：** 扩展可以添加新的功能，但是不能重写已经存在的方法。

## 扩展语法 (Extension Syntax)

使用`extension`关键字：

```swift
extension SomeType {
    // new functionality to add to SomeType goes here
}
```

可以使用扩展让一个已经存在的类遵循一个或多个协议：

```swift
extension SomeType: SomeProtocol, AnotherProtocol {
    // implementation of protocol requirements goes here
}
```

**注意：** 如果定义一个扩展添加新的功能到已经存在的类，那么这个存在的类的所有实例都可以使用这些新的方法。即使他们是在定义扩展之前创建的。

## 计算属性 (Computed Properties)

下面是一个例子：

```swift
extension Double {
    var km: Double { return self * 1_000.0 }
    var m: Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}
let oneInch = 25.4.mm
print("One inch is \(oneInch) meters")
// Prints "One inch is 0.0254 meters"
let threeFeet = 3.ft
print("Three feet is \(threeFeet) meters")
// Prints "Three feet is 0.914399970739201 meters"
```

所有新添加的属性都是只读属性返回值都是`Double`类型，可以使用加号直接相加：

```swift
let aMarathon = 42.km + 195.m
print("A marathon is \(aMarathon) meters long")
// Prints "A marathon is 42195.0 meters long"
```

**注意：** 扩展可以添加新的计算属性，但是不能添加存储属性或者添加属性观察者到已经存在的属性。

## 初始化器 (Initializers)

扩展可以为一个类添加新的便利初始化器，但是不能添加指定初始化器或者反初始化器到。

**注意：** 一个值类型为所有存储属性都提供默认值，并且没有自定义初始化器，如果我们使用扩展添加初始化器到这个值类型中，我们可以在扩展中调用默认的初始化器和逐一成员初始化器。

下面是一个例子：

```swift
struct Size {
    var width = 0.0, height = 0.0
}
struct Point {
    var x = 0.0, y = 0.0
}
struct Rect {
    var origin = Point()
    var size = Size()
}
```

因为`Rect`结构为所有属性提供了默认值，所以它自动获得默认初始化器和一个逐一成员初始化器。

```swift
let defaultRect = Rect()
let memberwiseRect = Rect(origin: Point(x: 2.0, y: 2.0),
                          size: Size(width: 5.0, height: 5.0))
```

我们可以使用扩展来提供另外一个初始化器：

```swift
extension Rect {
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

可以在扩展的初始化器实现中调用逐一成员初始化器。用扩展的初始化器创建一个`Rect`实例：

```swift
let centerRect = Rect(center: Point(x: 4.0, y: 4.0),
                      size: Size(width: 3.0, height: 3.0))
// centerRect's origin is (2.5, 2.5) and its size is (3.0, 3.0)
```

## 方法 (Methods)

添加一个实例方法：

```swift
extension Int {
    func repetitions(task: () -> Void) {
        for _ in 0..<self {
            task()
        }
    }
}

// 使用Int实例调用扩展的方法
3.repetitions {
    print("Hello!")
}
// Hello!
// Hello!
// Hello!

```

### Mutating 实例方法 (Mutating Instance Methods)

使用扩展添加的实例方法也可以改变实例自己。修改`self`或者它的属性的结构和枚举方法必须标记为`mutating`。

```swift
extension Int {
	mutating func square() {
		self = self * self
	}
}

var someInt = 3
someInt.square()
// someInt is now 9
```

## 下标 (Subscripts)

扩展可以添加新的下标。下面这个例子添加下标到`Int`类型中。下标`[n]`方法从右往左数的第`n`个位置的数字：

- `123456789[0]`返回`9`
- `123456789[1]`返回`8`
- ......

```swift
extension Int {
    subscript(digitIndex: Int) -> Int {
        var decimalBase = 1
        for _ in 0..<digitIndex {
            decimalBase *= 10
        }
        return (self / decimalBase) % 10
    }
}
746381295[0]
// returns 5
746381295[1]
// returns 9
746381295[2]
// returns 2
746381295[8]
// returns 7
```

如果下标索引大于`Int`数字的位数，那么返回值为`0`：

```swift
746381295[9]
// returns 0, as if you had requested:
0746381295[9]
```

## 嵌套类型 (Nested Types)

扩展可以添加嵌套类型到已经存在的类、结构和枚举中：

```swift
extension Int {
    enum Kind {
        case negative, zero, positive
    }
    var kind: Kind {
        switch self {
        case 0:
            return .zero
        case let x where x > 0:
            return .positive
        default:
            return .negative
        }
    }
}
```

嵌套的枚举可以用于`Int`值中：

```swift
func printIntegerKinds(_ numbers: [Int]) {
    for number in numbers {
        switch number.kind {
        case .negative:
            print("- ", terminator: "")
        case .zero:
            print("0 ", terminator: "")
        case .positive:
            print("+ ", terminator: "")
        }
    }
    print("")
}
printIntegerKinds([3, 19, -27, 0, -6, 0, 7])
// Prints "+ + - 0 - 0 + "
```
