## 初始化一个多行字符串

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

## 泛型下标 (Generic Subscripts)

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

## 协议组合中可以包含父类

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

## `final`关键字不能用在协议扩展中

在协议扩展里面，我们不能使用`final`。
