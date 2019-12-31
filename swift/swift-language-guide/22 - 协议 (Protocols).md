# 22 - 协议 (Protocols)

协议为方法、属性和其他适合特定任务或功能定义了一个蓝图。类、结构和枚举都可以遵循协议来对一些特殊要求提供真实的实现。

## 协议语法 (Protocol Syntax)

语法如下：

```swift
protocol SomeProtocol {
    // protocol definition goes here
}
```

遵循一个或多个协议：

```swift
struct SomeStructure: FirstProtocol, AnotherProtocol {
    // structure definition goes here
}
```

如果一个类有父类，那么把父类放在最前面，协议放后面：

```swift
class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
    // class definition goes here
}
```

## 属性要求 (Property Requirements)

协议可以要求遵循它的类型提供有特定名字和类型的实例属性或者类属性。协议没有要求这些属性一定得是存储属性或者计算属性，它指定了属性的名字和类型，还指定了属性是只读或者是可读可写。

如果协议要求了属性必须是可读可写，那么实现这个协议时不能把属性定义为常量存储属性或者只读计算属性；如果协议要求一个属性必须是可读的，那么实现这个协议时可以把属性定义为任何种类的属性，也可以是可写的。

协议的属性都定义为变量属性，使用`var`关键字：

```swift
protocol SomeProtocol {
	var mustBeSettable: Int { get set }
	var doesNotNeedToBeSettable: Int { get }
}
```

使用`static`定义类型属性：

```swift
protocol AnotherProtocol {
	static var someTypeProperty: Int { get set }
}
```

下面是一个例子：

```swift
protocol FullyNamed {
	var fullName: String { get }
}
```

遵循`FullyNamed`协议：

```swift
struct Person: FullyNamed {
    var fullName: String
}
let john = Person(fullName: "John Appleseed")
// john.fullName is "John Appleseed"
```

下面是一个更复杂的遵循`FullyNamed`协议的例子：

```swift
class Starship: FullyNamed {
    var prefix: String?
    var name: String
    init(name: String, prefix: String? = nil) {
        self.name = name
        self.prefix = prefix
    }
    var fullName: String {
        return (prefix != nil ? prefix! + " " : "") + name
    }
}
var ncc1701 = Starship(name: "Enterprise", prefix: "USS")
// ncc1701.fullName is "USS Enterprise"
```

## 方法要求 (Method Requirements)

协议可以定义一些特定类型的实例方法和类型方法来让遵循它的类型去实现。定义协议方法和定义正常的方法一样，不过协议方法没有方法体。允许使用可变参数，但是参数不能有默认值。

使用`static`来定义类型方法：

```swift
protocol SomeProtocol {
    static func someTypeMethod()
}
```

下面是一个`RandomNumberGenerator`协议：

```swift
protocol RandomNumberGenerator {
    func random() -> Double
}
```

下面是遵循`RandomNumberGenerator`协议的一个类：

```swift
class LinearCongruentialGenerator: RandomNumberGenerator {
    var lastRandom = 42.0
    let m = 139968.0
    let a = 3877.0
    let c = 29573.0
    func random() -> Double {
        lastRandom = ((lastRandom * a + c).truncatingRemainder(dividingBy:m))
        return lastRandom / m
    }
}
let generator = LinearCongruentialGenerator()
print("Here's a random number: \(generator.random())")
// Prints "Here's a random number: 0.37464991998171"
print("And another one: \(generator.random())")
// Prints "And another one: 0.729023776863283"
```

## Mutating 方法要求 (Mutating Method Requirements)

有时候我们需要在方法中改变实例。在方法前面加上`mutating`来表示这个方法可以修改实例或者实例的属性。

**注意：** 如果把协议的实例方法标记为`mutating`，在遵循这个协议的类中实现`mutating`方法无需加上`mutating`关键字。`mutating`只适用于结构和枚举类型。

下面是一个`Togglable`协议：

```swift
protocol Togglable {
	mutating func toggle()
}
```

下面是遵循`Togglable`协议的枚举`OnOffSwitch`：

```swift
enum OnOffSwitch: Togglable {
	case off, on
	mutating func toggle() {
		switch self {
			case .off:
				self = .on
			case .on:
				self = .off
		}
	}
}
var lightSwitch = OnOffSwitch.off
lightSwitch.toggle()
// lightSwitch is now equal to .on
```

## 初始化器要求 (Initializer Requirements)

协议可以指定遵循这个协议的类型初始化器：

```swift
protocol SomeProtocol {
	init(someParameter: Int)
}
```

### Class实现协议的初始化器要求 (Class Implementation of Protocol Initializer Requirements)

我们可以实现协议的初始化器并作为类的指定初始化器或者便利初始化器，不管是哪一种初始化器，都需要用`require`标记：

```swift
class SomeClass: SomeProtocol {
	required init(someParameter: Int) {
		// initializer implementation goes here
	}
}
```

`required`关键字保证了遵循这个协议的类的所有子类有一个对协议指定的初始化器有明确的实现，这样子类也遵循这个协议。

**注意：** 如果类对协议指定的初始化器的实现用`final`标记，那就无需再用`required`标记，因为`final`标记的初始化器是不能被继承的。

如果一个子类重写了父类的指定初始化器，并重写后的初始化器刚好匹配协议指定的初始化器，那么这个初始化器需要用`required`和`override`标记。

```swift
protocol SomeProtocol {
	init()
}

class SomeSuperClass {
	init() {
        // initializer implementation goes here
	}
}

class SomeSubClass: SomeSuperClass, SomeProtocol {
    // "required" from SomeProtocol conformance; "override" from SomeSuperClass
	required override init() {
        // initializer implementation goes here
	}
}
```

### 可能失败的初始化器要求 (Failable Initializer Requirements)

协议可以定义可能失败的初始化器要求。在遵循这个协议的类型中，可以用可能失败的初始化器或者正常的初始化器来实现协议的可能失败的初始化器；可以用隐式强制解包可能失败的初始化器或者正常的初始化器来实现协议的正常的初始化器。

## 协议作为类型 (Protocols as Types)

协议实际上是不实现任何功能的。尽管如此，我们创建的任何协议，都可以是一个完全成熟的类型。

因为协议是一个类型，所以我们可以在很多地方用到协议：

- 作为方法和初始化器的参数类型或返回值类型
- 作为常量、变量或者属性的类型
- 作为数组、字典或者其他集合的元素类型

下面是一个例子：

```swift
class Dice {
    let sides: Int
    let generator: RandomNumberGenerator
    init(sides: Int, generator: RandomNumberGenerator) {
        self.sides = sides
        self.generator = generator
    }
    func roll() -> Int {
        return Int(generator.random() * Double(sides)) + 1
    }
}
```

`generator`属性是`RandomNumberGenerator`类型，所以我们可以把任何遵循`RandomNumberGenerator`协议的类型实例赋给它，没有其他太多要求，只要这个实例的类型遵循了`RandomNumberGenerator`协议。

```swift
var d6 = Dice(sides: 6, generator: LinearCongruentialGenerator())
for _ in 1...5 {
    print("Random dice roll is \(d6.roll())")
}
// Random dice roll is 3
// Random dice roll is 5
// Random dice roll is 4
// Random dice roll is 5
// Random dice roll is 4
```

## 代理 (Delegation)

代理是一种设计模式，使用代理可以把一个类或结构把它们负责的任务转移给另外一个类型的实例去做。代理模式是通过定义协议实现的，这个协议封装了代理需要做的工作。代理模式可以用来响应一些特定的行为，或者从外部资源获取数据，但是不需要知道外部资源的基本类型。

下面是骰子游戏需要的两个协议：

```swift
protocol DiceGame {
	var dice: Dice { get }
	func play()
}

protocol DiceGameDelegate {
	func gameDidStart(_ game: DiceGame)
	func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int)
	func gameDidEnd(_ game: DiceGame)
}
```

`DiceGame`协议可以用于涉及到骰子的游戏；`DiceGameDelegate`协议可以用于跟踪`DiceGame`的过程。

下面是一个`Snakes`和`Ladders`的游戏：

```swift
class SnakesAndLadders: DiceGame {
    let finalSquare = 25
    let dice = Dice(sides: 6, generator: LinearCongruentialGenerator())
    var square = 0
    var board: [Int]
    init() {
        board = Array(repeating: 0, count: finalSquare + 1)
        board[03] = +08; board[06] = +11; board[09] = +09; board[10] = +02
        board[14] = -10; board[19] = -11; board[22] = -02; board[24] = -08
    }
    var delegate: DiceGameDelegate?
    func play() {
        square = 0
        delegate?.gameDidStart(self)
        gameLoop: while square != finalSquare {
            let diceRoll = dice.roll()
            delegate?.game(self, didStartNewTurnWithDiceRoll: diceRoll)
            switch square + diceRoll {
            case finalSquare:
                break gameLoop
            case let newSquare where newSquare > finalSquare:
                continue gameLoop
            default:
                square += diceRoll
                square += board[square]
            }
        }
        delegate?.gameDidEnd(self)
    }
}
```

`SnakesAndLadders`遵循了`DiceGame`协议。

下面是`DiceGameTracker`，遵循了`DiceGameDelegate`协议：

```swift
class DiceGameTracker: DiceGameDelegate {
    var numberOfTurns = 0
    func gameDidStart(_ game: DiceGame) {
        numberOfTurns = 0
        if game is SnakesAndLadders {
            print("Started a new game of Snakes and Ladders")
        }
        print("The game is using a \(game.dice.sides)-sided dice")
    }
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int) {
        numberOfTurns += 1
        print("Rolled a \(diceRoll)")
    }
    func gameDidEnd(_ game: DiceGame) {
        print("The game lasted for \(numberOfTurns) turns")
    }
}
```

`DiceGameTracker`实现了`DiceGameDelegate`协议的所有方法。

下面是`DiceGameTracker`的使用情况：

```swift
let tracker = DiceGameTracker()
let game = SnakesAndLadders()
game.delegate = tracker
game.play()
// Started a new game of Snakes and Ladders
// The game is using a 6-sided dice
// Rolled a 3
// Rolled a 5
// Rolled a 4
// Rolled a 5
// The game lasted for 4 turns
```

## 使用扩展遵循协议 (Adding Protocol Conformance with an Extension)

可以使用扩展来遵循一个协议，不必访问已存在类型的源代码。

**注意：** 当在一个类型的扩展遵循了协议，这个类型会自动遵循这个协议。

例如下面这个协议`TextRepresentable`：

```swift
protocol TextRepresentable {
    var textualDescription: String { get }
}
```

可以通过扩展`Dice`来遵循`TextRepresentable`协议：

```swift
extension Dice: TextRepresentable {
    var textualDescription: String {
        return "A \(sides)-sided dice"
    }
}
```

同样地，可以通过扩展`SnakesAndLadders`来遵循`TextRepresentable`协议：

```swift
extension SnakesAndLadders: TextRepresentable {
    var textualDescription: String {
        return "A game of Snakes and Ladders with \(finalSquare) squares"
    }
}
print(game.textualDescription)
// Prints "A game of Snakes and Ladders with 25 squares"
```

### 使用扩展声明遵循协议 (Declaring Protocol Adoption with an Extension)

如果一个类型已经实现了协议的所有要求，但是没有声明它已经遵循了这个协议，可以使用空扩展来实现：

```swift
struct Hamster {
    var name: String
    var textualDescription: String {
        return "A hamster named \(name)"
    }
}

extension Hamster: TextRepresentable {}
```

## 协议类型集合 (Collections of Protocol Types)

协议可以作为集合元素的类型，例如：

```swift
let things: [TextRepresentable] = [game, d12, simonTheHamster]
```

便利数组的元素，然后打印元素各自的`textualDescription`：

```swift
for thing in things {
    print(thing.textualDescription)
}
// A game of Snakes and Ladders with 25 squares
// A 12-sided dice
// A hamster named Simon
```

注意：`thing`常量是`TextRepresentable`类型，不是`Dice`、`DiceGame`或者`Hamster`，即使实际上是其中的一种类型。尽管如此，因为`thing`是`TextRepresentable`类型，并且它有`textualDescription`属性，所以我们可以直接访问`thing.textualDescription`。

## 协议继承 (Protocol Inheritance)

一个协议可以继承一个或多个协议，语法如下：

```swift
protocol InheritingProtocol: SomeProtocol, AnotherProtocol {
    // protocol definition goes here
}
```

下面是一个例子：

```swift
protocol PrettyTextRepresentable: TextRepresentable {
    var prettyTextualDescription: String { get }
}
```

`SnakesAndLadders`类遵循`PrettyTextRepresentable`协议：

```swift
extension SnakesAndLadders: PrettyTextRepresentable {
    var prettyTextualDescription: String {
        var output = textualDescription + ":\n"
        for index in 1...finalSquare {
            switch board[index] {
            case let ladder where ladder > 0:
                output += "▲ "
            case let snake where snake < 0:
                output += "▼ "
            default:
                output += "○ "
            }
        }
        return output
    }
}
```

这个扩展提供了`prettyTextualDescription`的实现。是`PrettyTextRepresentable`类型的任何东西都必须是`TextRepresentable`类型，所以`prettyTextualDescription`的实现始于`TextRepresentable`的属性`textualDescription`。

打印`prettyTextualDescription`属性，结果如下：

```swift
print(game.prettyTextualDescription)
// A game of Snakes and Ladders with 25 squares:
// ○ ○ ▲ ○ ○ ▲ ○ ○ ▲ ▲ ○ ○ ○ ▼ ○ ○ ○ ○ ▼ ○ ○ ▼ ○ ▼ ○
```

## 只适用于Class类型的协议 (Class-Only Protocols)

我们可以使用`class`关键字来限制一个协议能被class类型遵循，`class`必须放在最前面：

```swift
protocol SomeOnlyProtocol: class, SomeInheritantedProtocol {
    // class-only protocol definition goes here
}
```

## 协议组合 (Protocol Composition)

要求一个类型一次性遵循多个协议是非常有用的。使用`SomeProtocol & AnotherProtocol`来组合协议，我们想要组合多少个就组合多少个，只要使用`&`隔开即可。

下面是一个例子：

```swift
protocol Named {
	var name: String { get }
}

protocol Aged {
	var age: Int { get }
}

struct Person: Named, Aged {
	var name: String
	var age: Int
}

func wishHappyBirthday(to celebrator: Named & Aged) {
    print("Happy birthday, \(celebrator.name), you're \(celebrator.age)!")
}

let birthdayPerson = Person(name: "Malcolm", age: 21)
wishHappyBirthday(to: birthdayPerson)
// Prints "Happy birthday, Malcolm, you're 21!"
```

`celebrator`的参数类型是`Named & Aged`，意味着这个参数要同时遵循`Named`和`Aged`协议。不管具体类型是什么，只要遵循这两个协议即可。

**注意：** 协议组合并不是定义了一个新的、永久的协议类型。他们只是一个结合了多个协议要求的暂时本地协议。

## 检查协议一致性 (Checking for Protocol Conformance)

使用`is`和`as`来检查协议的一致性，并且转型到一个具体的协议：

- 如果一个实例遵循了一个协议，`is`运算符返回`true`，否则返回`false`
- `as?`向下转型运算符返回一个协议的可选类型，如果实例不遵循那个协议，返回`nil`
- `as!`向下转型运算符强制向下转型到那个协议，如果转型不成功，出发运行时错误

例如下面这个例子：

```swift
protocol HasArea {
	var area: Double { get } 
}

class Circle: HasArea {
    let pi = 3.1415927
    var radius: Double
    var area: Double { return pi * radius * radius }
    init(radius: Double) { self.radius = radius }
}

class Country: HasArea {
    var area: Double
    init(area: Double) { self.area = area }
}
```

下面是一个不遵循`HasArea`协议的类`Animal`:

```swift
class Animal {
    var legs: Int
    init(legs: Int) { self.legs = legs }
}
```

创建三个实例，并把他们放入`[AnyObject]`类型的数组中：

```swift
let objects: [AnyObject] = [
    Circle(radius: 2.0),
    Country(area: 243_610),
    Animal(legs: 4)
]
```

便利这个数组，使用`as?`向下转型：

```swift
for object in objects {
    if let objectWithArea = object as? HasArea {
        print("Area is \(objectWithArea.area)")
    } else {
        print("Something that doesn't have an area")
    }
}
// Area is 12.5663708
// Area is 243610.0
// Something that doesn't have an area
```

## 可选协议要求 (Optional Protocol Requirements)

我们可以定义可选协议要求，这些可选的要求不一定要有对应的实现。使用`optional`关键字来定义可选的协议要求。协议和协议要求都必须使用`@objc`属性标记。注意：`@objc`协议只能被继承于OC的类或者其他`@objc`类采用。不能被结构和枚举采用。

当我们在使用可选要求的方法或者属性时，他的类型自动变为可选类型。例如`(Int) -> String`变成`((Int) -> String)?`。

可以使用可选链来调用可选类型协议的要求。

```swift
@objc protocol CounterDataSource {
	@objc optional func increment(forCount count: Int) -> Int
	@objc optional var fixedIncrement: Int { get }
}

class Counter {
	var count = 0
	var dataSource: CounterDataSource?
	func increment() {
        if let amount = dataSource?.increment?(forCount: count) {
            count += amount
        } else if let amount = dataSource?.fixedIncrement {
            count += amount
        }
	}
}
```

`increment()`使用可选链来尝试调用`increment(forCount:)`。

下面对`CounterDataSource`的简单实现，实现了`fixedIncrement`属性：

```swift
class ThreeSource: NSObject, CountDataSource {
	let fixedIncrement = 3
}
```

创建一个`Counter`实例：

```swift
var counter = Counter()
counter.dataSource = ThreeSource()
for _ in 1...4 {
    counter.increment()
    print(counter.count)
}
// 3
// 6
// 9
// 12
```

下面是一个更复杂的类`TowardsZeroSource`：

```swift
@objc class TowardsZeroSource: NSObject, CounterDataSource {
    func increment(forCount count: Int) -> Int {
        if count == 0 {
            return 0
        } else if count < 0 {
            return 1
        } else {
            return -1
        }
    }
}
```

创建一个`TowardsZeroSource`实例：

```swift
counter.count = -4
counter.dataSource = TowardsZeroSource()
for _ in 1...5 {
    counter.increment()
    print(counter.count)
}
// -3
// -2
// -1
// 0
// 0
```

## 协议扩展 (Protocol Extensions)

协议可以被扩展以提供属性和方法实现，这可以允许我们定义协议的行为。

例如，扩展之前的`RandomNumberGenerator`协议：

```swift
extension RandomNumberGenerator {
	func RandomBool() -> Bool {
		return random() > 0.5
	}
}
```

创建了协议的扩展以后，所有遵循这个协议的类型自动获得这个扩展方法的实现，无需做任何修改：

```swift
let generator = LinearCongruentialGenerator()
print("Here's a random number: \(generator.random())")
// Prints "Here's a random number: 0.37464991998171"
print("And here's a random Boolean: \(generator.randomBool())")
// Prints "And here's a random Boolean: true"
```

### 提供默认实现 (Providing Default Implementations)

我们可以使用协议扩展来提供默认的实现给原有协议定义的方法和属性。如果遵循了这个协议的类型提供了自己的实现，那么将会使用这个类型自己的实现，而不是使用扩展的实现。

**注意：** 扩展提供了默认实现的协议要求不同于可选协议要求，虽然遵循这个协议的类型都不需要提供自己的实现，但是有默认实现的要求可以直接被调用，而不用使用可选绑定。

例如，继承于`TextRepresentable`的`PrettyTextRepresentable`协议，可以通扩展来提供`prettyTextualDescription`属性的默认实现：

```swift
extension PrettyTextRepresentable  {
    var prettyTextualDescription: String {
        return textualDescription
    }
}
```

### 向协议扩展添加约束 (Adding Constraint to Protocol Extensions)

当定义一个协议的扩展时，可以指定一些约束来要求遵循这个协议的类型必须满足这些约束条件才能访问扩展里定义的属性和方法。使用`where`语句来添加约束。

例如，定义一个`Collection`协议的扩展，并限定元素遵循`TextRepresentable`协议的集合才可以访问扩展里的属性和方法：

```swift
extension Collection where Iterator.Element: TextRepresentable {
	var textualDescription: String {
        let itemsAsText = self.map { $0.textualDescription }
        return "[" + itemsAsText.joined(separator: ", ") + "]"
	}
}
```

之前的`Hamster`结构遵循了`TextRepresentable`协议：

```swift
let murrayTheHamster = Hamster(name: "Murray")
let morganTheHamster = Hamster(name: "Morgan")
let mauriceTheHamster = Hamster(name: "Maurice")
let hamsters = [murrayTheHamster, morganTheHamster, mauriceTheHamster]
```

因为`Array`遵循了`Collection`协议，并且里面的元素遵循`TextRepresentable`协议，所以`hamsters`能直接使用`textualDescription`属性：

```swift
print(hamsters.textualDescription)
// Prints "[A hamster named Murray, A hamster named Morgan, A hamster named Maurice]"
```

**注意：** 如果有多个有限制的协议扩展为同一个方法或属性提供了实现，并且有一个类型遵循了这些协议，Swift将会使用对应最专用限制的实现。
