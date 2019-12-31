### 自动引用计数 (Automatic Reference Counting)

Swift使用*Automatic Reference Counting*(ARC)来管理应用内存。在大多数情况下，我们不必关心内存的管理。然而，在有些情况下ARC需要更多的信息来管理内存。这个章节就来讨论下这些情况。

**注意：**引用计数只适用于类的实例。结构和枚举是值类型，不是引用类型。

#### ARC如何工作 (How ARC Works)

每当我们创建一个类的实例，ARC会分配内存来存储这个实例的相关信息。当这个实例不再需要的时候，ARC会释放存储这个是实例相关信息的内存。

然而，如果ARC释放了那些还在使用的实例，那么我们就不能再访问实例的属性或者调用实例的方法。如果还尝试访问这个实例，应用将会崩溃。

为了保证一个实例还需要使用时不会被释放，ARC跟踪有多少个属性、常量和变量正在引用这个实例。只要至少还有一个引用，ARC不会释放这个实例。

当我们把这个实例赋给一个属性、常量或者变量，这个属性、常量或者变量就会有一个强引用引用着这个实例。之所以被称为“强”引用，是因为这个属性、常量或者变量牢牢地抓住的这个实例，只要还有强引用存在，这个实例就不允许被释放。

#### ARC实践 (ARC in Action)

首先有一个`Person`类：

```swift
class Person {
	let name: String
	init(name: String) {
		self.name = name
		print("\(name) is being initialized")
	}
	deinit {
		print("\(name) is being deinitialized")
	}
}
```

下面是三个`Person?`类型的变量，用于对一个`Person`实例进行多个引用。

```swift
var reference1: Person?
var reference2: Person?
var reference3: Person?
```

创建一个`Person`实例并赋给reference1:

```swift
reference1 = Person(name: "John Appleseed")
// Prints "John Appleseed is being initialized"
```

因为`Person`实例赋给了reference1，所以现在有一个强引用引用着`Person`实例，ARC不会被释放。

如果把`reference1`赋给另外两个变量：

```swift
reference2 = reference1
reference3 = reference1
```

那么现在有三个强引用引用着`Persjon`实例。

把`reference1`和`reference2`设置为`nil`，其中的两个强引用被打断，剩下一个强引用：

```swift
reference1 = nil
reference2 = nil
```

把`reference3`设置为`nil`，最后一个强引用被打断，没有其他属性在引用着`Person`实例，`deinit`方法执行，`Person`实例被释放：

```swift
reference3 = nil
// Prints "John Appleseed is being deinitialized"
```

#### 类实例之间的强引用循环 (Strong Reference Cycles Between Class Instances)

上面的例子跟踪了多个强引用引用着`Person`实例，当`Person`实例不在引用时，被释放。

然而，有可能写了一些代码使得一个类实例的强引用数量不能变为0。例如，如果两个实例之间各有一个强引用引用着对方，那么这两个实例的强引用数量都不能变为0，这就叫*强引用循环*。

要解决强引用循环，我们需要把一些强引用设置为弱引用或者无主引用。在解决强引用循环之前，我们先看看强引用循环是如何造成的。

新建`Person`和`Apartment`类：

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}
 
class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}
```

创建`Person`和`Apartment`类的实例：

```swift
var john: Person? = Person(name: "John Appleseed")
var unit4A: Apartment? = Apartment(unit: "4A")
```

下图是目前强引用情况：

![强引用](http://upload-images.jianshu.io/upload_images/2057254-607f5f61fd35fa44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们把两个实例联系起来之后，人有了公寓，公寓有了租客。

```swift
john!.apartment = unit4A
unit4A!.tenant = john
```

下图是目前强引用情况：

![强引用](http://upload-images.jianshu.io/upload_images/2057254-d90d8709647aaaa0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不幸的是，两个实例之间互相有一个强引用引用着。`Person`实例有一个强引用引用着`Apartment`实例，并且`Apartment`实例引用着`Person`实例。所以，当我们把`john`和`unit4A`变量设置为`nil`之后，两个实例的强引用数都不为0，所以不会被ARC释放：

```swift
john = nil
unit4A = nil
```

下图是目前强引用情况：

![强引用](http://upload-images.jianshu.io/upload_images/2057254-8d266b7ac8d37cb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

两个实例之间互相有一个强引用引用着。

#### 解决两个实例之间的强引用循环 (Resolving Strong Reference Cycles Between Class Instance)

Swift提供了两种方式来解决两个实例之间的强引用循环：弱引用(weak reference)和无主引用(unowned reference)。

当其他实例的生命周期比较短时（也就是说其他实例先被释放），使用弱引用；当其他实例有同样或者更长的生命周期时，使用无主引用。

##### 弱引用 (Weak References)

一个引用不会牢牢抓住它引用的实例，这就叫做弱引用。在属性或者变量声明时，在最前面加上`weak`来提示这将会创建一个弱引用。

当弱引用引用的实例被释放之后，ARC会自动把弱引用设置为`nil`。因为弱引用要求他们的值在运行的时候能被改为`nil`，所以它们总是被声明为变量可选类型，而不是常量可选类型。

**注意：**当ARC把弱引用设置为`nil`时，属性观察者不会被调用。

把上面的例子更改如下，`Apartment`的属性`tenant`属性声明为弱引用：

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}
 
class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
	weak var tenant: Person?
	deinit { print("Apartment \(unit) is being deinitialized") }
}
```

创建`Person`和`Apartment`实例，并联系起来：

```swift
var john: Person? = Person(name: "John Appleseed")
var unit4A: Apartment? = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = john
```

下图是目前引用情况：

![引用](http://upload-images.jianshu.io/upload_images/2057254-3cc60af37ade9dc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Person`实例仍然强引用着`Apartment`实例，但是`Apartment`实例弱引用着`Apartment`实例。这意味着，当我们把`john`实例设置为`nil`之后，就没有强引用对`Person`实例进行引用：

```swift
john = nil
// Prints "John Appleseed is being deinitialized"
```

因为没有强引用对`Person`实例进行引用，所以`Person`实例被释放，`tenant`属性被设置为`nil`：

![引用](http://upload-images.jianshu.io/upload_images/2057254-a88b4ba6f40569fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Apartment`实例只被一个强引用引用着，如果把`unit4A`也设置为`nil`，`Apartment`实例就没有强引用引用着：

```swift
unit4A = nil
```

`Apartment`实例就没有强引用引用着，也会被释放：

![引用](http://upload-images.jianshu.io/upload_images/2057254-efdb6b200b87c8dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 无主引用 (Unowned References)

当其他实例的有同样或更长的生命周期时，使用无主引用。在属性或者变量声明时，在最前面加上`unowned`来提示这将会创建一个无主引用。

下面是顾客和信用卡类：

```swift
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
    deinit { print("\(name) is being deinitialized") }
}
 
class CreditCard {
    let number: UInt64
    unowned let customer: Customer
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    deinit { print("Card #\(number) is being deinitialized") }
}
```

**注意：** `CreditCard`的`number`属性被设置为`UInt64`类型，以保证`number`的取值范围足够存储16位信用卡号码。

新建一个`Customer`实例：

```swift
var john: Customer? = Customer(name: "John Appleseed")
john!.card = CreditCard(number: 1234_5678_9012_3456, customer: john!)
```

目前的引用情况如下：

![引用](http://upload-images.jianshu.io/upload_images/2057254-5aacf1ac16385d14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Customer`实例有一个强引用引用着`CreditCard`实例，`CreditCard`实例有一个弱引用引用着`Customer`实例。

当我们把`john`设置为`nil`之后，就没有强引用引用着`Customer`实例：

![引用](http://upload-images.jianshu.io/upload_images/2057254-22017316c22bde72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为没有强引用引用着`Customer`实例，所以被释放；`Customer`实例被释放之后，`CreditCard`实例也没有被强引用引用着，也被释放：

```swift
john = nil
// Prints "John Appleseed is being deinitialized"
// Prints "Card #1234567890123456 is being deinitialized"
```

**注意：**上面这个例子演示的是如何使用安全的无主引用。Swift还提供了不安全的无主引用，可以在需要禁用运行时安全检查时使用。一旦使用了不安全无主引用，我们有责任去检查代码的安全性。使用`unowned(unsafe)`来定义一个不安全的无主引用。不安全的无主引用引用的实例被释放之后，如果我们还继续访问，那我们访问的是那个实例之前在内存的存储位置，这是一个不安全的操作。

##### 无主引用和隐式解包可选类型属性 (Unowned References and Implicitly Unwrapped Optional Properties)

上面演示了两种常见的造成强引用循环的情况。

然而，还有第三种，两个属性都应该有值，并且初始化完成之后，都不应该为`nil`。在这种情况下，我们要把一个类的无主属性(unowned property)和另外一个类的隐式解包可选类型属性结合起来。一旦初始化完成，我们就可以访问这两个属性，并且避免了引用循环。

新建`Country`和`City`类：

```swift
class Country {
    let name: String
    var capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        self.capitalCity = City(name: capitalName, country: self)
    }
}
 
class City {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}
```

`Country`在初始化器中调用了`City`的初始化器，然而在`Country`实例完全初始化之前，是不能把`self`属性传给`City`的初始化器的。

为了应对这种情况，我们把`Country`的`capital`属性定义为一个隐式解包可选类型属性，这意味着`capitalCity`属性有一个默认值`nil`，可以在不不用解包的情况下直接访问。

因为`capitalCity`有默认值`nil`，只要`name`属性有值，那么`Country`实例就被认为初始化完成。所以，在`Country`的初始化器中，设置好`name`的值之后，就可以把`self`属性传递给`City`的初始化器。

我们就可以用一行代码创建`Country`和`City`实例，并且没有循环引用：

```swift
var country = Country(name: "Canada", capitalName: "Ottawa")
print("\(country.name)'s capital city is called \(country.capitalCity.name)")
// Prints "Canada's capital city is called Ottawa"
```

#### 闭包的强引用循环 (Strong Reference Cycles for Closures)

如果我们把闭包赋给类的属性，而这个闭包又引用着这个类的实例（包括引用这个类的属性和方法），这也会造成强引用循环。造成循环引用是因为闭包，因为闭包向class一样，是引用类型。

下面是演示闭包强引用循环：

```swift
class HTMLElement {
    
    let name: String
    let text: String?
    
    lazy var asHTML: () -> String = {
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }
    
    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
    
}
```

创建一个`HTMLElement`实例：

```swift
var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// Prints "<p>hello, world</p>"
```

不幸的是，`HTMLElement`实例和赋给`asHTML`属性的闭包之间有强引用循环：

![强引用](http://upload-images.jianshu.io/upload_images/2057254-968ba8e960a6dc19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`asHTML`引用着闭包，闭包又引用着`self.name`和`self.text`。

**注意：**即使闭包多次引用`self`，但是只有一个强引用引用着`HTMLElement`实例。

把`paragraph`设置为`nil`，`HTMLElement`实例和闭包不会被释放，因为强引用循环：

```swift
paragraph = nil
```

#### 解决闭包的强引用循环 (Resolving Strong Reference Cycles for Closures)

定义一个捕获列表来解决闭包和类实例之间的强引用循环问题，并把捕获列表作为闭包的一部分。

##### 定义一个捕获列表 (Defining a Capture List)

语法如下：

```swift
lazy var someClosure: (Int, String) -> String = {
    [unowned self, weak delegate = self.delegate!] (index: Int, stringToProcess: String) -> String in
    // closure body goes here
}
```

如果闭包没有明确写出参数或者返回值类型，因为他们可以从上下文推断出来，那么可以简写成：

```swift
lazy var someClosure: () -> String = {
	[unowned self, weak delegate = self.delegate!] in
	// closure body goes here
}
```

##### 弱引用和无主引用 (Weak and Unowned References)

当闭包和闭包捕获的实例总是互相引用并同时被释放时，把捕获定义为无主引用。

当捕获的引用在未来某些时候可能变为`nil`时，把捕获定义为弱引用。弱引用永远都是一个可选类型，并且当他们引用的实例被释放时会自动变为`nil`。

**注意：**如果捕获的引用从不变成`nil`，一定要把捕获定义为无主引用，而不是弱引用。

无主引用可以用于解决上面`HTMLElement`这个例子的循环引用问题：

```swift
class HTMLElement {
    
    let name: String
    let text: String?
    
    lazy var asHTML: () -> String = {
        [unowned self] in
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }
    
    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
    
}
```

`[unowned self]`是捕获列表，意思是把捕获的`self`作为无主引用。

创建一个`HTMLElement`实例：

```swift
var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
print(paragraph!.asHTML())
// Prints "<p>hello, world</p>"
```

闭包和实例之间的引用如下：

![引用](http://upload-images.jianshu.io/upload_images/2057254-b8bc3e3e5be725d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时，闭包对`self`的引用是无主引用。如果把`paragraph`设置为`nil`，`HTMLElement`实例将会被释放。

```swift
paragraph = nil
// Prints "p is being deinitialized"
```
