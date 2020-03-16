> 本文是阅读 [Combine: Asynchronous Programming with Swift](https://store.raywenderlich.com/products/combine-asynchronous-programming-with-swift) 后的学习笔记，有需要的请点击链接购买。

# 01 - Combine 基础

在 Combine 框架面世之前，我们通常会使用以下几种方式来处理异步问题：
	- NotificationCenter
	- 代理模式
	- GCD 和 Operation
	- Closure (闭包)

但是这几种处理异步问题的方式不好处理执行顺序的问题。并且在实际开发中，我们会根据情况使用不同的方法来处理异步的问题。Combine 可以帮助我们在异步编程中更好地处理执行顺序的问题。

苹果已经在 Foundation 框架中集成了 Combine 的 API,例如 `Timer`、`NotificationCenter`和 **Core Data** 等等。

## Combine 的背景

声明式和响应式编程并不是一个新的概念。早在 2009 年，微软的一个团队就发布了 Rx.NET，并且在 2012 年开源。从那时开始，这个概念就扩展到了其他语言，例如 RxJava, RxJS, RxKotlin, RxScala, RxPHP 等等。

在苹果的平台上，也有类似的第三方库，例如 RxSwift 和 ReactiveSwift。终于在今年 2019 年，苹果发布了官方的响应式编程框架 Combine。Combine 只能兼容 iOS 13+, macOS 10.15+, tvOS 13.0+ 和 watchOS 6.0+ 的系统。因为 Combine 对系统的版本要求比较高，所以把 Combine 应用到实际开发中还为时尚早。但是相信这绝对是以后苹果平台开发的一个趋势。提前掌握这个核心的技术还是非常有必要的。

下面我们来看一下 Combine 框架的基础。Combine 的三个关键组成部分是：**Publisher**、**Operator** 和 **Subscriber**。

## Publisher

Publisher 可以不定时地向 一个或多个 Subscriber 发出某种类型的值。不管 Publisher 内部的逻辑如何，例如处理网络请求、用户事件等等，他们最终都可以发出下面这三种类型的事件：
	1）一个 Publisher 的泛型 `Output` 类型的值；
	2）一个成功的结束；
	3）一个带有 Error 的结束，类型是 Publisher 的泛型 `Failure` 类型。

一个 Publisher 可以发出一个或多个 `Output` 类型的值，如果发出了一个结束事件，不管是成功还是带有 Error 的结束，他都不再发出任何事件。

例如下面一个可以发出 Int 类型的 Publisher：

```
 					Publisher<Int, Never>
值： ---- 1 ---- 3 ---- 5 ---- 8 ---- 9 ---- 20 ---- 39 ---- 50 ---|>
时间：   0:01   0:03   0:06   0:10   0:19   0:15    0:20    0:50
```

虚线中间的数字是发出的值；最右边的 `|` 代表一个成功的结束。

我们来看一下 `Publisher` 的定义：

```swift
public protocol Publisher {
    associatedtype Output
    associatedtype Failure : Error
    func receive<S>(subscriber: S) where S : Subscriber, Self.Failure == S.Failure, Self.Output == S.Input
}
```

从定义中我们可以看到，`Publisher` 包含了两个泛型：
	- `Output` ： Publisher 发出的值类型。如果一开始定义为了 `Int` 类型，那么它就不能发出 `String`或者其他类型的值。
	-  `Failure` ：Publisher 可以发出的错误结束的类型。如果从头到尾都不发出错误结束，那么可以把 `Failure` 定义为 `Never` 类型。

当我们订阅 publisher 时，我们就知道了它发出的值的类型和错误结束的类型。

## Operators

Operators 是 在 Publisher 协议的基础上定义的一系列方法，可以返回同一个或新的 publisher，这样我们就可以把多个 operators 链接起来。它的任务是从上一个operator 接收到的数据通过处理，然后输出处理后的数据，传给下一个 operator。这样我们就保证的代码逻辑的处理顺序。

例如 `<Int, MyError>` 在经过了两个 Operators 之后变成了 `<Text, Never>`：

```
<Int, MyError> ---> Operator(<String, Error>) ---> Operator(<Text, Never>) ---> <Text, Never>
```

我们可以根据不同的需要调整 operators 的顺序。

## Subscribers

Subscribers 从 Publisher 中订阅相关事件，例如发出的值或者结束事件，得到这些事件后，去处理后续的逻辑。

```
                                    ---> 在屏幕上显示
                                  -
                                -
2 ---> 4 ---> 6 ---> Subscriber
                                -
                                  -
                                    ---> 发送到服务器
```

目前 Combine 提供了两个内置的 subscribers：
	- **sink**：可以让我们提供一个 closure 来接收发出的值和结束事件。
	- **assign**：可以让我们直接把发出的值绑定到数据模型或者 UI 控件的属性上，直接把最新的数据显示在 UI 上，不需要我们编写任何自定义代码。

如果这两个内置的 subscribers 无法满足需求，我们可以很容易地自定义 subscribers。

## Subscriptions

当我们订阅结束后添加一个 subscriber，他就会激活 publisher。如果没有 subscribers 去订阅 publisher，那么它就不会发出任何值。

Subscription 可以让我们一次性写好一系列的异步事件。当熟悉了 Combine 之后，我们可以通过 subscriptions 来描述整个 app 的逻辑。一旦把 subscriptions 的代码写好，我们就可以不用去管它了。

另外，我们不需要另外对 subscriptions 做内存管理。因为 Combine 提供了一个 `Cancellable` 协议，系统自带的 subscribers 都遵循了这个协议，在 subscription 创建成功之后就会返回一个 `Cancellable` 对象，当这个对象被释放后，整个订阅也会被释放。所以我们只需要把这个  `Cancellable` 对象当做一个属性存储在类似 ViewController 里就可以了。如果有多个 subscriptions，还可以把多个  `Cancellable` 对象存储到  `[Cancellable]`中。