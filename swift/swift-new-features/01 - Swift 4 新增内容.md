#### 初始化一个多行字符串

可以直接在多行字符串中包含`"`

```swift
let quotation = """
The White Rabbit put on his spectacles.  "Where shall I begin,
please your Majesty?" he asked.
 
"Begin at the beginning," the King said gravely, "and go on
till you come to the end; then stop."
"""
```

可以直接在多行字符串中包含`"""`

```swift
let threeDoubleQuotes = """
Escaping the first quote \"""
Escaping all three quotes \"\"\"
"""
```

以下两个字符串是等价的：

```swift
let singleLineString = "These are the same."
let multilineString = """
These are the same.
"""
```

在字符串前后加空行，可以这样写：

```swift
"""
 
This string starts with a line feed.
It also ends with a line feed.
 
"""
```

如果一个多行字符串定义在一个方法中，实际字符串的值是不包含每一行前面的空格：

```swift
func generateQuotation() -> String {
    let quotation = """
        The White Rabbit put on his spectacles.  "Where shall I begin,
        please your Majesty?" he asked.
 
        "Begin at the beginning," the King said gravely, "and go on
        till you come to the end; then stop."
        """
    return quotation
}
print(quotation == generateQuotation())
// Prints "true"
```

如果把字符串改为下面的这个样子，第二行字符串前面的空格是不能忽略的：

```swift
func generateQuotation() -> String {
    let quotation = """
        The White Rabbit put on his spectacles.  "Where shall I begin,
                please your Majesty?" he asked.
 
        "Begin at the beginning," the King said gravely, "and go on
        till you come to the end; then stop."
        """
    return quotation
}
print(quotation == generateQuotation())
// Prints "false"
```

#### 泛型下标 (Generic Subscripts)

下标可以是泛型的，并且可以包含`where`语句：

```swift
extension Container {
    subscript<Indices: Sequence>(indices: Indices) -> [Item]
        where Indices.Iterator.Element == Int {
            var result = [Item]()
            for index in indices {
                result.append(self[index])
            }
            return result
    }
}
```

上述代码的意思是：接受一个下标序列，并返回对应的元素组成的数组：
- `Indices`必须遵循`Sequence`协议
- `indices`必须是`Indices`类型
- `where`语句要求`Indices`的元素必须是`Int`类型

#### 协议组合中可以包含父类

例如下面这个例子：

```swift
class Location {
    var latitude: Double
    var longitude: Double
    init(latitude: Double, longitude: Double) {
        self.latitude = latitude
        self.longitude = longitude
    }
}
class City: Location, Named {
    var name: String
    init(name: String, latitude: Double, longitude: Double) {
        self.name = name
        super.init(latitude: latitude, longitude: longitude)
    }
}
func beginConcert(in location: Location & Named) {
    print("Hello, \(location.name)!")
}
 
let seattle = City(name: "Seattle", latitude: 47.6, longitude: -122.3)
beginConcert(in: seattle)
// Prints "Hello, Seattle!"
```

在`beginConcert`方法中，`location`参数必须`Location`类型，并且遵循`Named`协议。

#### `final`关键字不能用在协议扩展中

在协议扩展里面，我们不能使用`final`。

#### Swift 3中，在extension访问private属性会报错；而在Swift 4中，允许我们这么做。

![1.jpg](http://upload-images.jianshu.io/upload_images/2057254-41ac6b1e9658a5f9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
￼￼￼
#### Xcode 9可以自定义编译器的Swift版本

![2.jpg](http://upload-images.jianshu.io/upload_images/2057254-d1dfac96e73caa8d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Xcode 9集成新的编译系统，编译速度更快

![3.jpg](http://upload-images.jianshu.io/upload_images/2057254-0a388857284f80fd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 在Swift和OC混编时，我们要使用bridging headers，Xcode9可以预编译 bridging headers，大幅度减少编译时间，apple music的编译速度提高40%

![4.jpg](http://upload-images.jianshu.io/upload_images/2057254-8ba142db375fadee.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 在访问字符串的index时，不用再写`value.characters.startIndex`，直接写成`value.startIndex`即可

![5.jpg](http://upload-images.jianshu.io/upload_images/2057254-4baf3fe6c34751db.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 访问字符串的从中间某个index到endIndex的字符，可以省略endIndex

![6.jpg](http://upload-images.jianshu.io/upload_images/2057254-c1d94bb07f3c2aa8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 不允许在遍历数组的时候，修改当前遍历的数组

![7.jpg](http://upload-images.jianshu.io/upload_images/2057254-958fe8fd2fa8edee.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
