> 本文是阅读 [Combine: Asynchronous Programming with Swift](https://store.raywenderlich.com/products/combine-asynchronous-programming-with-swift) 后的学习笔记，有需要的请点击链接购买。

# 02 - Publishers 和 Subscribers

## 你好，Publisher

`Publisher` 协议是 Combine 的核心。此协议定义了一个类型能够随着时间的推移向一个或多个订阅者传输一系列值的要求。

如果你一起开发过 Apple 平台的项目，也许会把 `NotificationCenter` 看做是一个 `Publisher`。而事实上， `NotificationCenter` 确实有一个方法 `publisher(for:object:)` 可以创建一个 `Publisher` 类型。例如：

```swift
let myNotification = Notification.Name("myNotification")
let publisher = NotificationCenter.default
  .publisher(for: myNotification, object: nil)
```

publisher 可以发出两种类型的事件：1) 某种类型的值；2) 结束事件。

publisher 可以不发出值或者发出多个值，但是只能有一个结束事件，结束事件可以是一个正常的结束或者错误。一旦 publisher 发出了结束事件，publisher 的生命周期就结束了，不能再发出任何值和事件。

在更深入探讨 publisher 和 subscriber 时，我们接着继续看传统的 `NotificationCenter` 的 api：

```swift
let center = NotificationCenter.default
let observer = center.addObserver(
    forName: myNotification,
    object: nil,
    queue: nil) { notification in
        print("Notification received!")
}
center.post(name: myNotification, object: nil)
center.removeObserver(observer)
```

运行这段代码我们可以看到输出：`Notification received!`

上面这个例子中，这个输出并不是来源于 publisher。要想让输出来自 publisher，我们还需要 subscriber。

## 你好，Subscriber

`Subscriber` 协议定义了一个类型能够从 Publisher 接收输入的要求。

我们来看下面一个例子：

```swift
let myNotification = Notification.Name("MyNotification")
let publisher = NotificationCenter.default
    .publisher(for: myNotification, object: nil)
let center = NotificationCenter.default
```

如果我们现在发出一个通知，publisher 不会马上发出转发这个通知。publisher 只有在至少有一个 subscriber 时，publisher 才会发出事件。

### 使用 `sink(_:_:)` 订阅

使用 `sink(_:_:)` 订阅 publisher 的代码如下：

```swift
let subscription = publisher.sink { _ in
    print("Notification received from a publisher!")
}
center.post(name: myNotification, object: nil)
subscription.cancel() // 取消订阅
```

调用 `cancel` 之后，闭包里面的打印就不会再输出`sink` ；如果不调用 `cancel`，闭包就会不断的收到 publisher 发出的通知。

`sink` 还有另外一个重载方法，接收两个闭包（一个处理结束事件，另外一个处理值事件）。例如：

```swift
let just = Just("Hello world!")
_ = just
    .sink(
        receiveCompletion: {
            print("Received completion", $0)
    },
        receiveValue: {
            print("Received value", $0)
    })
```

运行这段代码，可以看到两行打印：

```
Received value Hello world!
Received completion finished
```

这个例子中的 `sink` 方法接收两个参数，另外还利用 `Just` 创建了一个 publisher。`Just` 是一个对每个订阅者只发出一个值事件然后马上发出一个结束事件的 Publisher，这也就时为什么我们能看到那两行打印的原因。

### 使用 `assign(to:on:)` 订阅

使用 `assign(to:on:)`，我们可以直接把接收到的值直接赋值给支持 KVO 的对象的某个属性。例如：

```swift
class SomeObject {
    var value: String = "" {
        didSet {
            print(value)
        }
    }
}

let object = SomeObject()
let publisher = ["Hello", "world!"].publisher
publisher.assign(to: \.value, on: object)
```

运行代码可以看到输出：

```
Hello
world!
```

## 你好，Cancellable

当一个订阅结束，并且不想再从 publisher 那里接收任何事件，我们需要取消订阅以释放资源或者停止对应的活动继续进行，例如网络请求。

在订阅 publisher 时会返回一个 `AnyCancellable` 实例，当我们需要时，可以利用这个实例来取消订阅。`AnyCancellable` 实现了 `Cancellable` 协议，这个协议有一个 `cancel()` 方法。

在 **使用 `sink(_:_:)` 订阅** 这部分内容中，我们已经接触了 `cancel()` 方法。代码如下：

```swift
let subscription = publisher.sink { _ in
    print("Notification received from a publisher!")
}
center.post(name: myNotification, object: nil)
subscription.cancel() // 取消订阅
```

如果没有调用 `cancel()` 方法，订阅会一直持续，除非 publisher 发出了结束事件，或者正常的内存管理把这个订阅取消。

## 订阅过程的理解

下面我们通过一张图来理解下整个订阅过程：

```
 -------------                        --------------
|  Publisher  |                      |  Subscriber  |
 -------------                        --------------
      |                                      |
      |                                      |
      | <<<-------- 订阅 publisher --------- 1️⃣
      |                                      |
      |                                      |
      2️⃣------------ 返回订阅对象 --------->>> |
      |                                      |
      |                                      |
      | <<< --------- 请求值事件 ------------ 3️⃣
      |                                      |
      |                                      |
      4️⃣------------- 发送值事件 ---------->>> |
      |                                      |
      |                                      |
      5️⃣------------ 发送结束事件 --------->>> |
      |                                      |
      |                                      |
      ⬇️                                     ⬇️
```

1. subscriber 向 publisher 订阅。
2. publisher 创建一个订阅对象，然后返回给 subscriber。
3. subscriber 向 publisher 请求数据。
4. publisher 发送值事件。
5. publisher 发送结束事件。

下面我们来看一下 `Publisher` 协议的 API 和它最重要的扩展方法：

```swift
public protocol Publisher {
    // 1
    associatedtype Output

    // 2
    associatedtype Failure : Error

    // 4
    func receive<S>(subscriber: S)
        where S: Subscriber,
        Self.Failure == S.Failure,
        Self.Output == S.Input
}
extension Publisher {
    // 3
    public func subscribe<S>(_ subscriber: S)
        where S : Subscriber,
        Self.Failure == S.Failure,
        Self.Output == S.Input
}
```

代码解读：

1. `Output` 是 publisher 可以发送的值的类型。
2. `Failure` 是 publisher 有可能发送的错误类型。如果 publisher 不会发出错误，那么 `Failure` 类型就是 `Never`。
3. subscriber 可以通过调用 publisher 的 `subscribe(_:)` 方法来订阅 publisher。
4. `subscribe(_:)` 方法的内部实现将会调用 `receive(subscriber:)` 方法来把 subscriber 附着在 publisher 上。

另外，`Subscriber` 的 `Input` 和 `Failure` 类型必须和 `Publisher` 的 `Output` 和 `Failure`。

接着我们看一下 `Subscriber` 协议的 API:

```swift
public protocol Subscriber: CustomCombineIdentifierConvertible {
    // 1
    associatedtype Input
    // 2
    associatedtype Failure: Error
    // 3
    func receive(subscription: Subscription)
    // 4
    func receive(_ input: Self.Input) -> Subscribers.Demand
    // 5
    func receive(completion: Subscribers.Completion<Self.Failure>)
}
```

代码解读：

1. `Input` 是 subscriber 能接收的值的类型。
2. `Failure` 是 subscriber 能接收的错误类型。如果 subscriber 不接收错误，那么 `Failure` 类型就是 `Never`。
3. publisher 会在 subscriber 上调用 `receive(subscription:)` 方法来把创建好的订阅对象返回给 subscriber。
4. publisher 会在 subscriber 上调用 `receive(_:)` 方法来告诉 subscriber 它刚刚发出的值。
5. publisher 会在 subscriber 上调用 `receive(completion:)` 方法来告诉 subscriber 它刚刚发出的结束事件。

publisher 和 subscriber 之间的联系是通过 subscription 来实现的。下面是 `Subscription` 协议的代码：

```swift
public protocol Subscription: Cancellable, CustomCombineIdentifierConvertible {
    func request(_ demand: Subscribers.Demand)
}
```

subscriber 会在 subscription 上调用 `request(_:)` 方法来表示它将会接收多个值，直到达到设定的能接收的值的最大数量，或者是无限接收。

> subscriber 可以设定能接收的值的最大数量，这个概念叫做 **背压管理** (backpressure management)。如果没有这个概念或者其他策略，subscriber 会被从 publisher 发出的很多值所淹没，这会导致一些问题。后面会具体讲解这个问题。

在 `Subscriber` 的 API 中，`receive(_:)` 返回的是一个 `Demand` 类型，即使 subscriber 能接受的值的最大数量是在 `receive(_:)` 方法中第一次调用 `subscription.request(_:)` 时设置的，但是我们也可以在每次接收值的时候调整那个最大数量。

> 在 `Subscriber.receive(_:)` 方法中修改能接受的值的最大数量时，它是在原来的基础上附加的，也就是新的最大值加上当前的最大值。而且新的最大值必须是正数，不能是负数。也就是说你只能增大那个最大值，而不能减小它。

## 创建自定义的 subscriber

我们来看下面的例子：

```swift
let publisher = (1...6).publisher

final class IntSubscriber: Subscriber {
    typealias Input = Int
    typealias Failure = Never

    func receive(subscription: Subscription) {
        subscription.request(.max(3))
    }

    func receive(_ input: Int) -> Subscribers.Demand {
        print("Received value", input)
        return .none
    }

    func receive(completion: Subscribers.Completion<Never>) {
        print("Received completion", completion)
    }
}
```

1. 首先定义了一个整型 publisher。
2. 定义了一个自定义的 `IntSubscriber`：
   - 1）能接收 `Int` 类型的值；
   - 2）`Failure` 类型是 `Never` 表示不接收错误；
   - 3）`receive(subscription:)` 方法将会被 publisher 内部调用，在方法的实现中，通过调用 `request(_:)` 设置 subscriber 能接收的值的最大数量；
   - 4）`receive(_:)` 方法的实现中，打印 `input`，并且返回 `.none`，意味着 subscriber 不会对 `Demand` 做出调整，等同于返回 `.max(0)`；
   - 5）`receive(completion:)` 方法实现中打印 `completion`。

下面创建 `IntSubscriber` 实例，并使用它：

```swift
let subscriber = IntSubscriber()
publisher.subscribe(subscriber)
```

运行代码看到下面的输出：

```
Received value 1
Received value 2
Received value 3
```

从这个打印结果可以看到，我们并没有收到结束事件，因为我们在 `receive(subscription:)` 方法中设置了值的最大的个数为 `.max(3)`

如果修改 `receive(_:)` 方法的实现如下：

```swift
func receive(_ input: Int) -> Subscribers.Demand {
    print("Received value", input)
    return .unlimited
}
```

运行代码将会看到以下输出：

```
Received value 1
Received value 2
Received value 3
Received value 4
Received value 5
Received value 6
Received completion finished
```

如果把 `.unlimited` 改为 `.max(1)`，得到的结果也是一样，因为每接收到一个值，subscriber 能接受的值的最大的个数就会在原有基础上加 `1`。

## 你好，Future

在前面我们用到了 `Just`，它是一个对每个订阅者只发出一个值事件然后马上发出一个结束事件的 Publisher。`Future` 跟 `Just` 很类似，它也是一个 `Publisher`，但它是异步地发出一个值事件，然后马上发出一个结束事件。

我们来看下面的例子：

```swift
func futureIncrement(
    integer: Int,
    afterDelay delay: TimeInterval
) -> Future<Int, Never> {
    Future<Int, Never> { promise in
        DispatchQueue.global()
            .asyncAfter(deadline: .now() + delay) {
                promise(.success(integer + 1)) }
    }
}
```

这里定义了一个方法，返回 `Future<Int, Never>`，也就是这个 publisher 会发出 `Int` 类型的值，并且永远不会发出错误的结束事件。点进去 `Future` 的源码可以看到，这个方法实现创建 `Future` 实例的闭包里面的 `promise` 其实是 `(Result<Output, Failure>) -> Void` 类型，别名是 `Promise`。

我们使用一下这个方法:

```swift
var subscriptions = Set<AnyCancellable>()
let future = futureIncrement(integer: 1, afterDelay: 3)
future
    .sink(
        receiveCompletion: { print($0) },
        receiveValue: { print($0) }
)
    .store(in: &subscriptions)
```

1. 首先创建一个集合用于存储订阅对象。对于一些长时间运行的的异步操作，我们必须把它们存起来，否则当所在的作用域的代码执行完成后，订阅就会被取消。
2. 调用那个方法创建一个 `future`。
3. 使用 `sink` 方法订阅 `future`，调用 `store` 把订阅对象存储在 `subscriptions` 集合中。

执行上面的代码，等待 3 秒后，就会得到以下输出：

```
2
finished
```

## 你好，Subject

Subject 可以作为一个中间人，把不是 Combine 的命令式代码像 Combine 的 subscriber 发送值。

### PassthroughSubject

首先自定义一个 `StringSubscriber`：

```swift
enum MyError: Error {
    case test
}

final class StringSubscriber: Subscriber {
    typealias Input = String
    typealias Failure = MyError

    func receive(subscription: Subscription) {
        subscription.request(.max(2))
    }

    func receive(_ input: String) -> Subscribers.Demand {
        print("Received value", input)
        return input == "World" ? .max(1) : .none
    }

    func receive(completion: Subscribers.Completion<MyError>) {
        print("Received completion", completion)
    }
}
```

然后添加如下代码：

```swift
let subscriber = StringSubscriber()
let subject = PassthroughSubject<String, MyError>()
subject.subscribe(subscriber)
let subscription = subject
    .sink(
        receiveCompletion: { completion in
            print("Received completion (sink)", completion)
    },
        receiveValue: { value in
            print("Received value (sink)", value)
    }
)
```

1. 定义一个 `StringSubscriber` 实例
2. 定义一个 `PassthroughSubject` 实例，值类型和错误类型与 `StringSubscriber` 一一对应，并且调用 `subscribe` 去订阅 subject
3. 再用 `sink` 的方式订阅 subject

使用 `send` 方法发送值：

```swift
subject.send("Hello")
subject.send("World")
```

执行代码后，看到如下输出：

```
Received value Hello
Received value (sink) Hello
Received value World
Received value (sink) World
```

再继续添加以下代码：

```swift
subscription.cancel()
subject.send("Still there?")
```

这里把 `subscription` 取消，然后在发送另外一个值。执行代码结果如下：

```
Received value Hello
Received value (sink) Hello
Received value World
Received value (sink) World
Received value Still there?
```

`subscription` 被 `cancel()` 后，只有 `subscriber` 还能继续接收值。

再继续添加以下代码：

```swift
subject.send(completion: .finished)
subject.send("How about another one?")
```

执行代码结果如下：

```
Received value Hello
Received value (sink) Hello
Received value World
Received value (sink) World
Received value Still there?
Received completion finished
```

两个订阅都不能再收到 `How about another one?`，因为 `subject` 已经发出了 `.finished` 结束事件，后面再发送的值 subscriber 是收不到的。

通过 `PassthroughSubject` 来发送值，是把命令式代码桥接到声明式代码的一个方式。

### CurrentValueSubject

`CurrentValueSubject` 允许我们在初始化的时候设置一个初始值，并且有 subscriber 向它订阅时，它会马上发送这个初始值给 subscriber。

我们来看一下例子：

```swift
let subject = CurrentValueSubject<Int, Never>(0)
subject
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
```

这里实例化了一个 `CurrentValueSubject` 对象，并设置初始值为 `0`；`subscriptions` 是之前创建的一个集合。执行这段代码后，可以看到输出 `0`。但是我们没有调用 `send` 方法去发送值，所以可以证明有 subscriber 向它订阅时，它会马上发送这个初始值给 subscriber。

使用 `send` 方法发送值：

```
 subject.send(1)
 subject.send(2)
```

执行代码后，我们就会看到：

```
0
1
2
```

我们还可以它的 `value` 属性，它代表当前的值：

```swift
print(subject.value) // 2
```

除了使用 `send` 方法可以发送值，也可以直接对 `value` 赋值来发送值：

```swift
subject.value = 3
```

通过这种方法，subscriber 也可以收到 `3`。

但是我们不能通过 `subject.value = .finished` 去发送结束事件。只能通过：`send` 方法：

```swift
subject.send(completion: .finished)
```

另外我们可以在 `sink` 方法前调用 `print()` 方法来查看具体日志，可以方便调试。例如：

```swift
subject
    .print()
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
subject.send(1)
subject.send(2)
subject.value = 3
subject.send(completion: .finished)
```

执行代码后结果如下：

```
receive subscription: (CurrentValueSubject)
request unlimited
receive value: (0)
0
receive value: (1)
1
receive value: (2)
2
receive value: (3)
3
receive finished
```

## 类型擦除 (Type erasure)

有时候我们想让 subscriber 从 publisher 订阅事件，但是不想让它访问关于 publisher 的其他细节。我们可以用类型擦除来解决这个问题。

看下面的例子：

```swift
let subject = PassthroughSubject<Int, Never>()
let publisher = subject.eraseToAnyPublisher()
publisher
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
subject.send(0)
```

这里主要关注 `eraseToAnyPublisher()` 方法，它把 `PassthroughSubject` 转变成了 `AnyPublisher` 类型。`AnyPublisher` 遵循 `Publisher` 协议，它可以让我们隐藏 publisher 的一些不想暴露给 subscriber 的细节。另外，`AnyPublisher` 没有 `send(_:)` 方法。

有一种我们想使用 `AnyPublisher` 的例子是：当我们想使用一对 public 和 private 属性时，想让 private publisher 来发送值，然后外部环境只能访问 public publisher 来订阅值。
