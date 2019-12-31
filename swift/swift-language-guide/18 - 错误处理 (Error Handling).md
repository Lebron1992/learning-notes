# 18 - 错误处理 (Error Handling)

错误处理是在程序中响应错误条件和恢复错误条件的过程。

## 表示和抛出错误 (Representing and Throwing Errors)

在Swift中，错误是用遵循了`Error`协议的类型的值来表示。枚举非常适合用来封装相关的错误。

```swift
enum VendingMachineError: Error {
    case invalidSelection
    case insufficientFunds(coinsNeeded: Int)
    case outOfStock
}
```

是用`throw`来抛出错误：

```swift
throw VendingMachineError.insufficientFunds(coinsNeeded: 5)
```

## 处理错误 (Handling Errors)

当错误抛出后，一些相关的代码必须处理错误，例如改正错误、尝试另外一种办法或者告知用户有错误。

在Swift中，有四种方法来处理错误。把错误传递给调用这个方法的代码；使用`do-catch`语句处理错误；把错误处理为一个可选类型的值；或者断言这个错误不会发生。下面会演示这四个方法。

当一个方法抛出了错误，它会改变程序的流程，所以及时发现错误的位置非常重要。为了发现错误的位置，在调用方法或初始化器的代码前使用`try`、`try?`或者`try!`。

### 使用抛出方法来传递错误 (Propagating Errors Using Throwing Functions)

为了表示一个方法或初始化器可以抛出异常，在方法参数后面加上`throw`关键字：

```swift
func canThrowErrors() throws -> String
func cannotThrowErrors() -> String
```

**注意：** 只有抛出方法才能传递错误，不能抛出错误的方法只能在方法内处理错误。

下面是一个例子：

```swift
struct Item {
    var price: Int
    var count: Int
}
 
class VendingMachine {
    var inventory = [
        "Candy Bar": Item(price: 12, count: 7),
        "Chips": Item(price: 10, count: 4),
        "Pretzels": Item(price: 7, count: 11)
    ]
    var coinsDeposited = 0
    
    func vend(itemNamed name: String) throws {
        guard let item = inventory[name] else {
            throw VendingMachineError.invalidSelection
        }
        
        guard item.count > 0 else {
            throw VendingMachineError.outOfStock
        }
        
        guard item.price <= coinsDeposited else {
            throw VendingMachineError.insufficientFunds(coinsNeeded: item.price - coinsDeposited)
        }
        
        coinsDeposited -= item.price
        
        var newItem = item
        newItem.count -= 1
        inventory[name] = newItem
        
        print("Dispensing \(name)")
    }
}
```

在`vend(itemNamed:)`方法的实现中，使用`guard`语句来抛出错误。

因为`vend(itemNamed:)`把错误传出去了，所以调用这个方法的代码必须处理错误：使用`do-catch`语句、try?、try!或者继续把错误传出去。下面演示的是用继续把错误传出去的方法来处理：

```swift
let favoriteSnacks = [
    "Alice": "Chips",
    "Bob": "Licorice",
    "Eve": "Pretzels",
]
func buyFavoriteSnack(person: String, vendingMachine: VendingMachine) throws {
    let snackName = favoriteSnacks[person] ?? "Candy Bar"
    try vendingMachine.vend(itemNamed: snackName)
}
```

因为`vend(itemNamed:)`能抛出错误，所以能再前面加上`try`。

初始化器也能抛出错误：

```swift
struct PurchasedSnack {
    let name: String
    init(name: String, vendingMachine: VendingMachine) throws {
        try vendingMachine.vend(itemNamed: name)
        self.name = name
    }
}
```

### 使用Do-Catch来处理错误 (Handling Errors Using Do-Catch)

`do-catch`语句的通用形式：

```swift
do {
    try expression
    statements
} catch pattern 1 {
    statements
} catch pattern 2 where condition {
    statements
}
```

在`catch`后面写一个样式来提示什么样的错误能被这个`catch`语句处理。如果`catch`语句没有样式，那么这个语句会匹配任何错误，并且绑定这个错误作为一个本地常量`error`。

`catch`语句不必处理每一个可能的错误。如果没有`catch`语句处理这个错误，那么这个错误将会传递给周围——要么用`do-catch`语句处理，要么通过内部的抛出方法。例如下面这个例子除了`VendingMachineError`的所有枚举值，但其他错误只能由周围的代码去处理：

```swift
var vendingMachine = VendingMachine()
vendingMachine.coinsDeposited = 8
do {
    try buyFavoriteSnack(person: "Alice", vendingMachine: vendingMachine)
} catch VendingMachineError.invalidSelection {
    print("Invalid Selection.")
} catch VendingMachineError.outOfStock {
    print("Out of Stock.")
} catch VendingMachineError.insufficientFunds(let coinsNeeded) {
    print("Insufficient funds. Please insert an additional \(coinsNeeded) coins.")
}
// Prints "Insufficient funds. Please insert an additional 2 coins."
```

### 把错误转换为可选值 (Converting Error to Optional Values)

使用`try?`把错误转为可选值。在使用`try?`的语句中，如果有错误抛出，那么这个语句的值为`nil`。

例如下面这个例子，x和y的值相同：

```swift
func someThrowingFunction() throws -> Int {
    // ...
}
 
let x = try? someThrowingFunction()
 
let y: Int?
do {
    y = try someThrowingFunction()
} catch {
    y = nil
}
```

当用同一个方法来处理所有错误时，使用`try?`能是代码更简洁：

```swift
func fetchData() -> Data? {
    if let data = try? fetchDataFromDisk() { return data }
    if let data = try? fetchDataFromServer() { return data }
    return nil
}
```

### 禁用错误传递 (Disabling Error Propagation)

有时候我们知道一个能抛出错误的方法在运行过程中时间上不会抛出错误。在这种情况下，我们可以在语句前使用`try!`来禁用错误传递，并且可以封装在断言内，如果真的有错误抛出，那么程序报运行时错误。

例如：

```swift
let photo = try! loadImage(atPath: "./Resources/John Appleseed.jpg")
```

## 指定清理操作 (Specifying Cleanup Actions)

使用`defer`语句在代码执行离开当前代码块之前执行一些语句。不管代码执行如何离开当前代码块，不管是因为报错、`return`或者`break`，`defer`中的语句都能让我们做一些必要的清理。例如，可以使用`defer`语句来保证文件描述符被关闭和手动分配的内存被释放。

`defer`语句直到当前代码块退出时才会执行。不能包括转移控制语句，例如`break`或者`return`，或者抛出一个错误。延迟操作将按照他们定义的顺序的反序执行，也就是说，第一个`defer`语句在第二个`defer`语句之后执行。

```swift
func processFile(filename: String) throws {
    if exists(filename) {
        let file = open(filename)
        defer {
            close(file)
        }
        while let line = try file.readline() {
            // Work with the file.
        }
        // close(file) is called here, at the end of the scope.
    }
}
```

上面这个例子使用`defer`语句来保证`open(_:)`方法有对应的`close(_:)`方法。

**注意：** 即使没有涉及错误处理代码也可以使用`defer`语句。
