# 23 - 泛型 (Generics)

泛型代码能让我们编写灵活、可重复使用的方法和类型。可以避免编写重复代码，并以清晰和抽象的方法表达其目的。

泛型是Swift非常强大的一个特性，并且很多Swift标准库都是用泛型编写的。实际上我们已经使用过泛型，例如Swift的`Array`和`Dictionary`都是泛型集合。

## 泛型能解决的问题 (The Problem That Generics Solve)

下面这个方法只能交换两个`Int`值，使用in-out参数来交换`a`和`b`：

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

上面这个方法只能交换`Int`，如果我想交换两个`String`类型的值或者两个`Double`类型的值，我们又得写两个方法`swapTwoStrings(_:_:)`和`swapTwoDoubles(_:_:) `:

```swift
func swapTwoStrings(_ a: inout String, _ b: inout String) {
    let temporaryA = a
    a = b
    b = temporaryA
}
 
func swapTwoDoubles(_ a: inout Double, _ b: inout Double) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

使用泛型代码让我们用一个方法就能解决上面的问题。

## 泛型方法 (Generic Functions)

上面的三个方法可以用下面的额泛型方法代替：

```swift
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

这个通用的方法使用`T`作为类型名称，而不是`Int`、`String`或者`Double`。`T`没有说明必须是什么类型，但是`a`和`b`的类型必须是一样的。

方法名后面还有一个`<T>`，告诉Swift只是一个占位类型，不是实际类型。

`swapTwoValues(_:_:)`方法可以像之前的`swapTwoInts`一样调用，只要两个用于交换的值类型是一样的。

用通用的方法交换两个`String`类型的值或者两个`Double`类型的值：

```swift
var someInt = 3
var anotherInt = 107
swapTwoValues(&someInt, &anotherInt)
// someInt is now 107, and anotherInt is now 3
 
var someString = "hello"
var anotherString = "world"
swapTwoValues(&someString, &anotherString)
// someString is now "world", and anotherString is now "hello"
```

**注意：** 其实Swift的标准库中已经有一个功能与`swapTwoValues(_:_:)`一样的方法`swap`。如果我们需要交换两个值，直接使用`swap`即可，无需自己另外实现这个功能。

## 类型参数 (Type Parameters)

上面的泛型方法`swapTwoValues(_:_:)`，占位类型`T`其实是一个类型参数。类型参数指定并命名占位类型，紧跟在方法名后面，用`<T>`表示。

只要定义好了类型参数，我们就可以用来定义方法的参数类型(例如`swapTwoValues(_:_:)`的参数`a`和`b`的类型)，或者作为方法的返回类型。

我们可以定义多个类型参数，写法是：`<T1, T2, ...>`。

## 类型参数命名 (Type Parameters)

在很多情况下，类型参数有描述性的名称，例如`Dictionary<Key, Value>`中的`Key`和`Value`，`Array<Element>`中的`Element`，这些名字都能告诉读者类型参数和泛型的关系。但是，在没有任何意义的情况下，我们一般把类型参数名命名为`T`、`U`和`V`。

**注意：** 需要使用骆驼命名法给类型参数命名，例如`T`和`MyTypeParameter`，以表示他们是一个类型的占位。

## 泛型类型 (Generic Types)

除了泛型方法，Swift还可以定义泛型类型，例如`Array`和`Dictionary`。

这一部分将演示如何写一个泛型集合`Stack`。一个栈是一个有序的值的集合，类似一个数组，但是比数组有更严格的运算。	数组可以在特定的位置移除或插入元素。而栈只允许在集合的最后添加元素，也只允许从最后面移除元素。

**注意：** 栈的概念就被用于`UINavigationController`管理控制器。使用`pushViewController(_:animated:)`来添加新的控制器到栈中，用`popViewControllerAnimated(_:)`移除控制器。栈非常适合用来管理“后进先去”的集合。

下图演示了栈的push/pop：

![Stack](http://upload-images.jianshu.io/upload_images/2057254-1bc8e8c0b05c7d73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 目前栈中有三个值
2. 第四个值被push到栈顶
3. 栈中有四个值
4. 第四个值被移除(popped)
5. 移除第四个值后，栈中剩下三个值

下面是一个不通用的`IntStack`:

```swift
struct IntStack {
	var items = [Int]()
	
	mutating fuc push(_ item: Int) {
		items.append(item)
	}
	
	mutating fuc pop() -> Int{
		return items.removeLast()
	}
}
```

下面是一个通用版本的Stack：

```swift
struct Stack<Element> {
	var items = [Element]()
	
	mutating func push(_ item: Element) {
		items.append(item)
	}
	
	mutating func pop() -> Element {
		return items.removeLast()
	}
}
```

`Element`定义了一个占位名字，并且在三个地方被用到：

- 创建一个空数组`items`，其元素类型为`Element`
- 作为`push(_:)`方法的参数类型
- 作为`pop()`方法的返回值

因为它是一个通用类型，所以`Stack`可以用来创建任意有效类型的栈：

```swift
var stackOfThings = Stack<String>()
stackOfStrings.push("uno")
stackOfStrings.push("dos")
stackOfStrings.push("tres")
stackOfStrings.push("cuatro")
// the stack now contains 4 strings
```

下面是`stackOfThings`的push过程：

![push](http://upload-images.jianshu.io/upload_images/2057254-f03fcdac59cd6721.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

移除栈顶的值`"cuatro"`：

```swift
let fromTheTop = stackOfStrings.pop()
// fromTheTop is equal to "cuatro", and the stack now contains 3 strings
```

移除过程如下：

![pop](http://upload-images.jianshu.io/upload_images/2057254-b76a505841dc16df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 扩展泛型类型 (Extending a Generic Type)

在扩展泛型类型是，我们不必提供类型参数，可以直接使用已有泛型类型的类型参数。

下面是对`Stack`的扩展：

```swift
extension Stack {
	va topItem: Element? {
		return items.isEnpty ? nil : items[items.count - 1]
	}
}
```

访问`topItem`属性：

```swift
if let topItem = stackOfStrings.topItem {
    print("The top item on the stack is \(topItem).")
}
// Prints "The top item on the stack is tres."
```

泛型类型的扩展也可以指定一些限定条件，满足这些条件才可以访问扩展里面的成员，下面会讲到。

## 类型约束 (Type Constraints)

`swapTwoValues(_:_:)`和`Stack`可以用于任何类型。然而，在某些类型上添加一些类型约束是非常有用的。类型约束指定一个类型参数必须继承于特定的类或者遵循特定的协议。

例如，Swift的`Dictionary`限定了键的类型。字典的键必须是*hashable*的，也就是说，必须提供一个方法保证它自己是唯一的，然后字典才能根据这个键去查找对应的值。Swift的基本类型默认都是`hashable`的，例如`String`、`Int`、`Double`和`Bool`。

### 类型约束语法 (Type Constraint Syntax)

语法如下：

```swift
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // function body goes here
}
```

上面的语法要求`T`必须是`SomeClass`的子类，`U`必须遵循`SomeProtocol`协议。

### 类型约束实践 (Type Constraint in Action)

下面是一个不通用的`findIndex(ofString:in:)`，用于查找指定字符串在字符串数组的索引：

```swift
func findIndex(ofString valueToFind: String, in array: [String]) -> Int? {
	for (index, value) in array.enumerated() {
		if value == valueToFind {
			return index
		}
	}
	return nil
}

let strings = ["cat", "dog", "llama", "parakeet", "terrapin"]
if let foundIndex = findIndex(ofString: "llama", in: strings) {
    print("The index of llama is \(foundIndex)")
}
// Prints "The index of llama is 2"
```

我们可以把上面的方法改为通用形式：

```swift
func findIndex<T>(of valueToFind: T in array: [T]) -> Int? {
	for (index, value) in array.enumerated() {
		if value == valueToFind {
			return value
		}
	}
	return nil
}
```

其实这个方法不能编译通过，因为`value == valueToFind`。不是所有Swift类型的都可使用`==`进行比较。

Swift定义了`Equatable`协议，要求所有遵循这个协议的类型必须实现`==`和`!=`运算符。Swift的标准类型都遵循了`Equatable`协议。

所以上面的通用方法改写为：

```swift
func findIndex<T: Equatable>(of valueToFind: T, in array: [T]) -> Int? {
	for (index, value) in array.enumerated() {
		if value == valueToFind {
			return index
		}
	}
	return nil
}
```

`T: Equatable`类型参数意味着任意类型`T`必须遵循`Equatalbe`协议。

例如：

```swift
let doubleIndex = findIndex(of: 9.3, in: [3.14159, 0.1, 0.25])
// doubleIndex is an optional Int with no value, because 9.3 is not in the array
let stringIndex = findIndex(of: "Andrea", in: ["Mike", "Malcolm", "Andrea"])
// stringIndex is an optional Int containing a value of 2
```

## 关联类型 (Associated Types)

当定义协议时，定义一个或多个关联类型有时候是非常有用的。关联类型提供了一个类型名字，然后这个类型可以在协议中使用。使用`associatedtype`关键字来定义关联类型。

### 关联类型实践 (Associated Types in Action)

下面是一个例子：

```swift
protocol Container {
	associatedtype Item
	mutating func append(_ item: Item)
	var count: Int { get }
	subscript(i: Int) -> Item { get }
}
```

这个容器协议中没有具体说明能存储那个类型的值。遵循这个协议的类型必须写清楚能存储的值类型。

之前那个`IntStack`类型遵循`Container`协议：

```swift
struct Instack: Container {
	// original Instack implementation
	var items = [Int]()
	mutating func push(_ item: Int) {
		items.append(item)
	}
	mutating func pop() -> Int {
		return items.removeLast()
	}
	
	// conformance to the Container protocol
	typealias Item = Int
	mutating func append(_ item: Int) {
		self.push(item)
	}
	var count: Int {
		return items.count
	}
	subscript(i: Int) -> Int {
		return items[i]
	}
}
```

`IntStack`指定了`Item`是一个`Int`类型。`typealias Item = Int`把抽象的`Item`转变为具体的`Int`。

因为Swift有类型推断功能，所以我们可以不用写`typealias Item = Int`，因为`IntStack`实现了`Container`协议的所有要求，Swift能推断出需要用的`Item`的具体类型。

之前那个`Stack`类型遵循`Container`协议：

```swift
struct Stack<Element>: Container {
    // original Stack<Element> implementation
    var items = [Element]()
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
    // conformance to the Container protocol
    mutating func append(_ item: Element) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Element {
        return items[i]
    }
}
```

### 通过扩展类型来指定关联类型 (Extending an Existing Type to Specifying an Associated Type)

我们可以使用扩展来遵循某一个协议，当然这个协议也可以是由关联类型的协议。

Swift的`Array`已经提供了`append(_:)`方法、`count`属性和下标，那么就满足了`Container`协议的要求。所以我们可以是下面这种形式来声明`Array`遵循`Container`协议：

```swift
extension Array: Container {}
```

我们就可以把`Array`当做是一个`Container`。

## 泛型的Where语句 (Generic Where Clauses)

泛型的`where`语句可以让我们要求关联类型遵循一个特定的协议，或者关联类型和类型参数必须相同。

例如下面这个例子：

```swift
func allItemsMatch<C1: Container, C2: Container> (_ someContainer: C1, _ anotherContainer: C2) -> Bool where C1.Item == C2.Item, C1.Item: Equatalbe {
	// Check that both containers contain the same number of items.
	if someContainer.count != anotherContainer.count { return false }
	
	// Check each pair of items to see if they are equivalent.
	for i in 0..<someContainer.count {
		if someContainer[i] != anotherContainer[i] { return false }
	}
	
	// All items match, so return true.
	return true
}
```

`allItemsMatch(_:_:)`方法的实际使用：

```swift
var stackOfStrings = Stack<String>()
stackOfStrings.push("uno")
stackOfStrings.push("dos")
stackOfStrings.push("tres")
 
var arrayOfStrings = ["uno", "dos", "tres"]
 
if allItemsMatch(stackOfStrings, arrayOfStrings) {
    print("All items match.")
} else {
    print("Not all items match.")
}
// Prints "All items match."
```

虽然`Array`和`Stack`是不同的类型，但是他们都遵循`Container`协议，并且都包含相同类型的值。

## 有泛型Where语句的扩展 (Extensions with a Generic Where Clause)

例如下面这个例子：

```swift
extension Stack where Element: Equatable {
	func isTop(_ item: Element) -> Bool {
		guard let topItem = items.last else {
			return false
		}
		return topItem == item
	}
}
```

如果没有泛型`where`语句，将会出现一个问题：`isTop(_:)`方法使用了`==`运算符，但是`Stack`没有要求它的元素是可以比较的，所以使用`==`将会编译错误。

下面是`isTop(_:)`方法的使用：

```swift
if stackOfStrings.isTop("tres") {
    print("Top element is tres.")
} else {
    print("Top element is something else.")
}
// Prints "Top element is tres."
```

下面是使用`where`扩展`Container`:

```swift
extension Container where Item: Equatalbe {
	func startsWith(_ item: Item) -> Bool {
		return count >= 1 && self[0] == item
	}
}
```

`startsWith(_:)`适用于遵循了`Container`协议的类型：

```swift
if [9, 9, 9].startsWith(42) {
    print("Starts with 42.")
} else {
    print("Starts with something else.")
}
// Prints "Starts with something else."
```

上面对`Container`的扩展要求`Item`遵循`Equatable`协议，我们还可以要求`Item`是一个具体的类型：

```swift
extension Container where Item == Double {
	func average() -> Double {
		var sum = 0.0
		for index in 0..<count {
			sum += self[index]
		}
		return sum / Double(count)
	}
}
print([1260.0, 1200.0, 98.6, 37.0].average())
// Prints "648.9"
```

我们还可以在`where`语句中可以添加多个约束条件，条件之间用逗号隔开即可。
