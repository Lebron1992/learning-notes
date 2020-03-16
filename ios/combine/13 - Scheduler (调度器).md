> 本文是阅读 [Combine: Asynchronous Programming with Swift](https://store.raywenderlich.com/products/combine-asynchronous-programming-with-swift) 后的学习笔记，有需要的请点击链接购买。

# Scheduler (调度器)

## 调度器简介

根据苹果的文档，调度器是一个**协议**，定义何时和如何执行闭包。虽然这个定义是正确的，但它只解释了其中的一部分。

调度器提供了一个上下文以尽快或在将来的日期执行将来的操作。该操作是协议本身定义的闭包。但是，闭包这个词还可以隐藏 `Publisher` 在特定调度器上执行的某些值的传递。

我们可以发现在这个定义中没有提到任何关于与线程相关的词语。这是因为调度器协议的具体实现就是调度器协议提供的“上下文”执行位置的实现！因此，代码将在哪个线程上执行的确切细节取决于您选择的调度器。

记住这个重要的概念：**调度器并不等于线程**。

## 调度器相关操作符

Combine 提供了两个基本的调度器相关操作符：

- `subscribe(on:)` 和 `subscribe(on:options:)` 在指定的调度器上创建订阅。
- `receive(on:)` 和 `receive(on:options:)` 在指定的调度器上传递值。

除此之外，下面这些操作符也与调度器相关：

- `debounce(for:scheduler:options:)`
- `delay(for:tolerance:scheduler:options:)`
- `measureInterval(using:options:)`
- `throttle(for:scheduler:latest:)`
- `timeout(_:scheduler:options:customError:)`

### `subscribe(on:)`

我们可能希望 publisher 在后台执行一些复杂的计算，以避免阻塞主线程。简单的方法是使用 `subscribe(on:)`。

假设有以下例子：

```swift
let computationPublisher = Publishers.ExpensiveComputation(duration: 3)
let queue = DispatchQueue(label: "serial queue")

let subscription = computationPublisher
    .sink { value in
        let thread = Thread.current.number
        print("Received computation result on thread \(thread): '\(value)'")
}
```

代码解释：

- `Publishers` 是 Combine 定义的一个 `Publisher` 协议相关类型的命名空间。假设 `ExpensiveComputation` 是一个在指定 `duration` 后发出一个 `Computation complete` 字符串并结束的 publisher。

上述代码中，整个过程从订阅、到 `ExpensiveComputation` 进行复杂计算，再到 subscriber 接收到值，都是在主线程中执行，闭包中 `Thread.current.number` 为 `1`。

如果把代码改为：

```swift
let subscription = computationPublisher
    .subscribe(on: queue)
    .sink { value in
        let thread = Thread.current.number
        print("Received computation result on thread \(thread): '\(value)'")
}
```

加上 `.subscribe(on: queue)` 之后，订阅的创建还是在主线程完成的，但是 `ExpensiveComputation` 进行复杂计算和 subscriber 接收到值是在子线程执行的，闭包中的 `Thread.current.number` 就不是 `1` 了，而是其他比 `1` 大的值。

但如果我想在闭包中接收到值后更新 UI 呢? 也许我们会想到用 `DispatchQueue.main.async { ... }`。其实 Combine 已经提供了更好的方式：使用 `receive(on:)`。

### `receive(on:)`

例如：

```swift
let subscription = computationPublisher
    .subscribe(on: queue)
    .receive(on: DispatchQueue.main)
    .sink { value in
        let thread = Thread.current.number
        print("Received computation result on thread \(thread): '\(value)'")
}
```

在使用 `sink` 订阅之前加上 `.receive(on: DispatchQueue.main)` 之后，订阅的创建还是在主线程完成的，`ExpensiveComputation` 进行复杂计算是在子线程执行的，但闭包中 subscriber 接收到值是在主线程中执行的，`Thread.current.number` 为 `1`。

## Scheduler 的实现

苹果提供了几个 `Scheduler` 协议的实现：

- `ImmediateScheduler`：一个简单的调度器，它在当前线程上立即执行代码，默认的执行上下文，除非使用 `subscribe(on:)` 、`receive(on:)` 或其他任何接受调度器作为参数的运算符进行修改。
- `RunLoop`：绑定到 Foundation 的 `Thread` 对象。
- `DispatchQueue`：可以是串行的，也可以是并发的。
- `OperationQueue`：管理工作项执行的队列。

### `ImmediateScheduler`

我们先看下面例子：

```swift
var subscriptions = Set<AnyCancellable>()
let publisher = ["Hello", "Combine"].publisher

publisher
    .receive(on: ImmediateScheduler.shared)
    .sink { value in
        print("value: \(value), thread \(Thread.current.number)")
    }
    .store(in: &subscriptions)
```

例子中使用了 `ImmediateScheduler.shared` 调度器。运行结果如下：

```
value: Hello, thread 1
value: Combine, thread 1
```

`ImmediateScheduler` 的作用是在当前线程上执行。因为在调用 `.receive(on: ImmediateScheduler.shared)` 之前的线程是主线程，所以闭包中打印的线程是主线程。

如果在 `.receive(on: ImmediateScheduler.shared)` 之前再添加以 `.receive(on: DispatchQueue.global())`，完整代码如下：

```swift
publisher
    .receive(on: DispatchQueue.global())
    .receive(on: ImmediateScheduler.shared)
    .sink { value in
        print("value: \(value), thread \(Thread.current.number)")
    }
    .store(in: &subscriptions)
```

运行结果如下：

```
value: Hello, thread 7
value: Combine, thread 3
```

因为在调用 `.receive(on: ImmediateScheduler.shared)` 之前的线程是子线程，所以闭包中打印的线程是就不是主线程了。

#### `ImmediateScheduler` 选项

在大多数接受调度器的运算符的参数中，还可以找到一个 `options` 参数，该参数接受 `SchedulerOptions` 值。对于 `ImmediateScheduler`，在使用 `ImmediateScheduler` 时，此类型被定义为 `Never`，所以我们不能为运算符的 options 参数传递值。

#### `ImmediateScheduler` 的隐患

对于 `ImmediateScheduler`，要注意的一个问题是 **immediate**。您将无法使用任何 `Scheduler` 的 `schedule(after:)` 方法的变体，因为需要指定延迟的 `SchedulerTimeType` 没有公共的初始化器，并且对立即调度没有意义。

### `RunLoop` 调度器

有经验的 iOS 和 macOS 开发者应该都比较熟悉 `RunLoop`。在 `DispatchQueue` 之前，它是一种在线程级别管理输入源的方法，包括在主（UI）线程中。应用程序的主线程仍有关联的 `RunLoop`。可以通过在任何线程中调用 `RunLoop.current` 得到当前的 `RunLoop`。

> 注意：现在 `RunLoop` 是一个不太有用的类，因为 `DispatchQueue` 在大多数情况下是一个更好的选择。也就是说，仍然有一些特定的情况下 `RunLoop` 是有用的。例如，`Timer` 就是在 `RunLoop` 上进行的。UIKit 和 AppKit 依赖于 `RunLoop` 及其执行模式来处理各种用户输入情况。

下面看一个例子：

```swift
var subscriptions = Set<AnyCancellable>()
let publisher = ["Hello", "Combine"].publisher

publisher
    .receive(on: DispatchQueue.global())
    .receive(on: RunLoop.current)
    .sink { value in
        print("value: \(value), thread \(Thread.current.number)")
    }
    .store(in: &subscriptions)
```

代码中的 `RunLoop.current` 指的是在调用它时所在线程关联的 `RunLoop`。在本例中，这个代码是在主线程中调用的，所以闭包中打印的线程是 `1`：

```
value: Hello, thread 1
value: Combine, thread 1
```

#### `RunLoop` 选项

与 `ImmediateScheduler` `一样，RunLoop` 不为采用 `SchedulerOptions` 参数的调用提供任何合适的选项。

#### `RunLoop` 的隐患

`RunLoop` 的使用应该限制在主线程的运行循环中，并且在需要时限制在您控制的 Foundation 线程中可用的 RunLoop 中。也就是说，任何你使用 `Thread` 对象的地方。

要避免的一个特殊陷阱是在 `DispatchQueue` 上执行的代码中使用 `RunLoop.current`。这是因为 `DispatchQueue` 线程可能是短暂的，这使得它们几乎不可能依赖于 `RunLoop`。

### `DispatchQueue` 调度器

`DispatchQueue` 可以是串行（默认）或并发的。串行队列按顺序执行您提供给它的所有工作项。并发队列将并行启动多个工作项，以最大化 CPU 使用率。两种队列类型都有不同的用法：

- 串行队列通常用于保证某些操作不会重叠。因此，如果所有操作都发生在同一队列中，则它们可以使用共享资源而无需锁定。
- 并发队列将同时执行尽可能多的操作。因此，它更适合于纯计算。

#### 队列和线程

我们一直使用的最熟悉的队列是 `DispatchQueue.main`。它直接映射到主（UI）线程，在这个队列上执行的所有操作都可以自由地更新用户界面。

所有其他队列（串行或并发）在系统管理的线程池中执行它们的代码。这意味着您不应该对队列中运行的代码中的当前线程做任何假设。特别是，不应该使用 `RunLoop.current` 调度工作，因为 `DispatchQueue` 管理其线程的方式。

所有调度队列共享同一线程池。一个串行队列被赋予要执行的工作，将使用该池中的任何可用线程。一个直接的结果是，来自同一队列的两个连续工作项可能使用不同的线程，同时仍然按顺序执行。

**注意**：当使用 `subscribe(on:)` 和 `receive(on:)` 或任何其他接受调度器参数的操作符时，永远不要假设每次支持调度器的线程都是相同的。

#### DispatchQueue 作为调度器

我们直接看下面例子：

```swift
var subscriptions = Set<AnyCancellable>()
let publisher = ["Hello", "Combine"].publisher
let serialQueue = DispatchQueue(label: "Serial queue")

publisher
    .receive(on: serialQueue)
    .sink { value in
        print("value: \(value), thread \(Thread.current.number)")
    }
    .store(in: &subscriptions)
```

运行结果：

```
value: Hello, thread 7
value: Combine, thread 7
```

`serialQueue` 是我们创建的一个串行队列，在子线程运行。

#### DispatchQueue 选项

`DispatchQueue` 是唯一一个提供一组选项的调度器，当操作符接受 `SchedulerOptions` 参数时，可以传递这些选项。这些选项主要围绕指定 QoS（Quality of Service）值，而不依赖于已在 `DispatchQueue` 上设置的值。对于工作项有一些额外的标志，但是在绝大多数情况下不需要它们。

下面是一个指定 QoS 的例子：

```swift
.receive(
    on: serialQueue,
    options: DispatchQueue.SchedulerOptions(qos: .userInteractive)
)
```

将 `DispatchQueue.SchedulerOptions` 的实例传递给指定 qos 的选项：`.userInteractive`。它让操作系统尽最大努力将值交付优先于不太重要的任务。当我们希望尽快更新用户界面时，可以使用这个选项。相反，如果快速传递的压力较小，你可以使用 `.background`。

在实际应用程序中使用这些选项有助于操作系统在多个队列同时繁忙的情况下决定首先调度哪个任务。它可以微调你的应用程序性能！

### `OperationQueue`

这个类被定义为一个队列，它控制操作的执行。它是一种丰富的管理机制，可以让我们创建具有依赖关系的高级操作。但是在 Combine 的环境下，将不使用这些机制。

下面来看一个例子：

```swift
let queue = OperationQueue()
let subscription = (1...10).publisher
    .receive(on: queue)
    .sink { value in
        print("Received \(value) on thread \(Thread.current.number)")
    }
```

运行结果：

```
Received 3 on thread 7
Received 10 on thread 8
Received 1 on thread 3
Received 5 on thread 9
Received 4 on thread 5
Received 2 on thread 4
Received 9 on thread 10
Received 8 on thread 11
Received 7 on thread 12
Received 6 on thread 13
```

我们可以看到，虽然 publisher 是按顺序发出值的，但 `sink` 接收时的顺序是不确定，并且不是在同一个线程。因为 OperationQueue 使用 Dispatch 框架（也就是 DispatchQueue）来执行操作。这意味着它不能保证对每个传递的值使用相同的底层线程。

此外，每个 `OperationQueue` 中都有一个参数解释了所有内容：它是 `maxConcurrentOperationCount`。它默认为系统定义的数字，允许操作队列同时执行大量操作。由于 publisher 几乎同时发出其所有值，因此它们通过 `Dispatch` 的并发队列被调度到多个线程！

如果把 `maxConcurrentOperationCount` 改为 `1`：

```swift
queue.maxConcurrentOperationCount = 1
```

运行结果为：

```
Received 1 on thread 3
Received 2 on thread 3
Received 3 on thread 3
Received 4 on thread 3
Received 5 on thread 4
Received 6 on thread 3
Received 7 on thread 3
Received 8 on thread 3
Received 9 on thread 3
Received 10 on thread 3
```

在同一个线程上按顺序输出。

把 `maxConcurrentOperationCount` 设置为 `1` 其实就相当于使用串行队列。

#### `OperationQueue` 选项

没有可用于 `OperationQueue` 的 `SchedulerOptions`。它实际上是 `RunLoop.SchedulerOptions` 的别名，它本身没有提供任何选项。

#### `OperationQueue` 的隐患

我们刚刚看到 `OperationQueue` 在默认情况下并发执行操作。我们需要非常清楚这一点，因为它可能会给您带来麻烦：默认情况下，`OperationQueue` 的行为类似于并发的 `DispatchQueue`。

不过，当每次 publisher 发出值时都有大量的工作要执行时，它可能是一个很好的工具。您可以通过调整 `maxConcurrentOperationCount` 参数来控制负载。
