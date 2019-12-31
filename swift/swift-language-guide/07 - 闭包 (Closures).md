### 闭包 (Closures)

闭包是具有一定功能的代码块。Swift的闭包类似于C和OC中的block和其他编程语言的匿名函数。

全局和嵌套方法实际上都是属于闭包的特殊情况。闭包有以下三种形式：

- 全局方法：有一个名字，不会不会捕获任何值
- 嵌套方法：有一个名字，可以从包含这个嵌套方法的方法内部捕获值
- 闭包语句：没有名字，可以从包含它的上下文捕获值

Swift的闭包语句经过了一系列的优化变得非常简单、整洁和清晰：

- 根据上下文推断参数和返回值的类型
- 一个闭包有隐藏的返回值
- 简略的参数名
- 后置闭包语法

#### 闭包表达式 (Closure Expressions)

##### 排序方法 (Sorted Methods)

Swift标准库中有一个方法`sorted(by:)`，可以用来对一个数组的值排序。当排序执行完成时，返回一个排好序的数组，并且不会修改原数组。`sorted(by:)`方法接收一个具有两个与数组元素类型相同的参数、并返回一个布尔值的闭包，闭包的返回值说明了是正序还是倒序。

例如下面这个例子，对一个`[String]`类型的数组，排序闭包需要一个`(String, String) -> Bool`的方法：

```swift
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]

func backward(_ s1: String, _ s2: String) -> Bool {
	return s1 > s2
}
var reverseNaems = names.sorted(by: backward)
// reversedNames is equal to ["Ewa", "Daniella", "Chris", "Barry", "Alex"]
```

然而，上面这种写法不够简洁，我们可以使用闭包表达式语法来写。

##### 闭包表达式语法 (Closure Expression Syntax)

闭包表达式语法的通用形式如下：

```swift
{ (parameters) -> return type in
    statements
}
```

闭包表达式语法的参数可以是in-out参数，但是不能有默认值，可以使用可变参数，多元组也可以作为参数和返回值。

`backward(_:_:)`的闭包表达式语法：

```swift
reverseNames = names.sorted(by: { (s1: String, s2: String) -> Bool in 
	return s1 > s2
})
```

##### 根据上下文推断类型 (Inferring Type From Context)

因为分类闭包当做参数被传入一个方法，Swift能推断参数的类型和返回值的类型。因为参数和返回值的类型都能被推断出来，所以参数的类型和返回值类型都可以忽略，写成：

```swift
reversNames = names.sorted(by: { s1, s2 in return s1 > s2 } )
```

##### 隐藏Return的单个表达式闭包 (Implicit Returns from Single-Expression Closures)

单个表达式闭包可以通过删除`return`关键字来隐式地返回单个表达式的结果：

```swift
reveresNames = names.sorted(by: s1, s2 in s1 > s2)
```

##### 简略参数名 (Shorthand Argument Names)

Swift可以自动提供简略参数名给单行闭包，这些简略参数名是被用来引用于闭包的参数，例如`$0`、`$1`和`$2`等等。

如果在闭包中使用这种形式，可以在定义包时把参数省略，`in`关键字也可以省略：

```swift
reverseNames = names.sorted(by: { $0 > $1 })
```

`$0`和`$1`是闭包的第一和第二个参数。

##### 运算符方法 (Operator Methods)

其实上面的闭包还可以用更简短的方式来实现。Swift的`String`类型把`>`定义为一个具有两个`String`类型并返回布尔值得方法。这刚好符合`sorted(by:)`方法需要的闭包参数。所以上面的例子可以简写成：

```swift
reverseNames = names.sorted(by: >)
```

#### 后置闭包 (Trailing Closures)

如果我们需要传入一个很长的闭包作为参数，并且这个参数是最后一个参数，后置闭包是非常有用的。当使用后置闭包语法时，不需要写参数的标签：

```swift
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // function body goes here
}
 
// 调用时没有使用后置闭包
 
someFunctionThatTakesAClosure(closure: {
    // closure's body goes here
})
 
// 调用时使用后置闭包
 
someFunctionThatTakesAClosure() {
    // trailing closure's body goes here
}
```

上面提到的把名字倒序排列的例子中，可以使用后置闭包语法写成：

```swift
reverseNames = names.sorted() { $0 > $1 }
```

如果方法的参数中只有一个方法类型的参数，在使用后置闭包语法时，可以把`()`省略：

```swift
reverseNames = names.sorted { $0 > $1 }
```

当闭包非常长而且不能用一行代码写完时，后置闭包是非常有用的。例如，Swift的`Array`类型有一个`map(_:)`方法，这个方法需要一个闭包表达式作为它唯一的参数。这个闭包会被数组中的每个元素调用一次，并返回一个与元素相关的值。`map(_:)`方法执行完后，会返回一个新的数组，数组包含着所有与各个元素对应的值，并且顺序与原素组相同。

```swift
let digitNames = [
    0: "Zero", 1: "One", 2: "Two",   3: "Three", 4: "Four",
    5: "Five", 6: "Six", 7: "Seven", 8: "Eight", 9: "Nine"
]
let numbers = [16, 58, 510]

let strings = numbers.map { (number) -> String in
	var number = number
	var output = ""
	repeat {
		output = digitName[number % 10]! + output
		number /= 10
	} while number > 0
}
// strings is inferred to be of type [String]
// its value is ["OneSix", "FiveEight", "FiveOneZero"]
```

`map(_:)`方法会让数组的每一个元素调用一次闭包。在闭包中，我们无需指定参数`number`的类型，因为`number`的类型可以从数组的元素类型中推断出来。

`number`变量使用闭包的`number`参数参数来初始化，以保证闭包后面的代码中能修改`number`的值，但是闭包参数`number`还是属于常量。

#### 捕获值 (Capturing Values)

闭包可以从包含这个闭包的方法中捕获这个方法内部定义的常量或变量，然后闭包可以修改这些常量或者变量，即使这些常量和变量不再存在。

在Swift中，最简单的能捕获值的闭包形式就是嵌套方法。一个嵌套方法可以捕获包含嵌套方法的方法参数和内部定义的常量或者变量。

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}
```

`incrementer()`方法没有参数，从包含它的方法中引用`runningTotal`和`amount`，这种引用是通过捕获对`runningTotal`和`amount`的指针实现的。通过捕获指针保证了`runningTotal`和`amount`在`makeIncrementer`执行完之后不会消失，并且在下一次调用`makeIncrementer`时还可以使用。

**注意：**因为优化，如果一个值在闭包中没有被修改，Swift会捕获和存储这个值的副本。Swift还会处理内存管理问题，当这些变量不再使用时，Swift会把他们销毁。

下面是使用`makeIncrementer`的一个例子：

```swift
let incrementByTen = makeIncrementer(forIncrement: 10)

incrementByTen()
// returns a value of 10
incrementByTen()
// returns a value of 20
incrementByTen()
// returns a value of 30
```

如果创建第二个incrementer，它会有自己的一个对`runningTotal`的引用：

```swift
let incrementBySeven = makeIncrementer(forIncrement: 7)
incrementBySeven()
// returns a value of 7
```

然后再调用之前的`incrementByTen`，`runningTotal`的值会继续往上增加，并且不会对`incrementBySeven`引用的`runningTotal`造成影响。

**注意：**如果把一个闭包赋值给一个类对实例的属性，这个闭包又通过引用这个类对象的实例或者成员来捕获值，这将会造成在闭包和类实例之间的循环引用。

#### 闭包是引用类型 (Closures Are Reference Types)

在上面的例子中，`incrementBySeve`和`incrementByTen`都是常量，但是这两个常量引用的闭包还可以让`runningTotal`继续增加。这是因为方法和闭包都是引用类型。

不管在什么时候，把方法和闭包赋值给变量和常量，实际上常量或者变量指向了方法和闭包。那么这就意味着如果把一个闭包赋值给不同的变量或者常量，这些常量和变量实际上是引用这同一个闭包：

```swift
let alsoIncrementByTen = incrementByTen
alsoIncrementByTen
// returns a value of 50
```

#### 逃逸闭包 (Escaping Closures)

当一个闭包作为参数传给一个方法时，但是在方法返回之后才调用，那么这个闭包被称为逃逸闭包。在声明方法时，在方法参数类型之前使用`@escaping`来说明这个闭包允许“逃脱”。

一个闭包能“逃脱”的一种方式，就是把这个闭包存储在方法之外定义的变量中。例如，很多方法开启一个异步操作，并用一个闭包参数作为一个completion handler。在异步操作开始之后，马上返回。但是闭包在异步操作完成之前并不会调用，闭包需要“逃脱”，在后面调用：

```swift
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
	completionHandlers.append(completionHandler)
}
```

在这个方法中，如果不加上`@escaping`，将会编译错误。

使用`@escaping`标记一个闭包，意味着在闭包中需要明确的引用`self`。

```swift
func someFunctionWithNonescapingClosure(closure: () -> Void) {
    closure()
}
 
class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}
 
let instance = SomeClass()
instance.doSomething()
print(instance.x)
// Prints "200"
 
completionHandlers.first?()
print(instance.x)
// Prints "100"
```

#### 自动闭包 (Autoclosures)

一个闭包自动创建并被包装成一个表达式，然后作为参数传入方法，这个闭包就被称为自动闭包。

一个自动闭包可以延迟执行，因为闭包中的代码在被调用之前并不会执行。延迟执行在一些有副作用或者计算昂贵的代码中非常有用，因为我们可以控制代码何时执行：

```swift
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
print(customersInLine.count)
// Prints "5"
 
let customerProvider = { customersInLine.remove(at: 0) }
print(customersInLine.count)
// Prints "5"
print("Now serving \(customerProvider())!")
// Prints "Now serving Chris!"
print(customersInLine.count)
// Prints "4"
```

即使`customersInLine`的第一个元素在闭包中被删除了，但是这个元素在闭包执行之前并不会被删除。**注意：** `customerProvider`不是`String`类型，而是`() -> String`。

把一个闭包作为参数传给方法也是一样：

```swift
// customersInLine is ["Alex", "Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: { customersInLine.remove(at: 0) } )
// Prints "Now serving Alex!"
```

使用`@autoclosure`：

```swift
// customersInLine is ["Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0))
// Prints "Now serving Ewa!"
```

**注意：**过度使用自动闭包会降低代码的可读性。

如果一个自动闭包想要“逃脱”，同时使用`@autoclosure`和`@escaping`。

```swift
// customersInLine is ["Barry", "Daniella"]
var customerProviders: [() -> String] = []
func collectCustomerProviders(_ customerProvider: @autoclosure @escaping () -> String) {
	customerProviders.append(customerProvider)
}
collectCustomerProviders(customersInLine.remove(at: 0))
collectCustomerProviders(customersInLine.remove(at: 0))

print("Collected \(customerProviders.count) closures.")
// Prints "Collected 2 closures."
for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
// Prints "Now serving Barry!"
// Prints "Now serving Daniella!"
```
