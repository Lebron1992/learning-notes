### 属性 (Properties)

存储属性作为实例的一部分，存在常量和变量的值，而计算属性是计算一个值，而不是存储一个值。计算属性由类、结构和枚举提供；而存储属性只能由类和结构提供。

存储和计算属性通常有关联着一个实例。然而，属性可以关联类型。

另外，可以定义属性观察者来监测属性值的变化。属性观察者可以添加到自己定义的存储属性中，也可以添加到从父类继承的属性中。

#### 存储属性 (Stored Properties)

```swift
struct FixedLengthRange {
    var firstValue: Int
    let length: Int
}
var rangeOfThreeItems = FixedLengthRange(firstValue: 0, length: 3)
// the range represents integer values 0, 1, and 2
rangeOfThreeItems.firstValue = 6
// the range now represents integer values 6, 7, and 8
```

`firstValue`是变量存储属性，`length`是常量存储属性。

##### 常量结构实例的存储属性 (Stored Properties of Constant Structure Instances)

如果创建一个结构实例，并赋给一个常量，那么就不能修改实例的属性，即使这个属性是一个变量属性：

```swift
let rangeOfFourItems = FixedLengthRange(firstValue: 0, length: 4)
// this range represents integer values 0, 1, 2, and 3
rangeOfFourItems.firstValue = 6
// this will report an error, even though firstValue is a variable property
```

因为`rangeOfFourItems`被声明为常量，所以不能修改`firstValue`属性，即使`firstValue`是变量属性。

出现这种现象，其实是因为结构是值类型。只要实例是常量，那么它的属性也都是常量。

而对于引用类型的类而言，把一个引用类型的实例赋值给一个常量，我们任然可以修改实例的变量属性。

##### 懒惰存储属性 (Lazy Stored Properties)

一个属性在第一次使用之前，他的初始值没有被计算，那么这个属性称为懒惰存储属性。

**注意：**懒惰存储属性必须声明为一个变量，因为实例初始化完成后，懒惰属性可能还没被获取。常量属性在实例初始化完成之前必须有一个值，所有不能声明为懒惰属性。

当一个属性的初始值依赖于外部因素，而外部因素的值需要在实例初始化完成之后才能确定，那么把这个属性声明为懒惰属性非常有用。当一个属性的初始值需要很复杂的计算时，懒惰属性也非常有用。

下面这个例子使用懒惰属性来避免不必要的一个复杂类的初始化。

```swift
class DataImporter {
    /*
     DataImporter is a class to import data from an external file.
     The class is assumed to take a non-trivial amount of time to initialize.
     */
    var fileName = "data.txt"
    // the DataImporter class would provide data importing functionality here
}
 
class DataManager {
    lazy var importer = DataImporter()
    var data = [String]()
    // the DataManager class would provide data management functionality here
}
 
let manager = DataManager()
manager.data.append("Some data")
manager.data.append("Some more data")
// the DataImporter instance for the importer property has not yet been created
```

`DataManager`定义了一个`DataImporter`实例，这里假设初始化`DataImporter`实例需要很长时间。`DataManager`实例可以自己管理数据，而不必使用`DataImporter`实例来打开文件，所以当创建`DataManger`实例时，无需把`DataImporter`实例同时创建，而是在第一次使用的时候才创建。

```swift
print(manager.importer.fileName)
// the DataImporter instance for the importer property has now been created
// Prints "data.txt"
```

**注意**：如果一个懒惰属性被多个线程同时访问，而且这个属性还没有被初始化时，不能保证这个属性只被初始化一次。

#### 计算属性 (Computed Properties)

计算属性提供了一个getter和一个可选的setter方法来间接地获取和设置其他属性和其他值。

```swift
struct Point {
    var x = 0.0, y = 0.0
}
struct Size {
    var width = 0.0, height = 0.0
}
struct Rect {
    var origin = Point()
    var size = Size()
    var center: Point {
        get {
            let centerX = origin.x + (size.width / 2)
            let centerY = origin.y + (size.height / 2)
            return Point(x: centerX, y: centerY)
        }
        set(newCenter) {
            origin.x = newCenter.x - (size.width / 2)
            origin.y = newCenter.y - (size.height / 2)
        }
    }
}
var square = Rect(origin: Point(x: 0.0, y: 0.0),
                  size: Size(width: 10.0, height: 10.0))
let initialSquareCenter = square.center
square.center = Point(x: 15.0, y: 15.0)
print("square.origin is now at (\(square.origin.x), \(square.origin.y))")
// Prints "square.origin is now at (10.0, 10.0)"
```

这个例子定义了三个结构：

- `Point`封装了一个点的x和y坐标
- `Size`封装了宽和高
- `Rect`使用原点和尺寸定义了一个矩形

`Rect`还定义了一个计算属性`center`。矩形的中心可以同原点和尺寸计算出来，所以不必用存储属性来定义矩形的中心。`Rect`为计算属性定义了一个自定义的getter和setter。

上面这个例子中创建了一个正方形`square`，如下图中的蓝色矩形。然后`square`的`center`属性通过点语法被访问，会使`center`的getter方法被调用，用来获取属性的当前值，getter会返回一个新的`Point`代表正方形的中心(当前是`(5, 5)`)。

接着`center`被设置为`(15, 15)`，把正方形往右上角移动，如下图的橙色正方形。设置`center`属性会调用setter方法，修改`x`和`y`的值，把正方形移到一个新的位置。

![Coordinate](http://upload-images.jianshu.io/upload_images/2057254-cf6aa3b0f66da8fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 简写Setter的定义 (Shorthand Setter Declaration)

如果setter方法没有给新的值定义名字，我们可以使用一个默认名字`newValue`。

```swift
struct AlternativeRect {
    var origin = Point()
    var size = Size()
    var center: Point {
        get {
            let centerX = origin.x + (size.width / 2)
            let centerY = origin.y + (size.height / 2)
            return Point(x: centerX, y: centerY)
        }
        set {
            origin.x = newValue.x - (size.width / 2)
            origin.y = newValue.y - (size.height / 2)
        }
    }
}
```

##### 只读计算属性 (Read-Only Computed Properties)

一个计算属性只有getter没有setter，那么这个计算属性就是只读计算属性。只读计算属性智能返回一个值，通过点语法访问，但是不能设置新的值。

**注意：**必须用`var`把计算属性(包括只读计算属性)定义为可变属性，因为他们的值是不固定的。

定义只读属性时，我们可以把`get`关键字和大括号省略：

```swift
struct Cuboid {
    var width = 0.0, height = 0.0, depth = 0.0
    var volume: Double {
        return width * height * depth
    }
}
let fourByFiveByTwo = Cuboid(width: 4.0, height: 5.0, depth: 2.0)
print("the volume of fourByFiveByTwo is \(fourByFiveByTwo.volume)")
// Prints "the volume of fourByFiveByTwo is 40.0"
```

#### 属性观察者 (Property Observers)

属性观察者观察属性值得变化，并作出响应。只要属性的值被设置，属性观察者就会被调用，即使新的值和属性的当前值一样。

我们可以把属性观察者添加到自己定义的任何存储属性中，除了懒惰存储属性。也可以通过重写父类的属性来添加到父类的属性中。不必为没有重写的计算属性定义属性观察者，因为我们可以在计算属性的setter方法观察他的值的变化，并作出响应。

两种观察属性值变化的方法：

- `willSet`：新的值被存储之前调用
- `didSet`：新的值被存储之后马上调用

如果实现一个`willSet`观察者，会把新的值作为一个常量参数。可以为这个新的值指定一个名字，如果没有指定，那么默认是`newValue`。

如果实现了一个`didSet`观察者，会把旧的值作为一个常量参数。可以为这个旧的值指定一个名字，如果没有指定，那么默认是`oldValue`。如果在`didSet`观察者里面给属性赋一个新值，那么这个新值会替换掉刚刚设置的那个值。

**注意：**当父类的属性在子类的初始化器被设置，那么父类属性的`willSet`和`didSet`会在父类的初始化器调用之后被调用。在设置父类自己的属性时，父类的`willSet`和`didSet`在父类的初始化器调用之前不会被调用。

```swift
class StepCounter {
    var totalSteps: Int = 0 {
        willSet(newTotalSteps) {
            print("About to set totalSteps to \(newTotalSteps)")
        }
        didSet {
            if totalSteps > oldValue  {
                print("Added \(totalSteps - oldValue) steps")
            }
        }
    }
}
let stepCounter = StepCounter()
stepCounter.totalSteps = 200
// About to set totalSteps to 200
// Added 200 steps
stepCounter.totalSteps = 360
// About to set totalSteps to 360
// Added 160 steps
stepCounter.totalSteps = 896
// About to set totalSteps to 896
// Added 536 steps
```

**注意：**如果把含有观察者的属性作为一个in-out参数传入一个方法，那么`willSet`和`didSet`总是会被调用，这是因为in-out参数的"copy-in copy-out"内存模式：在方法结束时，这个值会被返回被属性。

#### 全局和本地变量 (Global and Local Variables)

全局变量是在任何方法、闭包和类型上下文之外定义的。而局部变量是在方法、闭包和类型上下文之内定义的。

**注意：**全局常量和变量都是懒惰地计算，类似懒惰存储属性。但是全局常量和变量不需要使用`lazy`标记。本地常量和变量从来都不是懒惰的计算。

#### 类型属性 (Type Properties)

实例属性是属于一个特定类型的实例。我们可以定义属于类型的属性。

**注意：**不同于实例属性，我们必须给类型属性一个默认值。因为类型没有初始化器来给类型属性赋值。类型存储属性只要在第一次被访问的时候才会初始化，并且保证只会初始化一次，即使是在多线程中同时被访问，不需要使用`lazy`关键字标记。

##### 类型属性语法 (Type Property Syntax)

使用`static`关键字来定义类型属性，对于class类型的计算类型属性，可以使用`class`来定义，可以让子类重新父类的实现。

```swift
struct SomeStructure {
	static var storedTypeProperty = "Some value."
	static var computedTypeProperty: Int {
		return 1
	}
}

enum SomeEnumeration {
	static var storedTypeProperty = "Some value."
	static var computedTypeProperty: Int {
		return 6
	}
}

class SomeClass {
	static var storedTypeProperty = "Some value."
	static var computedTypeProperty: Int {
		return 27
	}
	class var overrideableComputedTypeProperty: Int {
		return 107
	}
}
```

##### 查询和设置类型属性 (Querying and Setting Type Properties)

就像实例属性一样，类型属性也是通过点语法来查询和设置的：

```swift
print(SomeStructure.storedTypeProperty)
// Prints "Some value."
SomeStructure.storedTypeProperty = "Another value."
print(SomeStructure.storedTypeProperty)
// Prints "Another value."
print(SomeEnumeration.computedTypeProperty)
// Prints "6"
print(SomeClass.computedTypeProperty)
// Prints "27"
```
