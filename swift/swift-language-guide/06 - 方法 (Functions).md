# 方法 (Functions)

## 声明和调用方法 (Defining and Calling Functions)

```swift
func greet(person: String) -> String {
    let greeting = "Hello, " + person + "!"
    return greeting
}
```

使用`func`来声明方法，`greet`是方法名，`person`是参数名（也是标签名），`String`是参数类型，`->`来指定返回值类型。

## 方法的参数和返回值 (Function Parameters and Return Values)

### 无参数的方法 (Functions Without Prameters)

```swift
func sayHelloWorld() -> String {
    return "hello, world"
}
print(sayHelloWorld())
// Prints "hello, world"
```

### 有多个参数的方法 (Function With Multiple Parameters)

```swift
func greet(person: String, alreadyGreeted: Bool) -> String {
    if alreadyGreeted {
        return greetAgain(person: person)
    } else {
        return greet(person: person)
    }
}
print(greet(person: "Tim", alreadyGreeted: true))
// Prints "Hello again, Tim!"
```

### 无返回值的方法 (Functions Without Return Values)

```swift
func greet(person: String) {
    print("Hello, \(person)!")
}
greet(person: "Dave")
// Prints "Hello, Dave!"
```

因为参数没有返回值，所以方法的定义不包含`->`和返回类型。

**注意：** 严格地说，这个方法还是有一个返回值，即使没有定义返回类型。没有定义返回值的方法返回一个`Void`类型的值，这个返回值是一个空多元组`()`。

方法被调用之后，返回值可以忽略：

```swift
func printAndCount(string: String) -> Int {
    print(string)
    return string.characters.count
}
func printWithoutCounting(string: String) {
    let _ = printAndCount(string: string)
}
printAndCount(string: "hello, world")
// prints "hello, world" and returns a value of 12
printWithoutCounting(string: "hello, world")
// prints "hello, world" but does not return a value
```

### 返回多个值的方法 (Functions with Multiple Return Values)

使用多元组作为返回值，来返回多个值：

```swift
fun mixMax(array: [Int]) -> (min: Int, max: Int) {
	var currentMix = array[0]
	var currentMax = array[1]
	for value in array[1..<arry.count] {
		if value < currentMix {
			currrentMix = value
		}
		else if value > currentMax {
			currentMax = value
		}
	}
	return (currentMix, currentMax)
}

let bounds = mixMax(array: [8, -6, 2, 109, 3, 71])
print("min is (bounds.min), and max is \(bounds.max)")
// Prints "min is -6 and max is 109"
```

再返回多元组的时候，不用再给元素命名，因为方法的返回值已经命名。

### 可选多元组返回类型 (Optional Tuple Return Types)

如果方法返回的多元组有可能没有值，我们在定义方法返回值时可以使用可选类型，例如`(Int, Int)?`和`(String, Int, Bool)?`。

**注意：** `(Int, Int)?`多元组可选类型不同于包含可选类型元素的多元组`(Int?, Int?)`。`(Int, Int)?`多元组是可选类型，而`(Int?, Int?)`其中的元素才是可选了类型。


## 方法参数标签和参数名 (Function Argument Labels and Parameter Names)

每个方法的参数都有一个参数标签和参数名。参数标签使你在调用方法的时候使用，参数名实在方法实现里调用。默认情况下，参数表前和参数名是相同的。

```swift
func someFunction(firstParameterName: Int, secondParameterName: Int) {
    // In the function body, firstParameterName and secondParameterName
    // refer to the argument values for the first and second parameters.
}
someFunction(firstParameterName: 1, secondParameterName: 2)
```

所有的参数名必须是唯一的，参数标签有可能相同。但是为了提高代码的可读性，参数标签唯一可以提高代码的可读性。

### 指定参数标签 (Specifying Argument Labels)

把参数标签写在参数名前面：

```swift
func someFunction(argumentLabel parameterName: Int) {
    // In the function body, parameterName refers to the argument value
    // for that parameter.
}
```

例如：

```swift
func greet(person: String, from hometown: String) -> String {
    return "Hello \(person)!  Glad you could visit from \(hometown)."
}
print(greet(person: "Bill", from: "Cupertino"))
// Prints "Hello Bill!  Glad you could visit from Cupertino."
```

使用参数标签，调用方法就像调用一个句子一样，非常清晰，可读性非常高。

### 删除参数标签 (Omitting Argument Labels)

如果我们不需要参数标签，使用`_`来代替参数标签：

```swift
func someFunction(_ firstParameterName: Int, secondParameterName: Int) {
    // In the function body, firstParameterName and secondParameterName
    // refer to the argument values for the first and second parameters.
}
someFunction(1, secondParameterName: 2)
```

如果一个参数有标签，在调用方法时必须写标签。

### 参数默认值 (Default Parameter Values)

在声明方法时，可以给参数一个默认值：

```swift
func someFunction(parameterWithoutDefault: Int, parameterWithDefault: Int = 12) {
    // If you omit the second argument when calling this function, then
    // the value of parameterWithDefault is 12 inside the function body.
}
someFunction(parameterWithoutDefault: 3, parameterWithDefault: 6) // parameterWithDefault is 6
someFunction(parameterWithoutDefault: 4) // parameterWithDefault is 12
```

### 可变参数 (Variadic Parameters)

一个可变参数可以接受0个或多个指定类型的值。使用可变参数来指定这个参数在方法调用时可以被传入多个值，
传入方法的多个值在方法体中会作为一个数组。：

```swift
func arithmeticMean(_ numbers: Double...) -> Double {
	var total: Double = 0
	for number in numbers {
		total += number
	}
	return total / Double(numbers.count)
}
arithmeticMean(1, 2, 3, 4, 5)
// returns 3.0, which is the arithmetic mean of these five numbers
arithmeticMean(3, 8.25, 18.75)
// returns 10.0, which is the arithmetic mean of these three numbers
```

**注意：** 一个方法最多只能有一个可变参数。

### In-Out 参数 (In-Ont Parameters)

方法的参数默认情况下是属于常量，在方法中是不能改为参数值的。如果我们想在一个方法中改变参数的值，并且在方法执行完之后保持改变，应该把参数定义为*in-out*参数。

使用`inout`关键字来定义一个in-out参数，把`inout`放在参数类型的前面。对于in-out参数，在方法调用时，我们只能传入变量，不能传入常量或者字面值，因为常量和字面值是不能修改的。把变量传给一个参数时，要在变量名前面写一个`&`，来提示它是可以变改变的。

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
	let tempA = a
	a = b
	b = tempA
}

var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
print("someInt is now \(someInt), and anotherInt is now \(anotherInt)")
// Prints "someInt is now 107, and anotherInt is now 3"
```

## 方法类型 (Function Types)

每个方法都有一个特定的方法类型，并且由参数类型和返回值类型组成。

```swift
func addTwoInts(_ a: Int, _ b: Int) -> Int {
    return a + b
}
func multiplyTwoInts(_ a: Int, _ b: Int) -> Int {
    return a * b
}
```

这两个方法的类型都是`(Int, Int) -> Int`，可以这么理解：这个方法有两个`Int`类型的参数，返回一个`Int`类型的值。

下面是一个没有参数的方法：

```swift
func printHelloWorld() {
    print("hello, world")
}
```

这个方法的类型是`() -> Void`。

### 使用方法类型 (Using Function Types)

```swift
var mathFunction: (Int, Int) -> Int = addTwoInts
```

可以理解为：定义一个`mathFunction`变量，这个变量的类型是有两个`Int`类型参数并返回值为`Int`类型的方法，然后把这个变量指向`addTwoInts`。

使用`mathFunction`方法：

```swift
print("Result: \(mathFunction(2, 3))")
// Prints "Result: 5"
```

类型相同的不同方法可以赋值给同一个变量，例如：

```swift
mathFunction = multiplyTwoInts
print("Result: \(mathFunction(2, 3))")
// Prints "Result: 6"
```

### 方法类型作为参数类型 (Function Types as Parameter Types)

我们可以使用方法类型作为参数类型。例如：

```swift
func printMathResult(_ mathFunction: (Int, Int) -> Int, _ a: Int, _ b: Int) {
	print("Result: \(mathFunction(a, b))")
}
printMathResult(addTwoInts, 3, 5)
// Prints "Result: 8"
```

### 方法类型作为返回值类型 (Function Types as Return Types)

```swift
func stepForward(_ input: Int) -> Int {
	return input + 1
}
func stepBackward(_ input: Int) -> Int {
	return input - 1
}

func chooseStepFunction(backward: Bool) -> (Int) -> Int {
	backward ? stepBackward : stepForward
}
```

使用`chooseStepFunction(backward:)`方法来选择前进的方向：

```swift
var currentValue = 3
let moveNearerToZero = chooseStepFunction(backward: currentValue > 0)
// moveNearerToZero now refers to the stepBackward() function
```

`moveNearerToZero`已经引用了一个正确的方法，可以让一个值越来越接近0：

```swift
print("Counting to zero:")
while currentValue != 0 {
	print("\(currentValue)...")
	currentValue = moveNearToZero(currentValue)
}
print("zero!")
// 3...
// 2...
// 1...
// zero!
```

## 嵌套方法 (Nested Functions)

目前所接触到的所有方法都是在全局区域定义的全局方法。其实我们还可以在一个方法内部定义其他方法。嵌套的方法默认情况下，外面是访问不到的，只有包含这个嵌套方法的方法内部才可访问。但是包含嵌套方法的方法可以把其嵌套方法作为返回值供外面使用，所以上面的`chooseStepFunction(backward:)`可以改写为：

```swift
func chooseStepFunction(backward: Bool) -> (Int) -> Int {
	func stepForward(input: Int) -> Int { return input +1 }
	func stepBackward(input: Int) -> Int { return input - 1 }
    return backward ? stepBackward : stepForward
}
var currentValue = -4
let moveNearerToZero = chooseStepFunction(backward: currentValue > 0)
// moveNearerToZero now refers to the nested stepForward() function
while currentValue != 0 {
    print("\(currentValue)... ")
    currentValue = moveNearerToZero(currentValue)
}
print("zero!")
// -4...
// -3...
// -2...
// -1...
// zero!
```
