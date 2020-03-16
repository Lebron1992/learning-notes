> 本文是阅读 [Combine: Asynchronous Programming with Swift](https://store.raywenderlich.com/products/combine-asynchronous-programming-with-swift) 后的学习笔记，有需要的请点击链接购买。

# 09 - Timers

## 使用 `RunLoop`

`RunLoop` 实现了 `Scheduler` 协议。它定义了几个相对底层的方法，和唯一一个允许您创建 cancellable timer 的方法：

```swift
let runLoop = RunLoop.main

let subscription = runLoop.schedule(
    after: runLoop.now,
    interval: .seconds(1),
    tolerance: .milliseconds(100)
) {
    print("Timer fired")
}
```

这个 timer 不会发出任何值，也不创建 publisher。它只是在指定的参数下启动，然后运行闭包里的代码。这里唯一跟 Combine 相关的是，它会返回 `subscription` 是一个 `Cancellable` 类型，让你可以随时这个 timer。

例如：

```swift
runLoop
    .schedule(
    after: .init(Date(timeIntervalSinceNow: 3))) {
        subscription.cancel()
}
```

但综合考虑，`RunLoop` 并不是创建 timer 的最佳方法。你最好用 `Timer` 类！

## 使用 `Timer` 类

你可以使用下面这种方式创建一个重复的 timer publisher：

```swift
let publisher = Timer.publish(every: 1.0, on: .main, in: .common)
```

- `on` 参数是指定 timer 的放在哪个 `RunLoop`。这里是 main RunLoop。
- `in` 参数是指定 timer 在 run loop 的哪种模式下运行。这里是 `.common` run loop 模式。

除非您了解 run loop 是如何运行的，否则应该坚持使用这些默认值。通过调用 `RunLoop.current`，您可以为自己创建的或从 Foundation 获得的任何线程获取 `RunLoop`，这样您也可以编写以下内容：

```swift
let publisher = Timer.publish(every: 1.0, on: .current, in: .common)
```

> **注意**：把这段代码放在一个 Dispatch queue 而不是 `DispatchQueue.main` 可能会导致不可预料的结果。Dispatch 框架没有使用 run loop 来管理它的线程。因为 run loop 需要调用它的其中一个 `run` 方法来处理事件。你永远不会看到除主队列以外的任何队列上的 timer 执行。因为为了你的 timer 保持安全，应该直接使用 `RunLoop.main`。

上面的 timer 返回 publisher 是一个 `ConnectablePublisher`。它是 `Publisher` 的一个变种，在你调用 `connect()` 方法之前，timer 不会启动。另外，你也可以调用 `autoconnect()` 来让它收到第一个 subscriber 订阅时就自动启动。

因此，创建在订阅时启动 timer 的 publisher 的最佳方法是写入：

```swift
let publisher = Timer
    .publish(every: 1.0, on: .main, in: .common)
    .autoconnect()
```

这样这个 timer 就会在有 subscriber 订阅时定时重复的发出当前的日期，因为它的 `Publisher.Output` 类型是 `Date`。

下面是一个例子：

```swift
let subscription = Timer
    .publish(every: 1.0, on: .main, in: .common)
    .autoconnect()
    .scan(0) { counter, _ in counter + 1 }
    .sink { counter in
        print("Counter is \(counter)")
    }
```

## 使用 `DispatchQueue`

可以使用 `DispatchQueue` 生成 timer 事件。虽然 Dispatch 框架有 `DispatchTimerSource` 事件源，但令人惊讶的是，Combine 没有为其提 timer 接口。相反，您将使用另一种方法在队列中生成 timer 事件。这可能有点复杂：

```swift

let queue = DispatchQueue.main

// 创建 Subject，用于发布 timer 的值
let source = PassthroughSubject<Int, Never>()

var counter = 0

// 计划一个重复的任务，这个任务会马上启动
let cancellable = queue.schedule(
    after: queue.now,
    interval: .seconds(1)
){
    source.send(counter)
    counter += 1
}

// 订阅 Subject
let subscription = source.sink {
    print("Timer emitted \($0)")
}
```
