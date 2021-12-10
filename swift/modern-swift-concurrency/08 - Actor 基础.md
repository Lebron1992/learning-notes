# 08 - Actor 基础

## 理解线程安全代码

线程安全代码通常意味着，无论是从主线程还是从后台线程调用 API，该方法的行为都将始终符合预期。换句话说，即使多个线程同时调用该方法，该方法仍然可以工作。

不幸的是，在 Objective-C 和 Swift 5.5 之前的版本中，没有语法将方法标记为线程安全的。必须依靠每种方法的文档来确定它是否安全。

例如下面的代码：

```swift
class Counter {
  private var count = 0

  func increment() {
    count += 1
  }
}
```

当看到上面的代码时，并没有什么会使它特别不安全。然而，如果两个并行运行的线程都调用了 `Counter.increment()`，那么可能不会得到正好增加 2 的 `count`。更糟糕的是，如果对 `Counter.increment()` 的两个调用恰好发生在同一时刻，那么应用程序将崩溃。

因此，我们可以说，任何不采取主动措施保护共享可变状态免受并发访问的代码本质上都不是线程安全的。

在 Swift 5.5 之前，我们可以使用**锁**或**串行调度队列**来确保对共享状态的独占访问。例如，使用锁，线程锁定对共享资源的访问，其他线程需要等待它解锁，然后才能读取或写入同一资源。

使用锁 API 的并发代码在编写良好时相当快速和安全。使用锁时，前面的代码示例如下所示：

```swift
class Counter {
  private var lock = os_unfair_lock_s()
  private var count = 0

  func increment() {
    os_unfair_lock_lock(&lock)
    count += 1
    os_unfair_lock_unlock(&lock)
  }
}
```

如果没有访问代码的权限，或者没有空闲时间彻底阅读代码，那么就无法判断它是否真的安全。Actor 就可以解决这些问题。

## 遇见 Actor

`actor` 类型是 Swift 5.5 中引入的与并发相关的改进之一。`actor` 是一种编程类型，就像这些类型一样：`enum`、`struct`、`class` 等等。更具体地说，它是一个类似 `class` 的引用类型。

使用 `actor` 重写上面的例子：

```swift
actor Counter {
  private var count = 0

  func increment() {
      count += 1
  }
}
```

Actor 是一个现有的、成熟的并发计算模型。它的行为遵循一些基本规则，这些规则使它保证其内部状态的安全。

Swift中的 Actor 可以安全地访问并改变自己的状态。一种称为串行执行器 (serial executor) 的特殊类型，由运行时管理，用于同步对 Actor 成员的所有调用。串行执行器很像 GCD 中的串行调度队列，一个接一个地执行任务。通过这样做，它可以保护 Actor 的状态不受并发访问。

当查看所有 actors 都遵循的 [`Actor`](https://developer.apple.com/documentation/swift/actor) 协议时，将看到只有一个要求。也就是说，所有 actors 都必须有一个名为 `unownedExecutor` 的属性，该属性是前面提到的序列化对 actor 状态的访问的执行器。

但数据竞争的真正原因是什么呢？如何保证另一个类型不会同时从多个线程调用 actor 并导致崩溃？

`actor` 与 Swift 编译器有特殊协议来处理这个问题。从其他类型对 actor 的访问将自动异步执行，并安排在 actor 的串行执行器上。这称为状态隔离层。状态隔离层确保所有状态变异都是线程安全的。`actor` 是 API、编译器和运行时使用者线程安全的保证。

## 认识 `MainActor`

在之前的章节中，已经接触过 `MainActor`：通过调用 `MainActor.run(...)` 在 main actor 上运行代码；使用 `@MainActor` 注释了应该自动在 main actor 上运行的方法。

`MainActor` 也是属于 `actor` 类型。main actor 在主线程上连续运行代码，并保护其共享状态：应用程序的 UI。它是一个可以从任何地方访问的全局 actor，可以在整个应用程序中使用它的共享实例。

## 理解 `Sendable`

`Sendable` 是一个协议，它指示在并发代码中使用给定值是安全的。

查看 [`Sendable`](https://developer.apple.com/documentation/swift/sendable) 的文档你会发现有一些协议继承它。例如，`Actor` 协议继承自 `Sendable`；因此，在并发代码中使用 actor 实例是安全的。还有 Swift 自带的很多类型都实现了 `Sendable`，例如： `Bool`、`Double`、`Int`、`StaticString`、`UnsafePointer` 等等。

一般来说，在并发代码中使用值类型是安全的。class 类型通常不是 `Sendable`，因为它们是引用类型，可以在内存中对同一实例进行修改。

可以使用 `@Sendable` 在代码中要求线程安全的值。换句话说，使用它来要求值必须遵循 `@Sendable` 协议。

例如，`Task` 类型。因为它创建了一个异步任务，该任务可能会不安全地改变共享状态。`Task.init(...)` 声明要求 `operation` 闭包是 `Sendable`：

```swift
init(
  priority: TaskPriority? = nil, 
  operation: @escaping @Sendable () async -> Success
)
```

`operation` 闭包是 `@escaping`，因为它是异步的；同时也是 `@Sendable`，在编译时验证闭包代码是线程安全的。

一旦我们自定义的某个类型遵循 `Sendable`，编译器将自动以各种方式限制它，以帮助确保其线程安全。例如，它会要求将类设为 `final`，使类属性不可变，等等。

`addTask(...)` 的闭包参数也是 `Sendable`：

```swift
mutating func addTask(
  priority: TaskPriority? = nil, 
  operation: @escaping @Sendable () async -> ChildTaskResult
)
```

因此，代码中的最佳实践是要求异步运行的任何闭包都是 `@Sendable`，并且异步代码中使用的任何值都遵循 `Sendable` 协议。

另外，如果 struct 或者 class 是线程安全的，那么还应该遵循 `Sendable` 协议，以便其他并发代码可以安全地使用它。
