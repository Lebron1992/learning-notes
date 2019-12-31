
在 7 月 29 日的发布的 Xcode 11 beta 5 中，包含了 Swift 5.1。如果想要体验这些新特性，需要至少安装好这个版本的 Xcode。本文内容主要参考 **Raywenderlich** 的[这篇文章](https://www.raywenderlich.com/4187396-what-s-new-in-swift-5-1?utm_campaign=rw-weekly-issue-228&utm_medium=email&utm_source=rw-weekly#toc-anchor-001) 编写，如果想要查看原文，请点击链接查看。

Swift 5.1 在 5.0 引入的 ABI 稳定性基础上增加了**模块稳定性**。虽然 ABI 稳定性在运行时考虑到应用程序的兼容性，但模块稳定性在编译时支持库的兼容性。这意味着你可以在任何编译器版本中使用第三方框架，而不只是构建它的那个版本。

下面我们一起看一下有哪些改进。

## 模糊的结果类型 (Opaque Result Types)

在开发的时候，有时候可能会使用 protocol 作为返回值类型。下面来看一个例子：

```swift
protocol BlogPost {
    var title: String { get }
    var author: String { get }
}

struct Tutorial: BlogPost {
    let title: String
    let author: String
}

func createBlogPost(title: String, author: String) -> BlogPost {
    guard !title.isEmpty && !author.isEmpty else {
        fatalError("No title and/or author assigned!")
    }
    return Tutorial(title: title, author: author)
}

let swift4Tutorial = createBlogPost(title: "What's new in Swift 4.2?",
                                    author: "Cosmin Pupăză")
let swift5Tutorial = createBlogPost(title: "What's new in Swift 5?",
                                    author: "Cosmin Pupăză")
```

上面代码：1）首先定义 `BlogPost` 协议；2）定义 `Tutorial` 并实现 `BlogPost` 协议；3）定义 `createBlogPost()` 方法用于创建 `BlogPost`；4）用 `createBlogPost()` 创建 `swift4Tutorial` 和 `swift5Tutorial` 两个实例。

下面我们想比较 `swift4Tutorial` 和 `swift5Tutorial` 是否相等：

```swift
// 错误：Binary operator '==' cannot be applied to two 'BlogPost' operands
let isSameTutorial = (swift4Tutorial == swift5Tutorial)
```

因为 `BlogPost` 还没有实现 `Equatable`，所以会出错。下面让 `BlogPost` 继承自 `Equatable`：

```swift
protocol BlogPost: Equatable {
    var title: String { get }
    var author: String { get }
}
````

这时另一个错误出现在 `createBlogPost()` 方法：

```swift
// Protocol 'BlogPost' can only be used as a generic constraint because it has Self or associated type requirements
func createBlogPost(title: String, author: String) -> BlogPost {
    guard !title.isEmpty && !author.isEmpty else {
        fatalError("No title and/or author assigned!")
    }
    return Tutorial(title: title, author: author)
}
```

这个错误的意思是 `BlogPost` 只能用来作为泛型约束，因为它有 `Self` 或者有关联类型要求。查看 `Equatable`的定义，确实是有 `Self`：

```swift
public protocol Equatable {
    static func == (lhs: Self, rhs: Self) -> Bool
}
```

具有关联类型的协议不是类型，即使他们看起来是类型。而它们更像是类型占位符，这个类型可以是任何实现了我这个协议的类型。

在 Swift 5.1 中，我们就可以在返回值类型前面加上 `some` 来解决这个问题。把 `createBlogPost()` 方法改为：

```swift
func createBlogPost(title: String, author: String) -> some BlogPost {
    guard !title.isEmpty && !author.isEmpty else {
        fatalError("No title and/or author assigned!")
    }
    return Tutorial(title: title, author: author)
}
```

`some` 的作用就是告诉编译器我这个方法的返回值可以是实现了 `BlogPost` 的任何类型。

修改完成之后，我们直接用 `==` 比较 `swift4Tutorial` 和 `swift5Tutorial` 就不会报错了。

在 SwiftUI 中，就是使用了 `some` ：

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello World")
    }
}
```

## 隐式返回

在 Swift 5.1 中，如果方法体只有一行语句，则可以省略 `return`：

```swift
func myName() -> String {
	"Lebron"
}
```

## 属性包装器

在 Swift 5.1 之前，使用计算属性时，可能出现类似下面的代码：

```swift
var settings = ["swift": true, "latestVersion": true]

struct Settings {
    var isSwift: Bool {
        get {
            return settings["swift"] ?? false
        }
        set {
            settings["swift"] = newValue
        }
    }
    
    var isLatestVersion: Bool {
        get {
            return settings["latestVersion"] ?? false
        }
        set {
            settings["latestVersion"] = newValue
        }
    }
}

var newSettings = Settings()
newSettings.isSwift
newSettings.isLatestVersion
newSettings.isSwift = false
newSettings.isLatestVersion = false
```

上面的代码中，如果有更多的计算属性，那么就要写跟多的重复代码。为了解决这个问题，Swift 5.1 引入了属性包装器，可以把上面的代码简写为：

```swift
var settings = ["swift": true, "latestVersion": true]

@propertyWrapper
struct SettingsWrapper {
  let key: String
  let defaultValue: Bool

  var wrappedValue: Bool {
    get {
      settings[key] ?? defaultValue
    }
    set {
      settings[key] = newValue
    }
  }
}

struct Settings {
  @SettingsWrapper(key: "swift", defaultValue: false)         var isSwift: Bool
  @SettingsWrapper(key: "latestVersion", defaultValue: false) var isLatestVersion: Bool
}
```

- `@propertyWrapper` 把 `SettingsWrapper` 标记为属性包装器。作为一个属性包装器，必须有一个名为`wrappedValue` 的属性。
- 使用 `@SettingsWrapper` 标记 `Settings`中对应的属性。

## 在 struct 中定义属性的默认值

在 Swift 5.1 前，如果想要给 struct 的属性定义默认值，必须这么写：

```swift
struct Author {
    let name: String
    var tutorialCount: Int
    
    init(name: String, tutorialCount: Int = 0) {
        self.name = name
        self.tutorialCount = tutorialCount
    }
}

let author = Author(name: "George")
```

而在 Swift 5.1 以后，可以直接像 class 那样给属性定义默认值：

```swift
struct Author {
    let name: String
    var tutorialCount = 0
}
```

## 使用 `Self` 调用静态成员

在 Swift 5.1 以前，需要使用 `类名.静态成员`来调用静态成员：

```swift
struct Editor {
    static func reviewGuidelines() {
        print("Review editing guidelines.")
    }
    
    func edit() {
        Editor.reviewGuidelines()
        print("Ready for editing!")
    }
}
```

而在 Swift 5.1 中，可以直接用 `Self.静态成员`：

```swift
struct Editor {
    static func reviewGuidelines() {
        print("Review editing guidelines.")
    }
    
    func edit() {
        Self.reviewGuidelines()
        print("Ready for editing!")
    }
}
```

## 创建未初始化的数组

Swift 5.1 给 `Array` 添加了一个新的初始化方法：`init(unsafeUninitializedCapacity:initializingWith:)`

```swift
let randomSwitches = Array<String>(unsafeUninitializedCapacity: 5) { buffer, count in
    for i in 0..<5 {
        buffer[i] = Bool.random() ? "on" : "off"
    }
    // 必须给 `count` 赋值，否则 `randomSwitches` 会变成空数组
    count = 5
}
```

## `static` 和 `class` 下标

在 Swift 5.1 中可以定义 `static` 和 `class` 下标：

```swift
@dynamicMemberLookup
class File {
    let name: String
    
    init(name: String) {
        self.name = name
    }
    
    // 定义 static 下标
    static subscript(key: String) -> String {
        switch key {
        case "path":
            return "custom path"
        default:
            return "default path"
        }
    }
    
    // 使用 Dynamic Member Lookup 重写上面的下标
    class subscript(dynamicMember key: String) -> String {
        switch key {
        case "path":
            return "custom path"
        default:
            return "default path"
        }
    }
}

File["path"] // "custom path"
File["PATH"] // "default path"
File.path    // "custom path"
File.PATH    // "default path"
```

用 `@dynamicMemberLookup` 标记 `File` 是为了可以使用点语法来访问自定义的下标。

## Keypath 支持动态成员查找

```swift
struct Point {
    let x, y: Int
}

@dynamicMemberLookup
struct Circle<T> {
    let center: T
    let radius: Int
    
    // 定义泛型下标，可以用 keypath 来访问 `center` 的属性
    subscript<U>(dynamicMember keyPath: KeyPath<T, U>) -> U {
        center[keyPath: keyPath]
    }
}

let center = Point(x: 1, y: 2)
let circle = Circle(center: center, radius: 1)
circle.x // 1
circle.y // 2
```

## Tuple 支持 Keypath

```swift
struct Instrument {
    let brand: String
    let year: Int
    let details: (type: String, pitch: String)
}

let instrument = Instrument(
    brand: "Roland",
    year: 2019,
    details: (type: "acoustic", pitch: "C")
)
// 使用 keypath 访问 tuple
let type = instrument[keyPath: \Instrument.details.type]
let pitch = instrument[keyPath: \Instrument.details.pitch]
```

## `weak` 和 `unowned` 属性自动实现 `Equatable` 和 `Hashable`

```swift
class Key {
    let note: String
    
    init(note: String) {
        self.note = note
    }
}

extension Key: Hashable {
    static func == (lhs: Key, rhs: Key) -> Bool {
        lhs.note == rhs.note
    }
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(note)
    }
}

class Chord {
    let note: String
    
    init(note: String) {
        self.note = note
    }
}

extension Chord: Hashable {
    static func == (lhs: Chord, rhs: Chord) -> Bool {
        lhs.note == rhs.note
    }
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(note)
    }
}

struct Tune: Hashable {
    unowned let key: Key
    weak var chord: Chord?
}
```

在 Swift 5.1 以前，`Tune` 的定义里面会报错：没有实现 `Equatable` 和 `Hashable`；在 Swift 5.1 则已经自动实现。

## 不明确的枚举 case

如果有不明确的枚举 case，在 Swift 5.1 中会产生警告⚠️。

```swift
enum TutorialStyle {
  case cookbook, stepByStep, none
}

// 会产生警告
let style: TutorialStyle? = .none
```

因为 `style` 是 `Optional` 类型，编译器不知道 `.none` 是 `Optional.none` 还是 `TutorialStyle.none` ，所有要写具体一点，例如：`let style: TutorialStyle? = TutorialStyle.none`

## 匹配可选类型的枚举

在 Swift 5.1 以前，`switch` 语句中匹配可选类型的枚举时，case 后面需要加问号：

```swift
enum TutorialStatus {
    case written, edited, published
}

let status: TutorialStatus? = .published

switch status {
    case .written?:
        print("Ready for editing!")
    case .edited?:
        print("Ready to publish!")
    case .published?:
        print("Live!")
    case .none:
        break
}
```

而在 Swift 5.1 中，可以把问号去掉：

```swift
switch status {
    case .written:
        print("Ready for editing!")
    case .edited:
        print("Ready to publish!")
    case .published:
        print("Live!")
    case .none:
        break
}
```

## Tuple 类型的转换

```swift
let temperatures: (Int, Int) = (25, 30)
let convertedTemperatures: (Int?, Any) = temperatures
```

在 Swift 5.1 以前，`(Int, Int)` 是不能转换成 `(Int?, Any)` 的，而在 Swift 5.1 可以。

## Any 和泛型参数的方法重载

```swift
func showInfo(_: Any) -> String {
  return "Any value"
}

func showInfo<T>(_: T) -> String {
  return "Generic value"
}

showInfo("Swift")
```

在 Swift 5.1 以前，`showInfo("Swift")` 返回的是 `Any value`；而在 Swift 5.1 中，返回的是 `Generic value`。也就是说在 Swift 5.1 以前，Any 参数类型的方法优先；而在 Swift 5.1 中，泛型参数的方法优先。

## autoclosure 参数可以定义别名

在 Swift 5.1 中，`@autoclosure` 标记的 closure 参数可以使用别名：

```swift
struct Closure<T> {
    typealias ClosureType = () -> T
    
    func apply(closure:  @autoclosure ClosureType) {
        closure()
    }
}
```

在 Swift 5.1 以前，只能这样写：

```swift
struct Closure<T> {
    func apply(closure: @autoclosure () -> T) {
        closure()
    }
}
```

## 在 Objective-C 方法中返回 `self`

在 Swift 5.1 以前，被 `@objc` 标记的方法中返回 `self`，必须继承自 `NSObject`：

```swift
class Clone: NSObject {
    @objc func clone() -> Self {
        return self
    }
}
```

在 Swift 5.1 中，则无需集成 `NSObject`

```swift
class Clone {
  @objc func clone() -> Self {
    self
  }
}
```
