> 本文是阅读 [Combine: Asynchronous Programming with Swift](https://store.raywenderlich.com/products/combine-asynchronous-programming-with-swift) 后的学习笔记，有需要的请点击链接购买。

# 自定义 Publisher 和背压 (Backpressure) 处理

## 自定义 Publisher

自定义 Publisher 的复杂性可以从简单到相当复杂，这里我们将用三种不同的方法来创建自己的 Publisher：

- 在 `Publisher` 命名空间中使用简单的扩展方法。
- 在 `Publishers` 名称空间中实现具有 `Subscription` 的能发出值的类型。
- 跟第二个相同，但 subscription 把上游 Publisher 的进行转换。

> **注意**：从技术上讲，可以在没有自定义订阅的情况下创建自定义 Publisher。如果这样做，我们将失去处理 subscriber 需求的能力，这将使我们的 publisher 在 Combine 生态系统中是非法的。提前取消订阅也可能成为一个问题。这不是一个推荐的方法。

## 使用扩展方法创建 Publisher

首先我们通过重用现有的操作符来实现一个简单的操作符。

例如下面的例子：

```swift
extension Publisher {
    func unwrap<T>() -> Publishers.CompactMap<Self, T> where Output == Optional<T> {
        compactMap { $0 }
    }
}
```

创建自定义操作符最麻烦的就是写方法的签名，我们把它分解开来看：

- `func unwrap<T>()` : 首先把操作符定义为泛型，因为它的 `Ouput` 是可选类型中解包后的类型。
- `-> Publishers.CompactMap<Self, T>` : 因为在方法的实现中使用 `compactMap(_:)`，所以我们自定义操作符的返回值类型就是它的返回值类型。通过查看 `Publishers.CompactMap` 文档，可以知道他是一个泛型：`public struct CompactMap<Upstream, Output>`。当自定义操作符时，`Upstream` 就是 `Self`(正在扩展的 publisher)，`Output` 就是解包后的类型。
- `where Output == Optional<T>` : 最后把操作符限制在 `Optional` 类型。

> **注意**：当开发更复杂的操作符作为方法时，例如当使用一系列操作符时，签名可能会很快变得非常复杂。一个好的方法是让操作符返回 `AnyPublisher<OutputType，FailureType>`。在该方法中，您将返回以 `eraseToAnyPublisher()` 结尾的 Publisher。

在学习另外两种自定义 Publisher 方法之前，先看看当我们订阅 publisher 时都发生了什么。

## 订阅机制

subscription 是 Combine 的无名英雄：当你看到到处都是 publisher 时，他们大多是无生命的实体。当订阅 publisher 时，它将实例化一个 subscription，该 subscription 负责接收来自 subscriber 的请求并生成事件（例如，值事件和结束事件）。

以下是 subscription 生命周期的详细信息 (如果下图乱了，请把窗口调宽一点)：

```
|-----------------------------------------------------------------------|
|                             Subscriber                                |
| 开始订阅       接收到subscription --> 请求值          接收到值 --> 添加需求 |  
     |                 ⬆️              |               ⬆️         |     |
|----|-----------------|---------  ----|---------------|----------|-----|
   ⓵|               ⓶|              ⓷|             ⓸|         ⓹|
     |                 |               |               |          |
     ⬇️                |               ⬇️              |          ⬇️
|------------------------------|    |------------------|----------------|
|    创建              移交     |    | 接收     开始    发出        更新    |
|subscription --> subscription |    | 需求 --> 工作 --> 值        需求    |
|                              |    |                  ⬆️         |     |
|                                   |                  |_ _ _ ⓺_ _|    |
|          Publisher           |    |         Subscription              |
|------------------------------|    |-----------------------------------|
```

图中每一步解释如下：

- 1. subscriber 向 publisher 订阅
- 2. publisher 创建 `Subscription` 对象，然后移交给 Subscriber (通过调用 `receive(subscription:)`)
- 3. subscriber 通过向 subscription 发送它想要的值的个数来请求值 (通过调用 subscription 的 `request(_:)` 方法)
- 4. subscription 开始工作并开始发出值，一个接一个发送给 subscriber（通过调用 subscriber 的 `receive(_:)` 方法）
- 5. 当接收到一个值时，subscriber 返回一个新的 `Subscribers.Demand`，添加以前的总需求。
- 6. subscription 将继续发送值，直到发送的值的个数达到请求的总数。

如果 subscription 发送的值与 subscriber 请求的值一样多，则它应在发送更多值之前等待新的请求。我们可以绕过此机制并继续发送值，但这会破坏 subscriber 和 subscription 之间的约定，并可能导致基于 Apple 定义的 publisher 树中的未定义行为。

最后，如果出现错误或 subscription 的值源头结束，则 subscription 将调用 subscriber 的 `receive(completion:)` 方法。

## 创建可以发送值的 Publisher

在这个例子中，我们通过基于 `Dispatch` 的 `DispatchSourceTimer` 来创建自己的 timer publisher。

因为这个稍微有点复杂，所以直接根据创建的不同类型放出完整代码，然后通过注释解释创建的流程。

### `DispatchTimerConfiguration`

首先我们定义一个 `DispatchTimerConfiguration`，用于更好地在 subscriber 和 subscription 之间共享一个 configuration。代码如下：

```swift
struct DispatchTimerConfiguration {

    // 指定 timer 在哪个队列执行，因为指定队列并不是必须的，所以把它定义为可选类型
    let queue: DispatchQueue?
    
    // timer 启动的时间间隔，从订阅时间开始算
    let interval: DispatchTimeInterval
    
    // 可接受的延迟时间，即系统可能延迟传递 timer 时间的截止时间之后的最大时间量
    let leeway: DispatchTimeInterval
    
    // 想要接收 timer 时间的次数。既然我们在自定义 Timer，所以让变得灵活并且能够发送有限的事件
    let times: Subscribers.Demand
}
```

### `DispatchTimer` publisher

现在我们开始创建 Publisher：

```swift
extension Publishers {
    struct DispatchTimer: Publisher {
        
        // 自定义的 Publisher 发出的值类型是 `DispatchTime`；错误类型是 `Never`
        typealias Output = DispatchTime
        typealias Failure = Never
        
        // 把 configuration 存起来，将在接收到 subscriber 时使用
        let configuration: DispatchTimerConfiguration

        init(configuration: DispatchTimerConfiguration) {
            self.configuration = configuration
        }
        
        // 实现 `Publisher` 协议的方法
        func receive<S: Subscriber>(subscriber: S) where Failure == S.Failure, Output == S.Input {
            // 接收到 subscriber 的订阅，对应订阅机制图中的第一步，开始创建 Subscription
            let subscription = DispatchTimerSubscription(
                subscriber: subscriber,
                configuration: configuration
            )

            // 对应订阅机制图中的第二步，把创建好的 Subscription 移交给 subscriber
            subscriber.receive(subscription: subscription)
        }
    }
}
```

### `DispatchTimerSubscription`

创建上一个类型中用到的 Subscription，它的作用是：

- 从 subscriber 中接收最初的请求
- 根据请求生成 timer 的事件
- 每次 subscriber 接收到值后都会累加请求的次数，然后返回新的请求
- 确保发出值的个数不超过请求的次数

下面看代码：

```swift
// 1. Subscription 不需要暴露给外部，所以定义为 `private`
// 2. 因为需要通过引用传递，所以定义为 `class`
// 3. Subscription 发出的值类型跟 Subscriber 的 `Input` 类型相同
private final class DispatchTimerSubscription<S: Subscriber>: Subscription where S.Input == DispatchTime {

    // 保存从 publisher 传过来的 configuration
    let configuration: DispatchTimerConfiguration

    // timer 发出值的总次数，从 configuration 中得到；
    // 把它作为一个计数器，每发送一个值就减 1
    var times: Subscribers.Demand

    // 保存当前 subscriber 的请求，每发送一个值就减 1
    var requested: Subscribers.Demand = .none

    // 生成 timer 事件的 DispatchSourceTimer
    var source: DispatchSourceTimer? = nil

    // subscriber，Subscription 负责持有 subscriber 直到结束或者取消；
    // 如果不持有它，它将永远不会收到值；当 subscription 从内存中移除后，所有东西都会停止
    var subscriber: S?
    
    init(subscriber: S,
         configuration: DispatchTimerConfiguration
    ) {
        self.configuration = configuration
        self.subscriber = subscriber
        self.times = configuration.times
    }
    
    // 实现 Subscription 协议的方法: 从 subscriber 中收到请求
    func request(_ demand: Subscribers.Demand) {
        // 首先判断是否已经发送足够的值给 subscriber
        guard times > .none else {
            // 已发送足够，发送 `.finished` 事件
            subscriber?.receive(completion: .finished)
            return
        }

        // 更新 subscriber 已请求的次数
        requested += demand

        // 判断 source 是否存在，如果不存在并且 subscriber 已请求的次数不为空，则创建 source
        if source == nil, requested > .none {

            // 创建 DispatchSourceTimer
            let source = DispatchSource.makeTimerSource(queue: configuration.queue)

            // 根据创建来的 configuration 计划 source；
            // 一旦它启动，就永远不会停止，即使你不是用它发送值给 subscriber。
            // 他将会一直在运行，直到 subscriber 取消 subscription，或者 subscription 从内存中移除
            source.schedule(
                deadline: .now() + configuration.interval,
                repeating: configuration.interval,
                leeway: configuration.leeway
            )
            
            // 设置 source 的 event handler，timer 启动时执行的闭包
            source.setEventHandler { [weak self] in
                // 保证 subscriber 已请求的次数不为空
                guard let self = self,
                    self.requested > .none else { return }
                
                // 因为即将发送值，所以已请求的次数和总次数都减 1
                self.requested -= .max(1)
                self.times -= .max(1)

                // 发送值给 subscriber
                _ = self.subscriber?.receive(.now())

                // 如果总的次数为空，那么发送 `.finished` 事件
                if self.times == .none {
                    self.subscriber?.receive(completion: .finished)
                }
            }
            
            // 把 source 保存起来，并激活它
            self.source = source
            source.activate()
        }
    }
    
    // 实现 Subscription 协议的方法
    func cancel() {
        source = nil // 让 DispatchSourceTimer 停止执行
        subscriber = nil // 将 subscriber 从 subscription 中释放
    }
}
```

### 定义操作符

为了方便使用刚刚创建的 Publisher，在 `Publishers` 上扩展一个操作符：

```swift
extension Publishers {
    static func timer(
        queue: DispatchQueue? = nil,
        interval: DispatchTimeInterval,
        leeway: DispatchTimeInterval = .nanoseconds(0),
        times: Subscribers.Demand = .unlimited
    ) -> Publishers.DispatchTimer {
        return Publishers.DispatchTimer(
            configuration: .init(
                queue: queue,
                interval: interval,
                leeway: leeway,
                times: times
            )
        )
    }
}
```

这样创建可以发送值的 Publisher 的流程就完成了。

## 创建可以转换值的 Publisher

在之前的文章有讲过如何分享一个 subscription：当底层的 publisher 执行一个比较耗时的操作，例如网络请求，我们想跟多个 subscribers **分享** 返回的结果，而不必每个订阅者来订阅时都去启动网络请求。

这里我们就自定义一个 `shareReplay()` 操作符来实现我们的需求。

为了编写这个操作符，我们需要自定义一个 publisher 来做下列事情：

- 在接收到第一个 subscriber 时订阅上游 publisher
- 重播最后 `N` 个值给新的 subscriber
- 如果已经发出了结束事件，则重播结束事件

为了实现 `shareReplay()`，我们需要：

- 一个遵循 `Subscription` 协议的类型，这是每个 subscriber 都会接收到的类型。为了能够处理每个 subscriber 的请求和取消，所以每个 subscriber 都要接收不同的 subscription。
- 一个遵循 `Publisher` 协议的类型，我们将会把它以 class 的形式去实现，因为它所有的 subscribers 要共享它。

### 创建 Subscription

这里我们使用 class，而不是 struct，因为 `Publisher` 和 `Subscriber` 都要访问并且修改它。

下面是详细代码：

```swift
private final class ShareReplaySubscription<Output, Failure: Error>: Subscription {
    
    // 重播缓存的最大容量
    let capacity: Int
    
    // 把 subscriber 保存起来，并且类型为 AnySubscriber；
    // 这样方便处理任何类型的 Subscriber
    var subscriber: AnySubscriber<Output,Failure>? = nil
    
    // 保存 subscriber 的请求
    var demand: Subscribers.Demand = .none
    
    // 存储准备就绪的值，直到它们被发送给 subscriber 或者被丢弃为止
    var buffer: [Output]
    
    // 存储潜在的结束事件，以准备发送给新的 subscriber
    var completion: Subscribers.Completion<Failure>? = nil
    
    // 初始化器从上游 publisher 接收相关参数并初始化自己的属性
    init<S>(
        subscriber: S,
        replay: [Output],
        capacity: Int,
        completion: Subscribers.Completion<Failure>?
    ) where S: Subscriber, Failure == S.Failure, Output == S.Input {
            self.subscriber = AnySubscriber(subscriber)
            self.buffer = replay
            self.capacity = capacity
            self.completion = completion
    }
    
    // 用于重播 completion 事件的方法
    private func complete(with completion: Subscribers.Completion<Failure>) {

        // 只有当前的 subscriber 不为 nil 时再继续往下执行
        guard let subscriber = subscriber else { return }
        
        // 因为即将发送 completion 事件，
        // 所以没有必要再持有 subscriber 和 completion，
        // 都设置为 nil
        self.subscriber = nil
        self.completion = nil
        
        // 清空 buffer
        self.buffer.removeAll()

        // 发送 completion 事件
        subscriber.receive(completion: completion)
    }
    
    // 用于重播值事件的方法
    private func emitAsNeeded() {
        // 只有当前的 subscriber 不为 nil 时再继续往下执行
        guard let subscriber = subscriber else { return }
        
        // 当请求和 buffer 都不为空时发送值
        while self.demand > .none && !buffer.isEmpty {
            // 每循环一次，请求数减 1
            self.demand -= .max(1)
            
            // 给 subscriber 发送值
            let nextDemand = subscriber.receive(buffer.removeFirst())
            
            // 如果 subscriber 返回的新请求不为空，则累加到请求中
            if nextDemand != .none {
                self.demand += nextDemand
            }
        }

        // 上面 buffer 中的数据已发送完成，如果保存的 completion 事件不为 nil，
        // 则发送给 subscriber
        if let completion = completion {
            complete(with: completion)
        }
    }
    
    // 实现 Subscription 协议的方法
    func request(_ demand: Subscribers.Demand) {
        if demand != .none {
            self.demand += demand
        }
        emitAsNeeded()
    }
    
    // 实现 Subscription 协议的方法
    func cancel() {
        complete(with: .finished)
    }
    
    // 接收上游 publisher 的值事件
    func receive(_ input: Output) {
        guard subscriber != nil else { return }
        
        // 把接收到的值保存到缓存中
        buffer.append(input)
        
        if buffer.count > capacity {
            // 如果 buffer 中的元素个数大于了设定的最大缓存容量，
            // 则移除最老的元素
            buffer.removeFirst()
        }
        
        // 发送值给 subscriber
        emitAsNeeded()
    }
    
    // 接收上游 publisher 的 completion 事件
    func receive(completion: Subscribers.Completion<Failure>) {
        guard let subscriber = subscriber else { return }
        
        // 移除 subscriber，清空 buffer
        self.subscriber = nil
        self.buffer.removeAll()
        
        // 发送 completion 事件
        subscriber.receive(completion: completion)
    }
}
```

### 编写 Publisher

在 `Publishers` 命名空间里面的 `Publisher` 通常是 struct 类型。但有些时候需要把它定义为 class 类型，例如 `multicast()` 方法返回的 `Publishers.Multicast` 类型。对于我们这个例子，也是需要使用 class 类型。

详细代码如下：

```swift
extension Publishers {
    
    // 因为想让多个 subscriber 共享一个实例，所有使用 class。
    // 并且是一个泛型，Upstream 是一个 Publisher
    final class ShareReplay<Upstream: Publisher>: Publisher {
        
        // 不改变 Output 和 Failure 的类型，而是跟上游的 Publisher 一样
        typealias Output = Upstream.Output
        typealias Failure = Upstream.Failure
        
        // 因为需要同时为多个 subscribers 服务，
        // 所以需要一个锁来保证同一时刻只有一个对象对变量进行更改
        private let lock = NSRecursiveLock()
        
        // 对上游 publisher 的引用
        private let upstream: Upstream
        
        // 重播的最大容量
        private let capacity: Int
        
        // 存储需要重播的值
        private var replay = [Output]()
        
        // 因为需要同时为多个 subscribers 服务，
        // 所以需要存储每个 subscriber 订阅时产生的 Subscription
        private var subscriptions = [ShareReplaySubscription<Output, Failure>]()
        
        // 因为操作符在发出 completion 事件后，还会重播缓存的值，
        // 所以需要记住上游的 completion 事件
        private var completion: Subscribers.Completion<Failure>? = nil
        
        init(upstream: Upstream, capacity: Int) {
            self.upstream = upstream
            self.capacity = capacity
        }
        
        private func relay(_ value: Output) {
            // 因为多个 subscribers 共享一个 publisher，当改变变量是必须用锁锁住。
            // 使用 defer 保证在方法返回之前执行解锁，防止如果不使用 defer 而在方法返回的地方忘记解锁
            lock.lock()
            defer { lock.unlock() }
            
            // 只在 completion 为 nil 的时候（上游 publisher 没有结束）才重播值
            guard completion == nil else { return }
            
            // 把值添加到缓存中，并且只保存最新的值
            replay.append(value)
            if replay.count > capacity {
                replay.removeFirst()
            }
            
            // 重播当前值给所有 subscribers
            subscriptions.forEach {
                _ = $0.receive(value)
            }
        }
        
        // 实现 Publisher 协议的方法
        func receive<S: Subscriber>(subscriber: S) where Failure == S.Failure, Output == S.Input {
            lock.lock()
            defer { lock.unlock() }
            
            // 创建  subscription
            let subscription = ShareReplaySubscription(
                subscriber: subscriber,
                replay: replay,
                capacity: capacity,
                completion: completion)
            
            // 保存 subscription
            subscriptions.append(subscription)
            
            // 把 subscription 发送给 subscriber
            subscriber.receive(subscription: subscription)
            
            // 开始订阅上游 publisher，因为这个操作需要做一次，
            // 所以接收到第一个 subscriber 时，才继续往下执行
            guard subscriptions.count == 1 else { return }
            
            let sink = AnySubscriber(
                // 当接收到 subscription 时，返回 `.unlimited` 请求，
                // 意味着无限接收值，直到上游 publisher 发出 completion 事件
                receiveSubscription: { subscription in
                    subscription.request(.unlimited)
                },
                // 当接收到值时，把它重播给 subscribers
                receiveValue: { [weak self] (value: Output) -> Subscribers.Demand in
                    self?.relay(value)
                    return .none
                },
                
                // 当接收到 completion 时，调用我们写的 complete 方法
                receiveCompletion: { [weak self] in
                    self?.complete($0)
                }
            )
            
            // 订阅上游 publisher
            upstream.subscribe(sink)
        }
        
        private func complete(_ completion: Subscribers.Completion<Failure>) {
            lock.lock()
            defer { lock.unlock() }

            // 为未来的 subscribers 保存 completion 事件
            self.completion = completion
            
            // 发送 completion 事件给所有 subscribers
            subscriptions.forEach {
                _ = $0.receive(completion: completion)
            }
        }
    }
}
```

### 添加便利地操作符

在 `Publisher` 基础上进行扩展，方便在操作符链中使用：

```swift
extension Publisher {
    func shareReplay(capacity: Int = .max) -> Publishers.ShareReplay<Self> {
        return Publishers.ShareReplay(
            upstream: self,
            capacity: capacity)
    }
}
```

至此，自定义可以转换值的 `Publisher` 就大功告成。

## 背压(backpressure)处理

在流体力学中，背压是一种阻力或与流体通过管道所需的流动方向相反的力。在 Combine 中，它是 publisher 发送值的流中的反向阻力。但这是什么阻力？通常，这是 subscriber 需要处理 publisher 发出的值的时间。例如：

- 处理高频数据，如传感器输入。
- 执行大文件传输。
- 数据更新时呈现复杂的用户界面。
- 等待用户输入。
- 更一般地说，subscriber 处理数据的速率无法跟上传入数据的速率。

Combine 提供的 publisher-subscriber 机制是灵活的。这是一个拉式(pull)设计，而不是推式(push)设计。这意味着 subscriber 向 publisher 请求值并指定要接收多少值。这种请求机制是自适应的：每次 subscriber 收到新值时，需求都会更新。这允许 subscriber 在不想接收更多数据时“关闭水龙头”，在准备接收更多数据时“打开水龙头”来应对背压。

> **注意**：请记住，只能以附加方式(只能累加，不能减少)调整需求。可以在 subscriber 每次收到新值时通过返回新的 `.max(N)` 或者 `.unlimited` 来增加需求；或者返回 `.none`，这表示需求不应增加。然而，subscriber 随后“挂起”以接收至少达到新的最大需求的值。例如，如果先前的最大需求是接收三个值，而订阅者只接收到一个值，则在 subscriber 的 `receive(_:)` 方法中返回 `.none` 不会停止订阅。当 publisher 准备好发出值时，subscriber 最多仍将接收两个值。

如果有更多的可用值，会发生什么完全取决于我们的设计。我们可以：

- 通过管理需求来控制流，以防止 publisher 发送超出处理能力的值。
- 把值缓存起来，直到可以处理它们，但是会有耗尽可用内存的风险。
- 丢弃无法立即处理的值。
- 根据您的要求，将上述各项结合起来。

除上述情况外，处理背压还可以采取以下形式：

- 具有处理拥塞的自定义 `Subscription` 的 publisher。
- 在 publishers 链末端提供值的 subscriber。

这里我们将重点实现后者，将创建 `sink` 函数的一个可暂停变种。

### 使用可暂停的 sink 来处理背压

首先定义一个可以从暂停中恢复的协议 `Pausable`:

```swift
protocol Pausable {
    var paused: Bool { get }
    func resume()
}
```

这里我们不需要 `pause()` 方法，因为我们会在接收到每个值时来决定是否暂停。

下面定义 `PausableSubscriber`，因为我们不想它的对象在使用过程中被复制，在它的生命周期中需要进行一些修改，所以要定义为 class：

```swift
final class PausableSubscriber<Input, Failure: Error>: Subscriber, Pausable, Cancellable {
    
    // Subscriber 必须有一个唯一的 id，提供给 Combine 管理和优化它的 publisher 流
    let combineIdentifier = CombineIdentifier()
    
    // 接收到值时调用的闭包，并且返回 Bool：
    // true 代表可以接收更多的值，
    // false 代表 subscription 先暂停
    let receiveValue: (Input) -> Bool
    
    // 接收到 completion 事件时调用的闭包
    let receiveCompletion: (Subscribers.Completion<Failure>) -> Void
    
    // 保存 subscription，以便随时恢复订阅
    private var subscription: Subscription? = nil
    
    // `Pausable` 协议的规定，代表当前是否已暂停
    var paused = false
    
    // 初始化函数，类似我们熟悉的 `sink` 函数；
    // 不同的是 receiveValue 有返回值
    init(receiveValue: @escaping (Input) -> Bool, receiveCompletion: @escaping
        (Subscribers.Completion<Failure>) -> Void) {
        self.receiveValue = receiveValue
        self.receiveCompletion = receiveCompletion
    }
    
    // Cancellable 协议的方法：取消订阅，并设置为 nil
    func cancel() {
        subscription?.cancel()
        subscription = nil
    }
    
    // Subscriber 协议的方法：接收到 subscription
    func receive(subscription: Subscription) {
        // 保存 subscription，方便后续使用
        self.subscription = subscription
        
        // 因为我们编写的 Subscriber 是可暂停的，所以一个一个值的请求
        subscription.request(.max(1))
    }
    
    // Subscriber 协议的方法：接收到值
    func receive(_ input: Input) -> Subscribers.Demand {
        // 调用 receiveValue 闭包，并判断是否需要暂停
        paused = receiveValue(input) == false
        
        // 如果暂停，返回空的需求；否则增加一个需求
        return paused ? .none : .max(1)
    }
    
    // Subscriber 协议的方法：接收到 completion
    func receive(completion: Subscribers.Completion<Failure>) {
        // 调用 receiveCompletion 闭包，
        // 订阅过程已经完成，把 subscription 设置为 nil
        receiveCompletion(completion)
        subscription = nil
    }
    
    // Pausable 协议的方法
    func resume() {
        guard paused else { return }
        paused = false
        // 恢复请求，增加一个需求
        subscription?.request(.max(1))
    }
}
```

下面是对 `Publisher` 进行扩展，定义 `pausableSink` 操作符：

```swift
extension Publisher {
    // 定义 `pausableSink` 方法，类似于 `sink`
    func pausableSink(
        receiveCompletion: @escaping ((Subscribers.Completion<Failure>) -> Void),
        receiveValue: @escaping ((Output) -> Bool)
    ) -> Pausable & Cancellable {

        // 创建 PausableSubscriber
        let pausable = PausableSubscriber(
            receiveValue: receiveValue,
            receiveCompletion: receiveCompletion
        )
        
        // 向 publisher 订阅
        self.subscribe(pausable)
        
        // 返回 PausableSubscriber，用于外部恢复或取消订阅
        return pausable
    }
}
```

### 测试

```swift
let subscription = [1, 2, 3, 4, 5, 6]
    .publisher
    .pausableSink(receiveCompletion: { completion in
        print("Pausable subscription completed: \(completion)")
    }) { value -> Bool in
        print("Receive value: \(value)")
        if value % 2 == 1 {
            print("Pausing")
            return false
        }
        return true
}

let timer = Timer.publish(every: 1, on: .main, in: .common)
    .autoconnect()
    .sink { _ in
        guard subscription.paused else { return }
        print("Subscription is paused, resuming")
        subscription.resume()
    }
```

运行结果：

```
Receive value: 1
Pausing
Subscription is paused, resuming
Receive value: 2
Receive value: 3
Pausing
Subscription is paused, resuming
Receive value: 4
Receive value: 5
Pausing
Subscription is paused, resuming
Receive value: 6
Pausable subscription completed: finished
```

> **注意**：如果 publisher 无法等待 subscriber 请求它们，该怎么办？在这种情况下，我们需要使用 `buffer(size:prefetch:whenFull:)` 运算符来缓存值。这个操作符可以缓存在 size 参数中指定的容量的值，并在 subscriber 准备好接收时传递给它。`prefetch` 参数确定缓冲区的填充方式（订阅时立即填充，保持缓存是满的；或根据 subscriber 的请求填充），`whenFull` 参数确定缓存已满时的处理策略（即删除接收到的最后一个值，删除最旧的值或因错误终止）。