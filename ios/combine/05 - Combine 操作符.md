> 本文是阅读 [Combine: Asynchronous Programming with Swift](https://store.raywenderlich.com/products/combine-asynchronous-programming-with-swift) 后的学习笔记，有需要的请点击链接购买。

# 05 - Combine 操作符

## `prepend(_:)`

`prepend` 可以用于在一个 publisher 之前添加数据。它有多个重载方法，传入的参数可以是某个类型的可变参数、Sequence 和 Publisher。

### 传入可变参数

代码演示如下：

```swift
let publisher = [3, 4].publisher
publisher
    .prepend(1, 2)
    .sink(receiveValue: { print($0) })
```

运行结果：

```
1
2
3
4
```

在 `3` 和 `4` 前面添加 `1` 和 `2`。因为这里的 `prepend` 接收的参数是可变参数，所以我们可以传入任意多个同类型的值。

### 传入 Sequence

跟上面的传入可变参数非常类似。例如：

```swift
let publisher = [5, 6, 7].publisher
publisher
    .prepend([3, 4])
    .prepend(Set(1...2))
    .sink(receiveValue: { print($0) })
```

运行结果：

```
1
2
3
4
5
6
7
```

这里要注意的是 `Set` 类型是无序的，所以 `1` 和 `2` 的输出可能是 `12` 或者 `21`。

### 传入 Publisher

除了传入具体的值，还可以传入一个同类型的 Publisher。

例如：

```swift
let publisher1 = [3, 4].publisher
let publisher2 = [1, 2].publisher
publisher1
    .prepend(publisher2)
    .sink(receiveValue: { print($0) })
```

运行结果：

```
1
2
3
4
```

`publisher2` 的值先于 `publisher1` 输出。

但是对于传入的 Publisher，我们需要注意的是：只有当被传入 `prepend` 操作符的 Publisher 发出结束事件之后，上面的 publisher 才能发送值。

我们来看下面的例子：

```swift
let publisher1 = [3, 4].publisher
let publisher2 = PassthroughSubject<Int, Never>()

publisher1
    .prepend(publisher2)
    .sink(receiveValue: { print($0) })

publisher2.send(1)
publisher2.send(2)
```

运行结果：

```
1
2
```

运行后我们只看到 `1` 和 `2` 输出，原因是 `publisher2` 还没有结束，所以 `publisher1` 的值不能发出。

如果在代码的最后加上：

```swift
publisher2.send(completion: .finished)
```

运行结果：

```
1
2
3
4
```

## `append(_:)`

`append` 可以用于在一个 publisher 之后添加数据。它有多个重载方法，传入的参数可以是某个类型的可变参数、Sequence 和 Publisher。跟 `prepend` 类似，只是添加值的位置不一样。

### 传入可变参数

代码演示如下：

```swift
let publisher = [1].publisher
publisher
    .append(2, 3)
    .append(4)
    .sink(receiveValue: { print($0) })
```

运行结果：

```
1
2
3
4
```

append 要等前面的 publisher 结束之后才能添加值。

例如下面的代码中 `append` 的值不会输出，因为 `publisher` 没有结束：

```swift
let publisher = PassthroughSubject<Int, Never>()

publisher
    .append(3, 4)
    .append(5)
    .sink(receiveValue: { print($0) })

publisher.send(1)
publisher.send(2)
```

加上 `publisher.send(completion: .finished)` 之后，就可以看到：

```
1
2
3
4
5
```

### 传入 Sequence

跟上面的传入可变参数非常类似。例如：

```swift
let publisher = [1, 2, 3].publisher
publisher
    .append([4, 5]) // Array
    .append(Set([6, 7])) // Set，6、7 的输出顺序不确定
    .append(stride(from: 8, to: 11, by: 2)) // Stride
    .sink(receiveValue: { print($0) })
```

运行结果：

```
1
2
3
4
5
7
6
8
10
```

### 传入 Publisher

除了传入具体的值，还可以传入一个同类型的 Publisher。

例如：

```swift
let publisher1 = [1, 2].publisher
let publisher2 = [3, 4].publisher

publisher1
    .append(publisher2)
    .sink(receiveValue: { print($0) })
```

运行结果：

```
1
2
3
4
```

## 更高级的 Combine 操作符

### `switchToLatest`

这个操作符比较复杂，但是非常有用。它允许你在取消挂起的 publisher 订阅的同时，动态切换整个 publisher 订阅，从而切换到最新的 publisher 订阅。并且只能在发出的值为 Publisher 的 Publisher 上使用，例如：`Publisher<Publisher<Int, Error>, Error>`。

下面画个图来演示一下：

```
Publisher<Publisher<Int, Error>, Error>

             |--------|
             |numbers2|
             |4️⃣ 5️⃣ 6️⃣|
             |--------|
        |--------| |--------|
        |numbers1| |numbers3|
        |1️⃣ 2️⃣ 3️⃣| |7️⃣ 8️⃣ 9️⃣|
        |--------| |--------|
-----------------------------------------> 时间线
                 ⬇️
       |----------------------|
       |   switchToLatest()   |
       |----------------------|
                 ⬇️
---------1️⃣-2️⃣-4️⃣-5️⃣-7️⃣-8️⃣-9️⃣------------> subscriber
              |     |
              |     |
        numbers1    numbers2
      cancelled     cancelled
```

简单解释一下上图：1）`numbers1`、`numbers2` 和 `numbers3` 都是被发出来的 publisher；2）`numbers1` 发出了 `1` 和 `2` 之后被取消，然后切换到了 `numbers2`，后面发出的 `3` subscriber 不会收到；3）`numbers2` 发出了 `4` 和 `5` 之后被取消，然后切换到了 `numbers3`，后面发出的 `6` subscriber 不会收到；4）`numbers3` 发出了 7`、`8`和`9`。

下面用代码演示上图：

```swift
// 创建 3 个 publisher
let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<Int, Never>()
let publisher3 = PassthroughSubject<Int, Never>()

// 创建能发送 publisher 的 publisher
let publishers = PassthroughSubject<PassthroughSubject<Int,
    Never>, Never>()

// 使用 switchToLatest
publishers
    .switchToLatest()
    .sink(receiveCompletion: { _ in print("Completed!") },
          receiveValue: { print($0) }
)

publishers.send(publisher1)
publisher1.send(1)
publisher1.send(2)

publishers.send(publisher2) // publishers 发送 publisher2，publisher1 已经被取消
publisher1.send(3) // publisher1 发送的 3，subscriber 不会收到
publisher2.send(4)
publisher2.send(5)

publishers.send(publisher3) // publishers 发送 publisher3，publisher2 已经被取消
publisher2.send(6) // publisher2 发送的 6，subscriber 不会收到
publisher3.send(7)
publisher3.send(8)
publisher3.send(9)

publisher3.send(completion: .finished)
publishers.send(completion: .finished)
```

运行后结果如下：

```
1
2
4
5
7
8
9
Completed!
```

可以看到 `3` 和 `6`，没有输出。

下面我们举一个例子来体会一下 `switchToLatest()` 的好处：用户点击一个 button 来触发一个网络请求，点击了一次 button 之后，网络请求被触发，然后马上又点击了一次 button，第二个网络请求被触发，这时我们应该取消第一个网络请求，只取第二个请求的值。`switchToLatest()` 就可以很好地解决这个问题。

我们编写模拟代码如下：

```swift
// 因为网络请求是异步的，所以要定义一个集合把订阅存起来
var subscriptions = Set<AnyCancellable>()

// 图片 URL
let url = URL(string: "https://source.unsplash.com/random")!

// 根据 URL 下载图片的方法
func getImage() -> AnyPublisher<UIImage?, Never> {
    return URLSession.shared
        .dataTaskPublisher(for: url)
        .map { data, _ in UIImage(data: data) }
        .print("image")
        .replaceError(with: nil)
        .eraseToAnyPublisher()
        .store(in: &subscriptions) // 异步的订阅，要保存起来
}

// 模拟用户点击
let taps = PassthroughSubject<Void, Never>()
taps
    .map { _ in getImage() } // 每点击一次，就去下载图片
    .switchToLatest() // 使用 switchToLatest，保证只有一个 publisher 发送值，之前的 publisher 会被取消
    .sink(receiveValue: { _ in })

// 用户第一次点击
taps.send()

// 3s 后，用户连续点击两次
DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
    taps.send()
}
DispatchQueue.main.asyncAfter(deadline: .now() + 3.1) {
    taps.send()
}
```

运行后结果如下：

```
image: receive subscription: (DataTaskPublisher)
image: request unlimited
image: receive value: (Optional(<UIImage:0x600000a86400 anonymous {1080, 720}>))
image: receive finished
image: receive subscription: (DataTaskPublisher)
image: request unlimited
image: receive cancel
image: receive subscription: (DataTaskPublisher)
image: request unlimited
image: receive value: (Optional(<UIImage:0x600000a810e0 anonymous {1080, 1620}>))
image: receive finished
```

这个结果是假设第一次点击后，3s 内能把图片下载完成，然后用户才进行连续点击两次。从结果可以看到，用户总共点击了 3 次，但是只获取到了两张图片，因为两次连续点击中的第一次被 `switchToLatest()` 取消，从输出可以看到 `image: receive cancel`。

### `merge(with:)`

这个操作符比较好理解，它是把两个 publishers 发出的值根据发出的时间顺序合并在一起。

演示图如下：

```
publisher2 -------------------3️⃣--------------5️⃣-------->
           ----1️⃣-----2️⃣--------------4️⃣---------------->
                |-----------------------------|
                |   merge(with: publisher2)   |
                |-----------------------------|
                               ⬇️
           ----1️⃣-----2️⃣------3️⃣------4️⃣------5️⃣-------->
```

编写代码如下：

```swift
let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<Int, Never>()

publisher1
    .merge(with: publisher2)
    .sink(receiveCompletion: { _ in print("Completed") },
          receiveValue: { print($0) }
)

publisher1.send(1)
publisher1.send(2)

publisher2.send(3)

publisher1.send(4)

publisher2.send(5)

publisher1.send(completion: .finished)
publisher2.send(completion: .finished)
```

运行后结果如下：

```
1
2
3
4
5
Completed
```

### `combineLatest`

这个操作可以用来结合不同类型的 publisher，非常有用，结合之后发出一个 tuple 类型的值。

演示图如下：

```
publisher2 ----------"a"--------"b"----------------"c"-------->
           --1-----2----------------------3------------------->
                |-----------------------------|
                |  combineLatest(publisher2)  |
                |-----------------------------|
                               ⬇️
           --------(2, "a")---(2, "b")--(3, "b")---(3, c")---->
```

编写代码如下：

```swift
let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<String, Never>()

publisher1
    .combineLatest(publisher2)
    .sink(receiveCompletion: { _ in print("Completed") },
          receiveValue: { print("P1: \($0), P2: \($1)") }
)

publisher1.send(1)
publisher1.send(2)

publisher2.send("a")
publisher2.send("b")

publisher1.send(3)

publisher2.send("c")

publisher1.send(completion: .finished)
publisher2.send(completion: .finished)
```

运行后结果如下：

```
P1: 2, P2: a
P1: 2, P2: b
P1: 3, P2: b
P1: 3, P2: c
Completed
```

### `zip`

这个操作符跟 `combineLatest` 有点类似，都是结合两个 publishers 发出的值，然后发出 tuple 类型的值。不同的是，`zip` 只结合索引相同的值。

演示图如下：

```
publisher2 ----------"a"--------"b"-------------"c"------"d"-->
           --1-----2----------------------3------------------->
                     |--------------------|
                     |  zip(publisher2)   |
                     |--------------------|
                               ⬇️
           --------(1, "a")---(2, "b")--------(3, c")--------->
```

编写代码如下：

```swift
let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<String, Never>()

publisher1 .zip(publisher2)
    .sink(receiveCompletion: { _ in print("Completed") }, receiveValue: { print("P1: \($0), P2: \($1)") }
)

publisher1.send(1)
publisher1.send(2)
publisher2.send("a")
publisher2.send("b")
publisher1.send(3)
publisher2.send("c")
publisher2.send("d")

publisher1.send(completion: .finished)
publisher2.send(completion: .finished)
```

运行后结果如下：

```
P1: 1, P2: a
P1: 2, P2: b
P1: 3, P2: c
Completed
```
