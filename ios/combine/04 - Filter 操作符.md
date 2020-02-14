# 04 - Filter 操作符

## `filter(_:)`

这个操作符接收一个返回 `Bool` 类型的闭包，返回 `false` 时，传入的值就会被过滤掉。

例如：

```swift
(1...10).publisher
    .filter { $0.isMultiple(of: 3) }
    .sink(receiveValue: { print($0) })
```

运行结果：

```
3
6
9
```

## `removeDuplicates`

这个操作符可以用来过滤重复值，这里的重复是指连续的值重复。例如： publisher 发出 `1` 、 `1` 和 `2`，`1` 是重复的，使用这个操作符之后，下面只会收到 `1` 和 `2`；而 `1` 、`2` 和 `1` 中，`1` 是不重复的，三个值都能通过这个操作符。

我们来看下面例子：

```swift
let words = "hey hey there! want to listen to mister mister ?" .components(separatedBy: " ")
    .publisher

words
    .removeDuplicates()
    .sink(receiveValue: { print($0) })
```

运行结果：

```
hey
there!
want
to
listen
to
mister
?
```

这里的值是 `String` 类型，遵循 `Equatable` 协议。如果没有遵循这个协议或者想要自己决定是否属于重复，可以使用它的一个重载方法 `removeDuplicates(by:)` 来过滤重复值，这个方法接收一个返回值为 `Bool` 类型的闭包。

## `compactMap(_:)`

这个操作符接收一个闭包，它可以帮我们把闭包返回的 `nil` 过滤掉。例如：

```swift
let strings = ["a", "1.24", "3",
               "def", "45", "0.23"].publisher
strings
    .compactMap { Float($0) }
    .sink(receiveValue: { print($0) })
```

运行结果：

```
1.24
3.0
45.0
0.23
```

## `ignoreOutput`

有时候我们只想知道 publisher 什么时候结束，但不关心它发出的值，我们可以使用这个操作符。例如：

```swift
let numbers = (1...10_000).publisher
numbers
    .ignoreOutput()
    .sink(
        receiveCompletion: { print("Completed with: \($0)") },
        receiveValue: { print($0) }
)
```

运行结果：

```
Completed with: finished
```

不会收到任何值，只看到了结束事件。

## `first(where:)` 和 `last(where:)`

这两个操作符分别用来查找符合闭包条件的第一个和最后一个值，找到之后就发出那个值，然后马上发出结束事件；如果没有符合条件的，则只会发出结束事件。

例如：

```swift
let numbers = (1...9).publisher
numbers
    .first(where: { $0 % 2 == 0 })
    .sink(
        receiveCompletion: { print("Completed with: \($0)") },
        receiveValue: { print($0) }
```

运行结果：

```
2
Completed with: finished
```

同样上面的代码，如果把 `first` 改为 `last`，运行结果为：

```
8
Completed with: finished
```

在内部的实现原理中，这两个操作符还是有区别的：`first(where:)` 是用来找到第一个满足条件的，所以当他找到一个值之后，就会取消对 `numbers` 的订阅，它不需要从 `numbers` 接收所有的值；而 `last(where:)` 因为要找到最后一个满足条件的值，所以它必须从 `number` 接收所有值，然后把最后一个满足条件的值发出去。所以对于 `last(where:)` 操作符，在操作符链中，它的上一个 publisher 必须是在某个时间点它会发出结束事件，否则操作符链就会一直停留在 `last(where:)`，无法继续进行下去。

例如，我们来看下面例子：

```swift
let numbers = PassthroughSubject<Int, Never>()
numbers
    .last(where: { $0 % 2 == 0 })
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) }
)
numbers.send(1)
numbers.send(2)
numbers.send(3)
numbers.send(4)
numbers.send(5)
```

运行上面的代码之后，没有任何输出，原因是 `last(where:)` 一直在等待 `numbers` 发出结束事件，可以是 `numbers` 没有发出结束事件。如果在最后加上:

```swift
numbers.send(completion: .finished)
```

运行后就能看到：

```
4
Completed with: finished
```

## `dropFirst(count:)`

这个操作符可以用来过滤前面的几个值，`count` 参数用于指定要过滤的数量，默认是 `1`。

例如：

```swift
let numbers = (1...10).publisher
numbers
    .dropFirst(8)
    .sink(receiveValue: { print($0) })
```

运行结果：

```
9
10
```

## `drop(while:)`

这个操作符接收一个返回 `Bool` 类型的闭包参数，在闭包返回 `false` 之前，前面的值都会被过滤掉，一旦闭包返回 `false`，他就会开始发送值，直到结束。例

例如：

```swift
let numbers = (1...10).publisher
numbers
    .drop(while: { $0 % 5 != 0 })
    .sink(receiveValue: { print($0) })
```

运行结果：

```
5
6
7
8
9
10
```

`filter(_:)` 和 `drop(while:)` 很容易混淆。他们的区别是：对于接收到的每个值，`filter(_:)` 操作符中，每个值都会经过闭包运算，只有返回 `true` 的值才能通过；而对于 `drop(while:)`，经过闭包运算返回 `true` 的值会被过滤，当第一次遇到返回 `false` 时，当前这个值可以通过，并且后面的值都不需要经过闭包运算就可以通过。

## `drop(untilOutputFrom:)`

这个操作符接收的参数是一个 publisher，只有当参数的 publisher 发出值的时候，它才会让它接收到的值通过，在此之前的值都会被过滤掉。

例如有这么一种情况：用户在点击一个 button，但是你想等一个 `isReady` publisher 发出值的时候，才对用户的点击进行响应。我们就可以使用 `drop(untilOutputFrom:)` 来处理。

下面用代码演示刚刚那种情况：

```swift
let isReady = PassthroughSubject<Void, Never>()
let taps = PassthroughSubject<Int, Never>()

taps
    .drop(untilOutputFrom: isReady)
    .sink(receiveValue: { print($0) })

(1...5).forEach { n in
    taps.send(n)
    if n == 3 {
        isReady.send()
    }
}
```

运行结果：

```
4
5
```

当 `taps` 发送了 `3` 之后，`isReady` 才发送值，所以只输出 `4` 和 `5`。

## `prefix` 家族

### `prefix(_:)`

跟 `dropFirst` 相反，`prefix(_:)` 只让前面几个值通过。

例如：

```swift
let numbers = (1...10).publisher
numbers
    .prefix(2)
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) }
)
```

运行结果：

```
1
2
Completed with: finished
```

## `prefix(while:)`

这个操作符接收一个返回 `Bool` 类型的闭包参数，只有前面闭包返回 `true` 的值才能通过，一旦遇到 `false`，上一个 publisher 就会结束。

例如：

```swift
let numbers = (1...10).publisher
numbers
    .prefix(while: { $0 < 3 })
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) }
)
```

运行结果：

```
1
2
Completed with: finished
```

### `prefix(untilOutputFrom:)`

这个操作符刚好跟 `drop(untilOutputFrom:)` 相反，它只让参数中的 publisher 发出值之前的值通过，一旦参数中的 publisher 发送值，上一个 publisher 就会结束。

例如：

```swift
let isReady = PassthroughSubject<Void, Never>()
let taps = PassthroughSubject<Int, Never>()

taps
    .prefix(untilOutputFrom: isReady)
    .sink(receiveCompletion: { print("Completed with: \($0)") },
          receiveValue: { print($0) }
)

(1...5).forEach { n in
    taps.send(n)
    if n == 2 {
        isReady.send()
    }
}
```

运行结果：

```
1
2
Completed with: finished
```
