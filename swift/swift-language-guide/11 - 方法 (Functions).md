# 方法 (Functions)

类、结构和枚举都可以定义实例方法和类方法。与C和OC一个主要的不同点，就是Swift的结构和枚举能定义方法，而在OC中只有类才可以定义方法。

## 实例方法 (Instance Methods)

实例方法内部可以访问其他实例方法和属性。

```swift
class Counter {
    var count = 0
    func increment() {
        count += 1
    }
    func increment(by amount: Int) {
        count += amount
    }
    func reset() {
        count = 0
    }
}
```

使用点语法调用方法：

```swift
let counter = Counter()
// the initial counter value is 0
counter.increment()
// the counter's value is now 1
counter.increment(by: 5)
// the counter's value is now 6
counter.reset()
// the counter's value is now 0
```

### self属性 (The self Property)

每一个类型的实例都有一个隐式的属性`self`，正好等于实例自己。在它自己的实例方法内部使用`self`来引用当前实例。

```swift
func increment() {
    self.count += 1
}
```

如果我们没有写`self`，Swift会认为你正在引用当前实例的属性或方法。当参数名和实例属性名一样时，我们必须明确写出`self`，以与参数名区分开。

```swift
struct Point {
    var x = 0.0, y = 0.0
    func isToTheRightOf(x: Double) -> Bool {
        return self.x > x
    }
}
let somePoint = Point(x: 4.0, y: 5.0)
if somePoint.isToTheRightOf(x: 1.0) {
    print("This point is to the right of the line where x == 1.0")
}
// Prints "This point is to the right of the line where x == 1.0"
```

### 在实例方法内部修改值类型 (Modifying Value Types from Within Instance Methods)

结构和枚举都是值类型。默认情况下，值类型的属性在自己的实例方法内部是不能修改的。

然而，如果我们要在方法内部修改结构或者枚举的属性，我们可以为那个方法加上*mutating*行为。然后这个方法就能在方法内部修改自己的属性，当方法结束时，任何改变都会写回给最初的那个结构。这个方法还可以完全赋一个新的实例给`self`属性，然后这个新的实例会替换掉已经存在的实例。

使用`mutating`关键字来实现这个行为：

```swift
struct Point {
    var x = 0.0, y = 0.0
    mutating func moveBy(x deltaX: Double, y deltaY: Double) {
        x += deltaX
        y += deltaY
    }
}
var somePoint = Point(x: 1.0, y: 1.0)
somePoint.moveBy(x: 2.0, y: 3.0)
print("The point is now at (\(somePoint.x), \(somePoint.y))")
// Prints "The point is now at (3.0, 4.0)"
```

注意：我们不能使用用一个结构的常量调用mutating方法，因为它的属性值是不能被改变的，即使是可变属性。

```swift
let fixedPoint = Point(x: 3.0, y: 3.0)
fixedPoint.moveBy(x: 2.0, y: 3.0)
// this will report an error
```

### 在Mutating方法内部给self赋值 (Assignto self Within a Mutating Method)

```swift
struct Point {
    var x = 0.0, y = 0.0
    mutating func moveBy(x deltaX: Double, y deltaY: Double) {
        self = Point(x: x + deltaX, y: y + deltaY)
    }
}
```

枚举类型的mutating方法：

```swift
enum TriStateSwitch {
    case off, low, high
    mutating func next() {
        switch self {
        case .off:
            self = .low
        case .low:
            self = .high
        case .high:
            self = .off
        }
    }
}
var ovenLight = TriStateSwitch.low
ovenLight.next()
// ovenLight is now equal to .high
ovenLight.next()
// ovenLight is now equal to .off
```

## 类方法 (Type Methods)

使用`static`来定义类方法，class类型可以使用`class`关键字，以允许子类重写父类的实现。

**注意：** 在OC中，我们只能在class中定义类方法。而在Swift，可以在类、结构和枚举中定义类方法。

```swift
class SomeClass {
	class func someTypeMethod() {
        // type method implementation goes here
	}
}
SomeClass.someTypeMethod()
```

在类方法的方法体中，隐式的`self`属性引用着类型自己，而不是这个类型的实例。
