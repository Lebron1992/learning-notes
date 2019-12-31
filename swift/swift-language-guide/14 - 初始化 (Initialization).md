# 初始化 (Initialization)

初始化是准备一个类、结构或者枚举实例的过程。在这个过程中，会给每个存储属性设置一个初始值和执行其他必要的初始化。不同OC的初始化器，Swift的初始化器不返回一个值。初始化的主要作用是保证一个实例第一次使用之前能正确地初始化。

## 设置存储属性的初始值 (Setting Inital Values for Stored Properties)

**注意：** 当设置存储属性的默认值或者在初始化器内设置初始值时，是直接把值设置给属性，不会调用属性观察者。

## 初始化器 (Initializers)

使用`init`关键字定义一个初始化器：

```swift
init() {
    // perform some initialization here
}
```

下面是一个例子：

```swift
struct Fahrenheit {
    var temperature: Double
    init() {
        temperature = 32.0
    }
}
var f = Fahrenheit()
print("The default temperature is \(f.temperature)° Fahrenheit")
// Prints "The default temperature is 32.0° Fahrenheit"
```

## 属性默认值 (Default Property Values)

如果属性的的初始值总是一样的，那么可以直接设置一个属性的默认值。上面那个例子可以改为：

```swift
struct Fahrenheit {
    var temperature = 32.0
}
```

## 自定义初始化 (Customizing Initialization)

### 初始化参数 (Initialization Parameters)

在定义初始化器时，可以提供一些初始化参数。下面这个例子提供了两个初始化器：

```swift
struct Celsius {
    var temperatureInCelsius: Double
    init(fromFahrenheit fahrenheit: Double) {
        temperatureInCelsius = (fahrenheit - 32.0) / 1.8
    }
    init(fromKelvin kelvin: Double) {
        temperatureInCelsius = kelvin - 273.15
    }
}
let boilingPointOfWater = Celsius(fromFahrenheit: 212.0)
// boilingPointOfWater.temperatureInCelsius is 100.0
let freezingPointOfWater = Celsius(fromKelvin: 273.15)
// freezingPointOfWater.temperatureInCelsius is 0.0
```

### 参数名和参数标签 (Parameter Names and Argumenet Labels)

想方法参数一样，初始化参数也可以由参数名(在初始化器内部使用)和参数标签(调用初始化器时使用)。

例如下面这个例子，提供了两个初始化器：

```swift
struct Color {
    let red, green, blue: Double
    init(red: Double, green: Double, blue: Double) {
        self.red   = red
        self.green = green
        self.blue  = blue
    }
    init(white: Double) {
        red   = white
        green = white
        blue  = white
    }
}
```

两个初始化都可以用来初始化一个实例：

```swift
let magenta = Color(red: 1.0, green: 0.0, blue: 1.0)
let halfGray = Color(white: 0.5)
```

注意：在调用上面的初始化器时不能不使用参数标签，如果把参数标签删掉，会报编译错误：

```swift
let veryGreen = Color(0.0, 1.0, 0.0)
// this reports a compile-time error - argument labels are required
```

### 没有参数标签的初始化参数 (Initializer Parameters Without Argument Labels)

如果不想使用参数标签，用`_`代替。

```swift
struct Celsius {
    var temperatureInCelsius: Double
    init(fromFahrenheit fahrenheit: Double) {
        temperatureInCelsius = (fahrenheit - 32.0) / 1.8
    }
    init(fromKelvin kelvin: Double) {
        temperatureInCelsius = kelvin - 273.15
    }
    init(_ celsius: Double) {
        temperatureInCelsius = celsius
    }
}
let bodyTemperature = Celsius(37.0)
// bodyTemperature.temperatureInCelsius is 37.0
```

### 可选属性类型 (Optional Property Types)

在自定义一个类时，某些存储属性可能没有值，也许是因为在初始化时还不能设置属性的值，或者是后续可以没有值，那么这个属性应该定义为*可选类型*。在初始化时，可选类型的属性的值是`nil`。

例如：

```swift
class SurveyQuestion {
    var text: String
    var response: String?
    init(text: String) {
        self.text = text
    }
    func ask() {
        print(text)
    }
}
let cheeseQuestion = SurveyQuestion(text: "Do you like cheese?")
cheeseQuestion.ask()
// Prints "Do you like cheese?"
cheeseQuestion.response = "Yes, I do like cheese."
```

### 在初始化期间设置常量属性的值 (Assigning Constant Properties During Initialization)

在初始化期间，可以设置常量属性的值，这个值必须是明确的不能改变的。一旦常量属性的值设置好，就不能在改变。

**注意：** 对于一个类实例，一个常量属性的值只能在定义这个属性的类的初始化器中被修改，不能被子类修改。

上面的例子可以改为：

```swift
class SurveyQuestion {
    let text: String
    var response: String?
    init(text: String) {
        self.text = text
    }
    func ask() {
        print(text)
    }
}
let beetsQuestion = SurveyQuestion(text: "How about beets?")
beetsQuestion.ask()
// Prints "How about beets?"
beetsQuestion.response = "I also like beets. (But not with cheese.)"
```

## 默认初始化器 (Default Initializers)

Swift给结构和类提供了一个默认的初始化器，这个初始化给所有属性设置了默认值。

```swift
class ShoppingListItem {
    var name: String?
    var quantity = 1
    var purchased = false
}
var item = ShoppingListItem()
```

因为`ShoppingListItem`的所有属性都默认值，并且是一个基类，所有`ShoppingListItem`自动获得一个默认的初始化器。直接使用`()`来调用默认的初始化器。

### 结构类型的逐个成员初始化器 (Memberwise Initializers for Structure Types)

如果一个结构没有自定义初始化器，那么会自动获得一个逐个成员初始化器。不同于默认初始化器，即使结构存储属性没有默认值，它都能获得一个逐个成员初始化器。

例如下面这个`Size`结构，它自动获得一个逐个成员初始化器`init(width:height:)`:

```swift
struct Size {
    var width = 0.0, height = 0.0
}
let twoByTwo = Size(width: 2.0, height: 2.0)
```

## 值类型的初始化器代理 (Initializer Delegaton for Value Types)

初始化器可以调用其他初始化器来执行初始化实例得到一部分工作，这个过程叫做*初始化器代理*，可以避免在多个初始化器中复制代码。

初始化器代理如果工作，代理的工作形式如何，对于值类型和引用类型来说是不一样的。值类型(结构和枚举)不支持继承，所以他们的初始化器代理过程相对简单，因为他们只能使用他们自己定义的初始化器。然而，对于引用类型，还可以从其他类中继承。这意味着一个类有额外的责任来保证所有继承得到的存储属性在初始化过程中都有一个合适的值。

对于值类型，可以使用`self.init`来引用其他的初始化器。`self.init`只能在初始化器内部调用。

注意：如果我们自定义了一个初始化器，那么默认的初始化器将不能再使用。

**注意：** 如果自定义的值类型想同时拥有默认初始化器、逐个成员初始化器和自己定义的初始化器，可以把自定义的初始化器定义在扩展中。

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
    init() {}
    init(origin: Point, size: Size) {
        self.origin = origin
        self.size = size
    }
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

在上面这个例子中，可以使用三种方式来实例化`Rect`。

**第一个初始化器**：功能上类似于默认的初始化器，没有方法体。

```swift
let basicRect = Rect()
// basicRect's origin is (0.0, 0.0) and its size is (0.0, 0.0)
```

**第二个初始化器**：功能上类似于逐个成员初始化器。

```swift
let originRect = Rect(origin: Point(x: 2.0, y: 2.0),
                      size: Size(width: 5.0, height: 5.0))
// originRect's origin is (2.0, 2.0) and its size is (5.0, 5.0)
```

**第三个初始化器**：引用了第二个初始化器。

```swift
let centerRect = Rect(center: Point(x: 4.0, y: 4.0),
                      size: Size(width: 3.0, height: 3.0))
// centerRect's origin is (2.5, 2.5) and its size is (3.0, 3.0)
```

## 类的继承和初始化 (Class Inheritance and Initializatoin)

所有类的存储属性(包括通过继承得到的属性)在初始化过程中，必须设置一个初始值。

Swift为class类型定义了两种初始化器，以保证所有存储属性在初始化过程中都有一个初始值。这两个初始化器是：指定初始化器(designated initializer)和便利初始化器(convenience initializer)。

### 指定初始化器和便利初始化器 (Designated Initializers and Convenience Initializers)

指定初始化器是一个类主要的初始化器。指定初始化器初始化了这个类定义的全部属性，并且调用合适的父类初始化器来继续父类链的初始化过程。

一个类拥有非常少的指定初始化器，而且通常一个类只有一个。

每个类必须至少有一个指定初始化器。在某些情况下，通过继承一个或多个指定初始化器来满足这个要求。

对于一个类来说，便利初始化器是次要的。我们可以定义一个便利初始化器，并调用同一个类定义的指定初始化器，把某些指定初始化器的参数设为默认值。

如果这个类不需要便利初始化器的话，我们可以不定义。

### 指定初始化器和便利初始化器的语法 (Syntax for Designated and Convenience Initializers)

定义指定初始化器：

```swift
init(parameters) {
    statements
}
```

定义便利初始化器：

```swift
convenience init(parameters) {
    statements
}
```

### Class类型的初始化器代理 (Initializer Delegation for Class Types)

为了简化指定初始化器和便利初始化器的关系，Swift制定了初始化器代理的规则：

- **规则1**
	一个指定初始化器必须调用离他最近的父类的指定初始化器
- **规则2**
	一个便利初始化器必须调用同一个类的其他初始化器
- **规则3**
	一个便利初始化最终必须调用一个指定初始化器

记住这些规则的简单方法：

- 指定初始化器必须*纵向*代理
- 便利初始化器必须*横向*代理

下面是规则的演示：

![two classes](http://upload-images.jianshu.io/upload_images/2057254-44fa495dfcb6fdb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

父类有一个指定初始化器和两个便利初始化器，一个便利初始化器调用另外一个初始化器，然后在调用一个指定初始化器，符合规则2和规则3。这个这个父类上面没有父类，所以不使用规则1。

子类有两个指定初始化器和一个便利初始化器。便利初始化器必须要调用其中的一个指定初始化器，因为它必须调用同一个类的另外一个初始化器，符合规则2和规则3。两个指定初始化器都调用了父类的指定初始化器，符合规则1。

下图演示了更复杂的四个类的继承关系，很好的演示了指定初始化器在层级关系中扮演着一个漏斗点的角色，简化了类之间的相互关系：

![four classes](http://upload-images.jianshu.io/upload_images/2057254-6110c6b594372706.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 两个阶段初始化 (Two-Phase Initialization)

Swift类的初始化需要两个阶段的过程。在第一阶段，给每个存储属性赋值。给所有的存储属性赋值完成后，第二阶段马上开始，这个类还有机会进一步自定义它的存储属性。

两个阶段初始化过程可以让初始化非常安全，可以防止属性在初始化之前被访问，也可以防止属性值被其他初始化器设置为不同的值。

**注意：** Swift的两个阶段初始化过程类似OC的初始化。最主要的不同是在第一阶段，OC把`0`或者`nil`赋给每一个属性。Swift的初始化过程更灵活，我们可以自定义初始值，而且可以应对那些默认值为`0`或者`nil`的类型。

Swift的编译器会执行四个非常有用的安全检查，以保证两阶段初始化过程正确的完成：

- **安全检查1**：一个指定初始化器在调用父类的初始化器之前，必须保证它的类自己定义的所有属性已经初始化。
- **安全检查2**：一个指定初始化器必须在给继承得到的属性赋值之前调用父类的初始化器。否则，这个指定初始化器给的新值将会被父类重写。
- **安全检查3**：一个便利初始化器在给任何属性赋值之前，必须调用当前类的另一个初始化器。否则，这个便利初始化器给的新值将会被这个类的指定初始化器重写。
- **安全检查4**：一个初始化器在第一阶段初始化完成之前，不能调用任何实例方法、读取任何实例属性的值或者引用`self`。

在第一阶段完成之前，类的实例不是完全有效的。只有在第一阶段完成以后，属性才可以被访问，方法才可以被调用。

下面是说明两个阶段如何工作，基于上面的四个安全检查：

**第一阶段**

- 调用类的指定初始化器或便利初始化器
- 为类的实例分配内存，但是内存还没有初始化
- 类的指定初始化器确认所有存储属性有值，这些存储属性的内存现在初始化完成
- 类的初始化器交给父类的初始化器去执行相同的任务，父类的初始化器给父类定义的属性赋值
- 继续沿着继承链往上，直到到达链的最顶部
- 当到达链的最顶部时，在继承链里最终的类已经确保所有的存储属性有值，那么这个实例的内存才被认为初始化完成，第一段完成。

**第二阶段**

- 沿着继承链从顶部往下，在这个链的指定初始化器可以选择进一步自顶一个这个实例。这时初始化器可以访问`self`、调用实例方法等等
- 最后，在这个继承链的其他便利初始化器可以进一步自定义实例，使用`self`。

### 初始化器的继承和重写 (Initializer Inheritance and Overriding)

不同于OC的子类，Swift的子类默认是不继承父类的初始化器的。Swift的方法是阻止父类的初始化器被子类继承并用来创建没有完全和正确地初始化的子类的实例。

**注意：** 在特定情况下，父类的初始化器是可以被继承的，但是只有在安全和合适的情况下。

如果想让自定义的子类显示父类的一个或多个初始化器，我们可以在子类提供父类初始化器的自定义实现。

当我们在写匹配父类指定初始化器的子类初始化器时，可以很有效的重写父类的指定初始化器。所有必须使用`override`关键字。

**注意：** 当重写父类的指定初始化器时，必须使用`override`关键字，即使父类的指定初始化器在子类的实现是便利初始化器。

相反地，如果在写匹配父类便利初始化器的子类初始化器时，父类的便利初始化器从来不是直接被子类调用。严格来说，子类并不是重写父类的初始化器。所有，在写匹配父类便利初始化器的子类初始化器时不用写`override`关键字。

下面是一个例子：

```swift
class Vehicle {
	var numberOfWheels = 0
	var description: String {
		return "\(numberOfWheels) wheel(s)"
	}
}

let vehicle = Vehicle()
print("Vehicle: \(vehicle.description)")
// Vehicle: 0 wheel(s)
```

新建一个子类`Bicycle`继承`Vehicle`:

```swift
class Bicycle: Vehicle {
	override init() {
		super.init()
		numberOfWheels = 2
	}
}
```

`Bicycle`子类定义了一个指定初始化器，`init()`。这个初始化器正好匹配父类的指定初始化器，所以之类的初始化器需要使用`override`关键字。

`Bicycle`的`init()`首先使用`super.init()`调用了父类的默认初始化器，这保证了通过继承得到的`numberOfWheels`属性首先被父类初始化，然后子类有机会再修改它的值。在调用了`super.init()`之后，`numberOfWheels`的值被替换为`2`。

```swift
let bicycle = Bicycle()
print("Bicycle: \(bicycle.description)")
// Bicycle: 2 wheel(s)
```

**注意：** 子类在初始化时可以修改通过继承得到的变量属性，但是不能修改常量属性。

### 自动继承初始化器 (Automatic Initializer Inheritance)

默认情况下，子类是不继承父类的初始化器的。然而，在一些特定条件下，子类可以自动继承父类的初始化器。

假设子类定义的所有存储属性都有默认值，以下两条规则适用：

- **规则1：**如果子类没有定义任何指定初始化器，那么子类继承所有父类的指定初始化器
- **规则2：**如果子类实现了父类的所有指定初始化器（要么按照规则1继承他们，或者提供了自定义实现），那么子类自动继承父类的所有便利初始化器。

即使子类有便利初始化器，这些规则仍然适用。

**注意：** 子类可以实现父类的指定初始化器并作为便利初始化器的一部分。

### 指定初始化器和便利初始化器实践 (Designated and Convenience Initializers in Action)

下面的例子演示指定初始化器、便利初始化器和自动继承初始化器。这个例子定义了一个三个类的等级关系：`Food`、`RecipeIngredient`和`ShoppingListItem`，并演示他们的初始化器如何交互。

```swift
class Food {
	var name: String
	init(name: String) {
		self.name = name
	}
	convenience init() {
		self.init(name: "[Unnamed]")
	}
}
```

下图演示了`Food`的初始化器链：

![Food](http://upload-images.jianshu.io/upload_images/2057254-7846b78ef9fe1247.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

类是没有默认的逐一成员初始化器的，所以`Food`提供了一个指定初始化器，并有一个参数`name`。

```swift
let nameMeat = Food(name: Bacon)
// namedMeat's name is "Bacon"
```

`Food`类没有父类，所以`init(name: String)`初始化器无需调用`super.init()`。

`Food`还提供了一个便利初始化器，`init()`，没有参数。

```swift
let mysteryMeat = Food()
// mysteryMeat's name is "[Unnamed]"
```

第二个类是`Food`的子类`RecipeIngredient`：

```swift
class RecipeIngredient: Food {
	var quantity: Int
	init(name: String, quantity: Int) {
		self.quantity = quantity
		super.init(name: name)
	}
	
	override convenience init(name: String) {
		self.init(name: name, quantity: 1)
	}
}
```

下图演示了`RecipeIngredient`的初始化器链：

![RecipeIngredient](http://upload-images.jianshu.io/upload_images/2057254-2924dfc939e56ab3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


`RecipeIngredient`定义了一个指定初始化器`init(name: String, quantity: Int)`。首先给`quantity`属性赋值，然后调用父类的初始化器`init(name: String)`。这个过程符合两阶段初始化的安全检查1(在调用父类的初始化器之前，子类必须给自己定义的所有属性赋值)。

`RecipeIngredient`还定义了一个便利初始化器`init(name: String)`，这个初始化器与父类的指定初始化器相同，所以必须使用`override`关键字。

尽管`RecipeIngredient`把`init(name: String)`作为便利初始化器，但`RecipeIngredient`也算是重写了父类的所有指定初始化器。所以，`RecipeIngredient`自动继承了父类的便利初始化器。（符合自动继承初始化器规则2：如果子类实现了父类的所有指定初始化器，那么子类自动继承父类的所有便利初始化器。）

在这个例子中，`RecipeIngredient`的父类是`Food`，父类有一个便利初始化器`init()`，这个初始化器被`RecipeIngredient`继承。继承得到的`init()`的版本就和`Food`的版本一样，除了继承得到的`init()`是调用`RecipeIngredient`的`init(name: String)`。

三个初始化器都可以用来创建`RecipeIngredient`实例：

```swift
let oneMysteryItem = RecipeIngredient()
let oneBacon = RecipeIngredient(name: "Bacon")
let sixEggs = RecipeIngredient(name: "Eggs", quantity: 6)
```

第三个类是`RecipeIngredient`的子类`ShoppingListItem`。

```swift
class ShoppingListItem: RecipeIngredient {
	var purchased = false
    var description: String {
        var output = "\(quantity) x \(name)"
        output += purchased ? " ✔" : " ✘"
        return output
    }
}
```

因为`ShoppingListItem`给自己定义的每个属性都提供了默认值，并且没有定义任何初始化器，所以`ShoppingListItem`自动继承了所有指定初始化器和便利初始化器。

下图演示了`ShoppingListItem`的初始化器链：

![ShoppingListItem](http://upload-images.jianshu.io/upload_images/2057254-3940c2f3fa681899.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

三个初始化器都可以用来创建`ShoppingListItem`实例：

```swift
var breakfastList = [
    ShoppingListItem(),
    ShoppingListItem(name: "Bacon"),
    ShoppingListItem(name: "Eggs", quantity: 6),
]
breakfastList[0].name = "Orange juice"
breakfastList[0].purchased = true
for item in breakfastList {
    print(item.description)
}
// 1 x Orange juice ✔
// 1 x Bacon ✘
// 6 x Eggs ✘
```

## 可能错误的初始化器 (Failable Initializers)

在定义类、结构或者枚举的初始化过程时报错，有时候是非常有用的。错误的原因有可能是：初始化参数无效、缺少需要的外部资源或者其他条件。

为了应对那些造成错误的初始化条件，我们可以定义一个或多个可能错误的初始化器作为类、结构或者枚举一部分。在`init`后面加上问号来定义可能错误的初始化器。

**注意：** 不能定义参数类型和参数名相同的可能错误的初始化器和正常的初始化器。

一个可能错误的初始化器创建当前类型的可选值，在可能触发初始化失败的位置使用`return nil`。

**注意：** 严格来说，初始化器是不返回任何值的。然而，初始化器的是保证`self`完全并正确地初始化。虽然在可能初始化失败的地方要写`return nil`，但是能正常初始化成功的地方不用写`return`。

例如下面这个例子，如果species是空的，那么Animal实例化失败：

```swift
struct Animal {
    let species: String
    init?(species: String) {
        if species.isEmpty { return nil }
        self.species = species
    }
}

// species不为空
let someCreature = Animal(species: "Giraffe")
// someCreature is of type Animal?, not Animal
 
if let giraffe = someCreature {
    print("An animal was initialized with a species of \(giraffe.species)")
}
// Prints "An animal was initialized with a species of Giraffe"

// species为空
let anonymousCreature = Animal(species: "")
// anonymousCreature is of type Animal?, not Animal
 
if anonymousCreature == nil {
    print("The anonymous creature could not be initialized")
}
// Prints "The anonymous creature could not be initialized"
```

**注意：** 检查一个字符串是否为空与检查一个字符串是否为`nil`是不一样的。

### 枚举的可能错误的初始化器 (Failable Initializers for Enumerations)

我们可以使用带有一个或多个参数的可能错误的初始化器来选择一个合适的枚举情况。如果这个参数不匹配枚举的其中一个情况，那么就初始化失败。

```swift
enum TemperatureUnit {
    case kelvin, celsius, fahrenheit
    init?(symbol: Character) {
        switch symbol {
        case "K":
            self = .kelvin
        case "C":
            self = .celsius
        case "F":
            self = .fahrenheit
        default:
            return nil
        }
    }
}

// 匹配到一个case
let fahrenheitUnit = TemperatureUnit(symbol: "F")
if fahrenheitUnit != nil {
    print("This is a defined temperature unit, so initialization succeeded.")
}
// Prints "This is a defined temperature unit, so initialization succeeded."
 
 // 没有匹配到一个case
let unknownUnit = TemperatureUnit(symbol: "X")
if unknownUnit == nil {
    print("This is not a defined temperature unit, so initialization failed.")
}
// Prints "This is not a defined temperature unit, so initialization failed."
```

### 有原始值枚举的可能错误的初始化器 (Failable Initializers for Enumerations with Raw Values)

有原始值的枚举会自动获得一个可能错误的初始化器`init?(rawValue:)`，带有一个参数`rawValue`，类型与枚举的原始值类型相同。

重写上面的例子：

```swift
enum TemperatureUnit: Character {
	case kelvin = "K", celsius = "C", fahrenheit = "F"
}

let fahrenheitUnit = TemperatureUnit(rawValue: "F")
if fahrenheitUnit != nil {
    print("This is a defined temperature unit, so initialization succeeded.")
}
// Prints "This is a defined temperature unit, so initialization succeeded."
 
let unknownUnit = TemperatureUnit(rawValue: "X")
if unknownUnit == nil {
    print("This is not a defined temperature unit, so initialization failed.")
}
// Prints "This is not a defined temperature unit, so initialization failed."
```

### 初始化失败的传播 (Propagation of Initialization Failure)

类、结构和枚举的可能错误的初始化器可以传递到另一个可能错误的初始化器。类似地，一个类的可能错误的初始化器可以传递到父类的可能错误的初始化器。

如果你调用了造成失败的初始化器，那么整个初始化过程将会失败，更进一步的初始化过程代码将不能执行。

下面是一个例子：

```swift
class Product {
	let name: String
	init?(name: Strig) {
		if name.isEmpty { return nil }
		self.name = name
	}
}

class CartItem: Product {
	let quantity: Int
	init?(name: String, quantity: Int) {
		if quantity < 1 { return nil }
		self.quantity = quantity
		super.init(name: name)
	}
}
```

创建一个实例，名字不为空且数量大于等于1，初始化成功：

```swift
if let twoSocks = CartItem(name: "sock", quantity: 2) {
    print("Item: \(twoSocks.name), quantity: \(twoSocks.quantity)")
}
// Prints "Item: sock, quantity: 2"
```

创建一个实例，名字不为空，数量为0，初始化失败：

```swift
if let zeroShirts = CartItem(name: "shirt", quantity: 0) {
    print("Item: \(zeroShirts.name), quantity: \(zeroShirts.quantity)")
} else {
    print("Unable to initialize zero shirts")
}
// Prints "Unable to initialize zero shirts"
```

创建一个实例，名字为空，数量大于等于1，初始化失败：

```swift
if let oneUnnamed = CartItem(name: "", quantity: 1) {
    print("Item: \(oneUnnamed.name), quantity: \(oneUnnamed.quantity)")
} else {
    print("Unable to initialize one unnamed product")
}
// Prints "Unable to initialize one unnamed product"
```

### 重写可能失败的初始化器 (Overriding a Failable Initializer)

像其他的初始化器一样，我们可以重写父类的可能失败的初始化器。同样地，可以把父类的可能失败的初始化器重写为正常的初始化器。

注意：如果子类的正常初始化器调用父类的可能失败的初始化器，唯一的方法是必须把父类的可能失败的初始化器返回的结果强制解包。

```swift
class Document {
    var name: String?
    // this initializer creates a document with a nil name value
    init() {}
    // this initializer creates a document with a nonempty name value
    init?(name: String) {
        if name.isEmpty { return nil }
        self.name = name
    }
}
```

下面这个例子定义了`Document`的子类`AutomaticallyNamedDocument`，并重写了父类的两个指定初始化器。重写完成之后保证`AutomaticallyNamedDocument`实例有一个初始名字`[Untitled]`

```swift
class AutomaticallyNamedDocument: Document {
	override init() {
		super.init()
		self.name = "[Untitled]"
	}
	
	override init(name: String) {
		super.init()
		if name.isEmpty {
			self.name = "[Untitled]"
		}
		else {
			self.name = name
		}
	}
}
```

`AutomaticallyNamedDocument`用正常的初始化器`init(name:)`重写了父类的可能失败的初始化器`init?(name:)`。我们还可以调用父类的`init?(name:)`，并把结果强制解包来重写父类的`init?()`：

```swift
class UntitledDocument: Document {
    override init() {
        super.init(name: "[Untitled]")!
    }
}
```

如果`name`为空字符串，那么会解包错误。

### 可能失败的初始化器：init! (The init! Failable Initializer)

通常来说，通过在`init`后面加上`!`来创建可能失败的初始化器。

我们可以在`init?`调用`init!`，反之亦然；也可以把`init?`重写为`init!`，反之亦然。也可以在`init`调用`init!`，虽然这么做可能引发错误(如果`init!`初始化失败)。

## 必须的初始化器 (Required Initializers)

在定义初始化器时在最前面加上`required`关键字，来要求子类必须重写这个初始化器。

```swift
class SomeClass {
	required init() {
		// initializer implementation goes here
	}
}
```

子类在重写父类的`required`初始化器时，也需要加上`required`，以此来提醒子类的子类继续重写。重写父类的`required`指定初始化器不需要写`override`。

```swift
class SomeSubclass: SomeClass {
    required init() {
        // subclass implementation of the required initializer goes here
    }
}
```

**注意：** 如果能满足继承初始化器的要求，我们可以不必提供明确的required初始化器实现(例如，在子类没有提供初始化器的情况下，不需要重写)。

## 使用一个闭包或方法来设置属性的默认值 (Setting a Default Property Value with a Closure or Function)

如果存储属性的默认值需要自定义或者设置，我们可以使用闭包或者方法来提供默认值。当这个属性对应的实例初始化完成后，这个闭包或方法会被调用，然后闭包或者方法的返回值就会被赋给这个属性作为默认值。

下面是使用一个闭包或方法来设置属性的默认值的模型：

```swift
class SomeClass {
	let someProperty: SomeType = {
		// create a default value for someProperty inside this closure
		// someValue must be of the same type as SomeType
		return someValue
	}()
}
```

注意不要忘记了后面的`()`，这是用来提醒Swift马上执行闭包，如果漏掉了`()`，那么就是把闭包赋给这个属性。

**注意：** 如果使用闭包来初始化一个属性，一定记得在闭包执行的时候，这个实例并没有初始化完成。也就意味着不能在闭包内部访问实例的其他属性，即使那些属性有默认值。也不能使用`self`属性，或者调用其他实例方法。

下面这个例子定义了`Chessboard`结构，制作了一个象棋游戏的棋盘，大小为 8 x 8黑白相间的正方形。

![Chessboard](http://upload-images.jianshu.io/upload_images/2057254-36524202216dfe74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Chessboard`只有一个属性`boardColors`，是一个有64个布尔值的数组，`true`代表黑色的正方形，`false`代表白色正方形。第一个元素代表左上角的正方形，第二个元素代表右下角的正方形。

```swift
struct Chessboard {
	let boardColors: [Bool] = {
		var temporaryBoard = [Bool]()
		var isBlack = false
		for i in 1...8 {
			for j in 1...8 {
				temporaryBoard.append(isBlack)
				isBlack = !isBlack
			}
			isBlack = !isBlack
		}
		return temporaryBoard
	}()
	func squareIsBlackAt(row: Int, column: Int) -> Bool {
		return boardColors[(row * 8) + column]
	}
}
```

当`Chessboard`实例被创建后，这个闭包就会被执行，从而`boardColors`有了默认值。

```swift
let board = Chessboard()
print(board.squareIsBlackAt(row: 0, column: 1))
// Prints "true"
print(board.squareIsBlackAt(row: 7, column: 7))
// Prints "false"
```
