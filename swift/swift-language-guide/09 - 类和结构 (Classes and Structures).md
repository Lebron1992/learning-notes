### 类和结构 (Classes and Structures)

在Swift中，Swift的类和结构不需要创建接口和实现两个文件，只需要在一个文件中定义类或者结构，外部接口会自动生成给其他代码使用。

#### 类和结构的对比 (Comparing Classes and Structures)

Swift的类和结构有很多共同点：

- 定义属性来存储值
- 定义提供某些功能的方法
- 定义下标来通过下标访问它们的值
- 定义初始化器来设置他们的初始状态
- 可以扩展来扩展默认实现没有的功能
- 可以遵循协议来提供某种标准化的功能

类拥有结构没有的功能：

- 从其他类中继承
- 类型转换
- 类的实例可以使用Deinitializers清除他自己定义的资源
- 引用计数允许一个类实例有一个或多个引用

**注意：**结构在使用中被传递时，是通过复制的方式，不是引用计数。

##### 定义语法 (Definition Syntax)

通用形式如下：

```swift
class SomeClass {
    // class definition goes here
}
struct SomeStructure {
    // structure definition goes here
}
```

下面是一个例子：

```swift
struct Resolution {
    var width = 0
    var height = 0
}
class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
}
```

##### 类和结构的实例 (Class and Structure Instances)

创建实例的语法：

```swift
let someResolution = Resolution()
let someVideoMode = VideoMode()
```

通过这种方式创建的实例，实例的属性都初始化为默认值。

##### 访问属性 (Accessing Properties)

使用点语法访问属性：

```swift
print("The width of someResolution is \(someResolution.width)")
// Prints "The width of someResolution is 0"

print("The width of someVideoMode is \(someVideoMode.resolution.width)")
// Prints "The width of someVideoMode is 0"
```

使用点语法给变量赋值：

```swift
someVideoMode.resolution.width = 1280
print("The width of someVideoMode is now \(someVideoMode.resolution.width)")
// Prints "The width of someVideoMode is now 1280"
```

**注意：**不同于OC，Swift允许我们直接修改结构属性的子属性，在上面的例子中，`someVideoMode`的`resolution`属性的`width`属性是直接设置的，无需设置整个`resolution`属性的值。

##### 结构类型的成员逐一始化器 (Memberwise Initializers for Structure Types)

所有结构都有一个自动生成的成员逐一始化器，我们可以用来初始化成员属性的值。

```swift
let vga = Resolution(width: 640, height: 480)
```

不同于结构，类是没有默认的初始化器的。

#### 结构和枚举都是值类型 (Structures and Enumerations Are Value Types)

值类型就是当它被赋值给变量或常量，或者被当做参数传入方法时，是通过赋值的方式实现的。

实际上，Swift的基本类型，例如整型、浮点型、布尔值、字符串、数组和字典都是是值类型，底层都是通过结构的实现的。

```swift
let hd = Resolution(width: 1920, height: 1080)
var cinema = hd
```

`hd`和`cinema`有同样的高度和宽度，但是这两个是完全不一样的实例。

```swift
cinema.width = 2048
print("cinema is now \(cinema.width) pixels wide")
// Prints "cinema is now 2048 pixels wide"

print("hd is still \(hd.width) pixels wide")
// Prints "hd is still 1920 pixels wide"
```

`cinema`的`width`属性改为2048，但是`hd`的`width`属性还是1920。

#### 类是引用类型 (Classes Are Reference Types)

不同于值类型，引用类型被赋值给变量或常量，或者被当做参数传入方法时，不是通过复制实现的，而是引用同一个实例。

```swift
let tenEighty = VideoMode()
tenEighty.resolution = hd
tenEighty.interlaced = true
tenEighty.name = "1080i"
tenEighty.frameRate = 25.0

let alsoTenEighty = tenEighty
alsoTenEighty.frameRate = 30.0
```

因为类是引用类型，实际上`tenEighty`和`alsoTenEighty`引用着同一个`VideoMode`实例。`tenEighty`的`alsoTenEighty.frameRate`也是30.0。

```swift
print("The frameRate property of tenEighty is now \(tenEighty.frameRate)")
// Prints "The frameRate property of tenEighty is now 30.0"
```

注意：`tenEighty`和`alsoTenEighty`是常量，但是我们任然可以修改`frameRate`属性，因为`tenEighty`和`alsoTenEighty`的值实际上没有改变。`tenEighty`和`alsoTenEighty`不是存储`VideoMode`实例，而是引用着同一个`VideoMode`实例。在底层里实际上是`VideoMode`实例的`frameRate`属性在改变，而不是引用着`VideoMode`实例的常量改变。

##### 相同运算符 (Identity Operators)

使用下面两个运算符来判断两个常量或变量是否引用着同一个实例：

- 等于 (`===`)
- 不等于 (`!==`)

```swift
if tenEighty === alsoTenEighty {
    print("tenEighty and alsoTenEighty refer to the same VideoMode instance.")
}
// Prints "tenEighty and alsoTenEighty refer to the same VideoMode instance."
```

注意：`===`和`==`是不一样的：

- `===`意味着两个变量引用着同一个类实例。
- `==`是用来判断两个值是否相等

##### 指针 (Pointers)

如果你有C、C++或者是OC的经验，你肯定会知道这些语言是使用指针来引用内存地址。Swift的常量或变量引用一个类实例类似于C语言的指针，但不是一个直接的指针指向内存地址，而且也不需要使用`*`来暗示我们正在创建一个引用。

#### 类和结构的选择 (Choosing Between Classes and Structures)

我们可以使用类和结构来定义数据类型，但是结构是值类型，而类是引用类型。

在下列的情况下，考虑使用结构：

- 结构最初的目的是用来封装一些相关并简单的数据
- 封装的值在被复制给其他变量或者方法时，是通过复制的方式而不是引用
- 结构的所有属性都是值类型，通过复制的方式更合理
- 结构不需要集成其他类型的属性或方法

下面是非常适合用结构的场景：

- 几何形状的尺寸，也许要封装`width`和`height`属性，都是`Double`类型
- 在一个系列类引用一个范围的一个方式，封装`start`和`length`属性，都是`Int`类型
- 3D坐标系统的一个点，封装`x`、`y`和`z`属性，都是`Double`类型

#### 字符串、数组和字典的赋值和赋值行为

在Swift中，很多基本数据类型，例如`String`、`Array`和`Dictionary`都是用结构实现的。也就意味着他们被赋值给一个常量或者变量、或者被传入一个方法时是通过复制的方式实现的。

这个不同于`NSString`、`NSArray`和`NSDictionary`，他们是通过类实现的，不是结构。他们被赋值给一个常量或者变量、或者被传入一个方法时是通过引用的方式实现的，不是复制。
