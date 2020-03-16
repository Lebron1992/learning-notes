> 本文是阅读 [Combine: Asynchronous Programming with Swift](https://store.raywenderlich.com/products/combine-asynchronous-programming-with-swift) 后的学习笔记，有需要的请点击链接购买。

# 10 - Key-Value Observing

前面已经学习了 `assign(to:on:)`，它使我们能够在每次 publisher 发出新值时更新给定对象的属性值。

但是，我们如何观察一个变量的变化呢？Combine 提供了以下几个方法：

- 它为支持 KVO 的对象的任何属性提供 publisher。
- `ObservableObject` 协议用于处理多个变量发生变化的情况。

## `publisher(for:options:)`

我们很容易观察到支持 KVO 的属性。下面是使用 `OperationQueue` 的示例：

```swift
let subscription = queue.publisher(for: \.operationCount)
    .sink {
        print("Outstanding operations in queue: \($0)")
    }
```

每次向队列中添加新 operation 时，它的 `operationCount` 都会增加，subscriber 就会收到新计数。当队列已执行了一个 operation 时，计数将递减，然后 subscriber 接收更新的计数。

## 准备并订阅自己的 KVO 属性

我们可以在代码中实现自己的 `KVO` 属性，需要满足的条件如下：

- 对象必须是 class (不能是 struct)，并且继承自 `NSObject`。
- 把属性标记为 `@objc dynamic`。

然后我们的属性就可以被 Combine 观察。

> 虽然 Swift 语言不直接支持 KVO，但是用 `@objc dynamic` 标记属性会强制编译器生成触发 KVO 机制的隐藏方法。可以说，这个机制在很大程度上依赖于 `NSObject` 的特定方法，这就解释了为什么您的对象需要从继承它。

下面是一个例子：

```swift
class TestObject: NSObject {
    @objc dynamic var integerProperty: Int = 0
}

let obj = TestObject()

let subscription = obj.publisher(for: \.integerProperty)
    .sink {
        print("integerProperty changes to \($0)")
    }

obj.integerProperty = 100
obj.integerProperty = 200
```

运行结果：

```
integerProperty changes to 0
integerProperty changes to 100
integerProperty changes to 200
```

从打印可以看到，初始值 `0` 也被观察到了。

另外要注意的是，KVO 对于任何 Objective-C 类型和任何桥接到 Objective-C 的 Swift 类型都可以正常工作。这包括所有本地 Swift 类型以及数组和字典，前提是它们的值都可以桥接到 Objective-C。

如果把一个没有桥接到 Objective-C 的纯 Swift 类型定义成 `@objc dynamic` 类型，那就回报错。

例如：

```swift
struct PureSwift {
    let a: (Int, Bool)
}
```

在刚刚的 `TestObject` 添加：

```swift
@objc dynamic var structProperty: PureSwift = .init(a: (0,false))
```

报错：

```
Property cannot be marked @objc because its type cannot be represented in Objective-C
```

## 观察选项

`publisher(for:options:)` 方法还有第二个参数，是一个 `OptionSet` 类型，可选的值为：`.initial` 、 `.prior` 、 `.old` 和 `.new`。默认值是 `[.initial, .new]`，其中包含了 `.initial`，这也就为什么刚刚能看到打印初始值的原因。

以下是每个值的解释：

- `.initial`：发出初始值。
- `.prior`：发出前一个值和当前值。
- `.old` 和 `.new` 在这里不起任何作用，也就是发出新的值，没有它们也一样。

如果不想要初始值，直接像下面这样写即可：

```swift
obj.publisher(for: \.integerProperty, options: [])
```

如果设置了 `.prior`：

```swift
 let subscription = obj.publisher(for: \.integerProperty options: [.prior])
```

则打印结果如下：

```
integerProperty changes to 0
integerProperty changes to 100
integerProperty changes to 100
integerProperty changes to 200
```

每次改变属性的值，都会收到两个值：前一个老值和新的值。

## `ObservableObject`

Combine 的 `ObservableObject` 协议对 Swift 对象有效，而不仅仅是对继承自 NSObject 的对象有效。它与 `@Published` 属性包装器协作，帮助我们创建带有编译器生成的 `objectWillChange` publisher 的类。

它避免了编写大量模板文件，并允许创建自监视其自身属性，并在其中任何属性发生更改时通知的对象。

例如：

```swift
class MonitorObject: ObservableObject {
    @Published var someProperty = false
    @Published var someOtherProperty = ""
}

let object = MonitorObject()
let subscription = object.objectWillChange.sink {
    print("object will change")
}

object.someProperty = true
object.someOtherProperty = "Hello world"
```

运行结果：

```
object will change
object will change
```

`ObservableObject` 协议使编译器自动生成 `objectWillChange` 属性。它是一个可观察的 `ObservableObjectPublisher`，它发出 `Void` 值，错误类型为 `Never`。

每次对象的 `@Published` 变量之一发生更改时，都会让 `objectWillChange` 触发。不幸的是，你不知道到底是哪个属性改变了。它被设计成与 SwiftUI 一起非常好地工作，SwiftUI 将事件合并到一起以简化 UI 更新。
