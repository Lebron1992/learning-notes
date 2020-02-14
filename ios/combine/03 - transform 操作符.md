# 03 - Transform 操作符

## `collect()`

`collect` 操作符可以把 publisher 发出的一系列值收集到一个数组中。

首先来看不使用 `collect` 的例子：

```swift
["A", "B", "C", "D", "E"].publisher
    .sink(
        receiveCompletion: { print($0) }, receiveValue: { print($0) }
)
```

运行后结果如下：

```
A
B
C
D
E
finished
```

如果在调用 `sink` 前调用 `collect`：

```swift
["A", "B", "C", "D", "E"].publisher
    .collect()
    .sink(
        receiveCompletion: { print($0) }, receiveValue: { print($0) }
)
```

运行后结果如下：

```
["A", "B", "C", "D", "E"]
finished
```

另外还有其他形式的 `collect` 操作符。例如指定收集值的个数：

```swift
["A", "B", "C", "D", "E"].publisher
    .collect(2)
    .sink(
        receiveCompletion: { print($0) }, receiveValue: { print($0) }
)
```

运行后结果如下：

```
["A", "B"]
["C", "D"]
["E"]
finished
```

最后一个数组只有 `E`，因为 publisher 只剩下 `E`，然后就结束了。

## 映射

### `map(_:)`

这个 `map` 跟 Swift 标准的 `map` 非常类似，把从 publisher 接收到的值映射为另外一个值。例如：

```swift
let formatter = NumberFormatter()
formatter.numberStyle = .spellOut
[123, 4, 56].publisher
    .map {
        formatter.string(for: NSNumber(integerLiteral: $0)) ?? ""
}
    .sink(receiveValue: { print($0) })
```

运行后，结果如下：

```
one hundred twenty-three
four
fifty-six
```

### map key paths

`map` 还有三个针对于 key path 的版本：

- `map<T>(_:)`
- `map<T0, T1>(_:_:)`
- `map<T0, T1, T2>(_:_:_:)`

首先假设我们有以下代码：

```swift
public struct Coordinate {
  public let x: Int
  public let y: Int

  public init(x: Int, y: Int) {
    self.x = x
    self.y = y
  }
}

public func quadrantOf(x: Int, y: Int) -> String {
  var quadrant = ""

  switch (x, y) {
  case (1..., 1...):
    quadrant = "1"
  case (..<0, 1...):
    quadrant = "2"
  case (..<0, ..<0):
    quadrant = "3"
  case (1..., ..<0):
    quadrant = "4"
  default:
    quadrant = "boundary"
  }

  return quadrant
}
```

`Coordinate` 代表坐标；`quadrantOf` 方法根据传入的 `x` 和 `y` 返回 `(x, y)` 处于第几象限。

然后我们来看 `map<T0, T1>(_:_:)` 的例子：

```swift
let publisher = PassthroughSubject<Coordinate, Never>()
publisher
    .map(\.x, \.y)
    .sink(receiveValue: { x, y in
        print(
            "The coordinate at (\(x), \(y)) is in quadrant",
            quadrantOf(x: x, y: y)
        )
    })
publisher.send(Coordinate(x: 10, y: -8))
publisher.send(Coordinate(x: 0, y: 5))
```

这里的 `map` 方法把 `Coordinate` 映射成他的两个属性 `x` 和 `y`。

另外两个类型的 `map` 方法用法类似。

### `tryMap(_:)`

`tryMap(_:)` 可以接收会抛出错误的闭包。例如：

```swift
Just("Directory name that does not exist")
    .tryMap {
        try FileManager.default
            .contentsOfDirectory(atPath: $0)
    }
    .sink(
        receiveCompletion: { print($0) },
        receiveValue: { print($0) }
    )
```

运行代码后，`receiveCompletion` 就会接收到一个错误。

## `flatMap(maxPublishers:_:)`

这个 `flatMap` 操作符可以把多个 publishers 扁平化为一个 publisher。

我们用模拟聊天的例子来演示 `flatMap`。首先有以下辅助类型 `Chatter`：

```swift
public struct Chatter {
    public let name: String
    public let message: CurrentValueSubject<String, Never>

    public init(name: String, message: String) {
        self.name = name
        self.message = CurrentValueSubject(message)
    }
}
```

其中的 `message` 属性是一个 `CurrentValueSubject`。

然后在 Playground 添加以下代码：

```swift
let charlotte = Chatter(name: "Charlotte", message: "Hi, I'm Charlotte!")
let james = Chatter(name: "James", message: "Hi, I'm James!")

let chat = CurrentValueSubject<Chatter, Never>(charlotte)
chat.sink(receiveValue: { print($0.message.value) })
```

运行后得到 `Hi, I'm Charlotte!`，这个很好理解，因为 `CurrentValueSubject` 在有 subscriber 订阅时就会发送当前的值。

继续添加以下代码：

```swift
charlotte.message.value = "Charlotte: How's it going?"
chat.value = james
```

运行后结果如下：

```
Hi, I'm Charlotte!
Hi, I'm James!
```

这里我们没有看到 charlotte 的新消息 `"Charlotte: How's it going?"`，因为我们使用 `sink` 订阅的是 `Chatter` 对象，而不是 `Chatter` 对象的 `message`。

为了当 charlotte 和 james 改变 `message` 时，我们能接收到，我们可以利用 `flatMap` 把：

```swift
chat.sink(receiveValue: { print($0.message.value) })
```

改为：

```swift
chat
    .flatMap { $0.message }
    .sink(receiveValue: { print($0) })
```

这时在运行代码就能看到：

```
Hi, I'm Charlotte!
Charlotte: How's it going?
Hi, I'm James!
```

再继续添加代码：

```swift
james.message.value = "James: Doing great. You?"
charlotte.message.value = "Charlotte: I'm doing fine thanks."
```

运行后可以看到，他们的对话都打印出来了：

```
Hi, I'm Charlotte!
Charlotte: How's it going?
Hi, I'm James!
James: Doing great. You?
Charlotte: I'm doing fine thanks.
```

这就是 `flatMap` 给我带来的好处，它把它所有接收到的 `Chatter` 类型 publisher 都扁平化为 `String` 类型的 publisher。另外，这有可能导致内存问题，因为它会缓存所有接收到的 publishers 来更新扁平化后的 publisher。

为了管理内存问题，`flatMap` 方法还可以传入另外一个参数。我们把上面演示的例子中的：

```swift
.flatMap { $0.message }
```

改为：

```swift
.flatMap(maxPublishers: .max(2)) { $0.message }
```

这意味着 `chat` 最多只能接收两个 publishers。如果我们在创建一个 `Chatter`：

```swift
let morgan = Chatter(name: "Morgan",
                     message: "Hey guys, what are you up to?")
chat.value = morgan
charlotte.message.value = "Did you hear something?"
```

运行后发现我们并没有看到 morgan 说的 `"Hey guys, what are you up to?"`，但是看到了 charlotte 说的 `"Did you hear something?"`。因为我们设置 `maxPublishers` 为 `.max(2)`，第三个人 morgan 已经不能被 `flatMap` 接收了。

## 替换

### `replaceNil(with:)`

`replaceNil(with:)` 可以把 publisher 中的 `nil` 替换成指定的值。例如：

```swift
["A", nil, "C"].publisher
    .replaceNil(with: "-")
    .sink(receiveValue: { print($0) })
```

把 `nil` 替换为 `-`。运行之后结果如下：

```
Optional("A")
Optional("-")
Optional("C")
```

我发现输出的值还是 `Optional` 类型。如果想变成非 Optional 类型，则可以使用 `map` 进行强转：

```swift
["A", nil, "C"].publisher
    .replaceNil(with: "-")
    .map { $0! }
    .sink(receiveValue: { print($0) })
```

### `replaceEmpty(with:)`

如果一个 publisher 在结束之前没有发出任何值，那么可以使用这个操作符在 publisher 发出结束事件之前替换或者插入值。

例如：

```swift
let empty = Empty<Int, Never>()
empty
    .sink(receiveCompletion: { print($0) },
          receiveValue: { print($0) })
```

运行之后，只看到输出 `finished`。

下面是我们使用 `replaceEmpty(with:)`：

```swift
let empty = Empty<Int, Never>()
empty
    .replaceEmpty(with: 1)
    .sink(receiveCompletion: { print($0) },
          receiveValue: { print($0) })
```

运行后结果如下：

```
1
finished
```

可以看到在结束事件之前发出了一个 `1`。

## 增量转换

### `scan(_:_:)`

这个操作符的第一个参数是初始值；第二参数是接受两个参数的闭包，第一个是闭包最后一次返回的值，第二个是 publisher 当前发出的值。

我们来看下面的演示图，`scan` 的初始值是 `0`，闭包的实现是两个参数相加：

```
------ 1 ------ 2 ------ 3 ------>
       ⬇️       ⬇️       ⬇️
  |--------------------------|
  |  scan(0) { $0 + $1 }     |
  |--------------------------|
       ⬇️       ⬇️       ⬇️
------ 1 ------ 3 ------ 6 ------>
```

上图演示的逻辑用代码编写如下：

```
[1, 2, 3].publisher
    .scan(0) { $0 + $1 }
    .sink(receiveValue: { print($0) })
```

运行结果：

```
1
3
6
```

### `tryScan(_:_:)`

`tryScan(_:_:)` 和上面的 `scan(_:_:)` 非常类似，唯一不同的是第二参数闭包可以抛出错误。

例如：

```swift
[1, 2, 3].publisher
    .tryScan(0) {
        let result = $0 + $1
        if result == 3 {
            throw NSError(
                domain: "example.com",
                code: -1,
                userInfo: ["key": "test error"]
            )
        }
        return result
}
.sink(receiveCompletion: {
    print("completion: \($0)")
}, receiveValue: {
    print("value: \($0)")
})
```

运行结果：

```
value: 1
completion: failure(Error Domain=example.com Code=-1 "(null)" UserInfo={key=test error})
```
