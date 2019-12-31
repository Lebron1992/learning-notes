### 下标 (Subscripts)

类、结构和枚举都可以定义下标，能让我们快速访问一个集合、列表或序列的成员元素。一个类型可以定义多个下标。

#### 下标语法 (Subscript Syntax)

下标语法类似于实例方法和计算属性语法。使用`subscript`关键字来定义下标，然后指定一个或多个参数和返回类型，就像实例方法一样。但不同于实例方法，下标可以读写或者只读。

```swift
subscript(index: Int) -> Int {
	get {
        // return an appropriate subscript value here
	}
	set {
        // perform a suitable setting action here
	}
}
```

`newValue`的类型与下标返回值类型相同，当然我们也可以不用指定参数名，Swift会默认提供一个`newValue`的参数名供我们使用。

就像只读计算属性一样，我们写只读下标时可以把`get`去掉：

```swift
subscript(index: Int) -> Int {
	return an appropriate subscript value here
}
```

下面是一个例子：

```swift
struct TimesTable {
	let multiplier: Int
	subscript(index: Int) -> Int {
		return multiplier * index
	}
}
let threeTimesTable = TimeTable(multiplier: 3)
print("six times three is \(threeTimeTable[6])")
```

#### 下标的使用 (Subscript Usage)

下标的意义决定于它所在的上下文。下标通常作为一个捷径，用于方法集合、列表或者序列的成员元素。

例如，Swift的`Dictionary`就是实现下标来设置和获取存储在字典的值。

```swift
var numberOfLegs = ["spider": 8, "ant": 6, "cat": 4]
numberOfLegs["bird"] = 2
```

#### 下标选项 (Subscript Options)

下标可以接受任意数量任意类型的参数，也可以返回任意类型的值，还可以使用可变参数，但是不能使用in-out参数和给参数提供默认值。

如果有需要的话，类和结构可以提供多个下标实现。在使用时，他会根据中括号内的值或者值的类型来选择合适的下标。定义多个下标被称为*下标重载*。

通常情况下，下标只带一个参数，但是也可以带多个参数。例如下面的矩阵结构：

```swift
struct Matrix {
	let rows: Int, columns: Int
	var grid: [Double]
	
	init(rows: Int, columns: Int) {
		self.rows = rows
		self.columns = columns
		grid = Array(repeat: 0.0, count: rows * columns)
	}
	
	func indexIsValid(row: Int, column: Int) -> Bool {
		return row >= 0 && row < rows && column >= 0 && column < columns
	}
	
	subscript(row: Int, column: Int) -> Double {
		get {
			assert(indexIsValid(row: row, column: column), "Index out of range")
			return grid[(row * columns) + column]
		}
		set {
			assert(indexIsValid(row: row, column: column), "Index out of range")
			grid[(row * columns) + column] = newValue
		}
	}
}
```

创建一个矩阵：

```swift
var matrix = Matrix(rows: 2, columns: 2)
```

可以使用下图表示：

![matrix](http://upload-images.jianshu.io/upload_images/2057254-4a66b32cc1e7562f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用下标来设置矩阵里面的值：

```swift
matrix[0, 1] = 1.5
matrix[1, 0] = 3.2
```

矩阵的值变为：

![matrix](http://upload-images.jianshu.io/upload_images/2057254-42190cceffedfa7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
