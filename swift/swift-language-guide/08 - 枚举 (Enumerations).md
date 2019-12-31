# 枚举 (Enumerations)

## 枚举语法 (Enumeration Syntax)

```swift
enum CompassPoint {
    case north
    case south
    case east
    case west
}
```

**注意：** 不同于C和OC，Swift枚举的每一种情况不用给一个默认整数值。在上面的这个例子中，`north`、`south`、`east`和`west`不会隐式地等于`0`、`1`、`2`和`3`。

多个情况可以写在一行：

```swift
enum Planet {
    case mercury, venus, earth, mars, jupiter, saturn, uranus, neptune
}
```

访问枚举的其中一种情况：

```swift
var directionToHead = CompassPoint.west
```

`directionToHead`被推断为`CompassPoint`类型。一旦`directionToHead`被声明为`CompassPoint`，我们就可以使用更简短的方式改变它的值：

```swift
directionToHead = .east
```

## 使用Switch语句来匹配枚举值 (Matching Enumeration Values with a Switch Statement)

```swift
directionToHead = .south
switch directionToHead {
case .north:
    print("Lots of planets have a north")
case .south:
    print("Watch out for penguins")
case .east:
    print("Where the sun rises")
case .west:
    print("Where the skies are blue")
}
// Prints "Watch out for penguins"
```

## 关联值 (Associated Values)

可以使用Swift的枚举来存储任何类型的关联值，而且每种情况的关联值可以不一样。

例如，有一个库存跟踪系统，需要根据商品的条形码和二维码来跟踪商品。

条形码是以UPC格式(使用数字0到9)给商品打标签，每个条形码最前面是代表数字系统的一个数字，接着是代表厂商代码的5个数字和代表商品代码的5个数字，最后代表检查代码的一个数字(用于验证条形码的数字已经正确的扫描)。

![条形码](http://upload-images.jianshu.io/upload_images/2057254-4b2dae49c2ad48cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

二维码使用QR代码格式(使用ISO 8859-1的字符，并且能编码一个最多2953个字符的字符串)给商品打标签。

![二维码](http://upload-images.jianshu.io/upload_images/2057254-616a5de1b1fdea07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于这个库存跟踪系统，如果能把UPC条形码存储为一个多元组、把二维码存储为一个字符串，将会非常方便。

```swift
enum Barcode {
	case upc(Int, Int, Int, Int)
	case qrCode(String)
}
```

`Int`和`String`是关联值的类型。

```swift
// 创建新的条形码
var productBarcode = Barcode.upc(8, 85909, 51226, 3)
productBarcode = Barcode.qrCode("ABCDEFGHIJKLMNOP")

// 使用switch语句匹配
switch productBarcode {
	case .upc(let numberSystem, let manufacturer, let product, let check):
		print("UPC: \(numberSystem), \(manufacturer), \(product), \(check).")
	case .qrCode(let productCode):
		print("QR code: \(productCode).")
}
// Prints "QR code: ABCDEFGHIJKLMNOP."
```

如果所有的关联值都想被分解为常量或变量，可以使用下面更简短的方式：

```swift
switch productBarcode {
	case let .upc(numberSystem, manufacturer, product, check):
		print("UPC: \(numberSystem), \(manufacturer), \(product), \(check).")
	case var .qrCode(productCode):
		print("QR code: \(productCode).")
}
// Prints "QR code: ABCDEFGHIJKLMNOP."
```

## 原始值 (Raw Values)

与关联值类似的，枚举的每一种情况还可以有默认值(被称为*原始值*)，并且每个默认值的类型相同。

```swift
enum ASCIIControlCharacter: Character {
    case tab = "\t"
    case lineFeed = "\n"
    case carriageReturn = "\r"
}
```

`ASCIIControlCharacter`的原始值被定义为`Character`类型。

**原始值可以是字符串、字符或者是整数和浮点型。在枚举中，每一个原始值必须是唯一的。**

**注意：** 原始值和关联值是不一样的。原始值是在第一次定义枚举时预先设置的值，一个特定情况的原始值总是相同的。关联值是在基于枚举情况创建常量或变量的时候设置的，而且可以是不同的。

### 隐式地给原始值赋值 (Implicitly Assigned Raw Values)

当枚举存储的原始值是`Integer`或者`String`类型时，我们不必明确地给每一种情设置原始值。

当使用`Integer`作为原始值的类型，每个情况的原始值是前面情况的原始值加1。如果第一个情况没有设置原始值，那么默认就是`0`。

```swift
enum Planet: Int {
    case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
}
```

当使用`String`类型作为原始值的类型，每个情况默认的原始值就是情况自己的名称。

```swift
enum CompassPoint: String {
    case north, south, east, west
}
```

使用`rawValue`属性来访问每一个情况的原始值：

```swift
let earthOrder = Planet.earth.rawValue
// earthOrder is 3

let sunsetDirection = CompassPoint.west.rawValue
// sunsetDirectionis "west"
```

### 使用原始值初始化 (Initializing from a Raw Value)

如果定义了一个有原始值类型的枚举，那么这个枚举将会自动拥有一个构造函数，这个构造函数有一个参数类型与原始值类型相同的参数，并返回枚举的一个情况或者`nil`。

```swift
let possiblePlanet = Planet(rawValue: 7)
// possiblePlanet is of type Planet? and equals Planet.uranus
```

如果使用`11`来寻找一个planet，那么这个默认的构造函数将返回`nil`：

```swift
let positionToFind = 11
if let somePlanet = Planet(rawValue: positionToFind) {
    switch somePlanet {
    case .earth:
        print("Mostly harmless")
    default:
        print("Not a safe place for humans")
    }
} else {
    print("There isn't a planet at position \(positionToFind)")
}
// Prints "There isn't a planet at position 11"
```

## 递归枚举 (Recursive Enumerations)

一个枚举使用这个枚举的实例作为关联值，那么这个枚举就被称为递归枚举。在使用了枚举实例作为关联值的情况前面加上`indirect`关键字，来表明这个枚举情况是递归的。

```swift
enum ArithmeticExpression {
    case number(Int)
    indirect case addition(ArithmeticExpression, ArithmeticExpression)
    indirect case multiplication(ArithmeticExpression, ArithmeticExpression)
```

也可以在`enum`前面加上`indirect`来表明所有的情况都是递归的：

```swift
indirect enum ArithmeticExpression {
	case number(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}
```

这个枚举可以存储三种类型对的数学表达式：一个纯数字、加法、乘法。`addition`和`multiplication`有两个关联值，这两个关联值也是数学表达式，这种类型的表达式可以用来嵌套表达式。例如`(5 + 4) * 2`，右边是乘法，左边的是加法。因为数据是嵌套的，用来存储嵌套数据的枚举也需要支持嵌套，这也就意味着枚举必须是可递归的。例如下面这个为`(5 + 4) * 2`创建的可递归枚举：

```swift
let five = ArithmeticExpression.number(5)
let four = ArithmeticExpression.number(4)
let sum = ArithmeticExpression.addition(five, four)
let product = ArithmeticExpression.multiplication(sum, ArithmeticExpression.numaber(2))
```

一个递归方法可以非常直接的用来处理有递归结构的数据。例如：

```swift
func evaluate(_ expression: ArithmeticExpression) -> Int {
	switch expression {
		case let .number(value):
			return value
		case let .addition(left, right):
			return evalute(left) + evalue(right)
		case let .multiplication(left, right):
			return evalute(left) * evalute(right)
	}
}

print(evaluate(product))
// Prints "18"
```
