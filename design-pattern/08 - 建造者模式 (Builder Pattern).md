> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

建造者模式允许我们一步一步的创建复杂的对象，而不是通过初始化函数一次性创建。这种模式有三个主要的部分组成：

- **Director**：接收输入，并控制Builder；通常是一个View Controller
- **Product**：需要创建的复杂对象
- **Builder**：接受一步一步传入的输入，并负责Product的创建；通常是一个Class类型

### 什么时候使用

当想要一步一步的创建一个复杂对象时，使用建造者模式。

### 简单Demo 

我们来看一个经典的例子：实现汉堡包的制作。**Product**是`Hamburger`；**Director**是员工`Employee`，**Builder**是`HamburgerBuilder`。

#### Product

```swift
struct Hamburger {
    let meat: Meat
    let toppings: Toppings
    let sauces: Sauces
}
extension Hamburger: CustomStringConvertible {
    var description: String {
        return meat.rawValue + " Hamburger"
    }
}

enum Meat: String {
    case beef
    case chicken
}

struct Toppings: OptionSet {
    static let cheese = Toppings(rawValue: 1 << 0)
    static let lettuce = Toppings(rawValue: 1 << 1)
    static let tomatoes = Toppings(rawValue: 1 << 2)

    let rawValue: Int

    init(rawValue: Int) {
        self.rawValue = rawValue
    }
}

struct Sauces: OptionSet {
    static let mustard = Sauces(rawValue: 1 << 0)
    static let ketchup = Sauces(rawValue: 1 << 1)

    let rawValue: Int

    init(rawValue: Int) {
        self.rawValue = rawValue
    }
}
```

`Hamburger`由肉`Meat`、配料`Toppings`和调味汁`Sauces`组成；另外实现了`CustomStringConvertible`，待会我们打印的时候能看到是什么汉堡。。其中肉可以选择牛肉和肌肉中的一种；配料可以多选：奶酪、生菜和番茄；调味汁也可以多选：芥末和番茄酱。

#### Builder

```swift
class HamburgerBuilder {
    private(set) var meat: Meat = .chicken
    private(set) var toppings: Toppings = []
    private(set) var sauces: Sauces = []

    func setMeat(_ meat: Meat) {
        self.meat = meat
    }

    func addToppings(_ toppings: Toppings) {
        self.toppings.insert(toppings)
    }

    func removeToppings(_ toppings: Toppings) {
        self.toppings.remove(toppings)
    }

    func addSauces(_ sauces: Sauces) {
        self.sauces.insert(sauces)
    }

    func removeSauces(_ sauces: Sauces) {
        self.sauces.remove(sauces)
    }

    func build() -> Hamburger {
        return Hamburger(meat: meat,
                         toppings: toppings,
                         sauces: sauces)
    }
}
```

定义了制作汉堡需要的三种原料，并且使用了`private(set)`，防止被外部篡改（我们在写程序的时候，也要有这种思想，只暴露外部需要的api）；还有其他添加或者移除材料的方法；最后是最终完成制作的方法。

#### Director

```swift
class Employee {
    private let builder = HamburgerBuilder()

    func createBeefHamburger() -> Hamburger {
        builder.setMeat(.beef)
        builder.addSauces(.ketchup)
        builder.addToppings([.lettuce, .tomatoes])
        return builder.build()
    }

    func createChickenHamburger() -> Hamburger {
        let builder = HamburgerBuilder()
        builder.setMeat(.chicken)
        builder.addSauces(.mustard)
        builder.addToppings([.lettuce, .tomatoes])
        return builder.build()
    }
}
```

有两个方法分别用于制作鸡肉汉堡和牛肉汉堡。

#### 使用

```swift
let employee = Employee()

let beefHamburger = employee.createBeefHamburger()
print(beefHamburger.description) // beef hamburger

let chickenHamburger = employee.createChickenHamburger()
print(chickenHamburger.description) // chicken hamburger
```

创建一个员工对象，然后制作汉堡就非常简单了。

### 总结

当创建对象时，需要通过一系列的步骤来传入参数时，非常适合使用建造者模式。如果我们的Product不需要很多的参数，或者需要一次性创建，建议还是使用初始化函数。
