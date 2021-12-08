# 04 - 用 AsyncStream 自定义异步序列

## 深入了解 `AsyncSequence``、AsyncIteratorProtocol` 和 `AsyncStream`

```swift
@rethrows public protocol AsyncSequence {

  associatedtype AsyncIterator : AsyncIteratorProtocol

  associatedtype Element where Self.Element == Self.AsyncIterator.Element

  func makeAsyncIterator() -> Self.AsyncIterator
}
```

`AsyncSequence` 协议只要求定义元素的类型和提供一个 iterator。关于如何生成元素，没有进一步的要求，对类型生存期没有任何约束。另外，打开 `AsyncSequence` 的[文档](https://developer.apple.com/documentation/swift/asyncsequence)，可以看到该协议附带了一系列类似于 `Sequence` 提供的方法：

```swift
func contains(_:) -> Bool
func allSatisfy(_:) -> Bool
func first(where:) -> Self.Element?
func min() -> Self.Element?
func max() -> Self.Element?
...
```

通过遵循 `AsyncSequence` 协议，可以免费利用协议的默认实现：`prefix(while:)`、 `contains()`、 `min()`、 `max()` 等等。

提供的 iterator 需要遵循 `AsyncIteratorProtocol` 协议。它只有一个要求：返回序列中下一个元素的 `async` 方法：

```swift
@rethrows public protocol AsyncIteratorProtocol {

  associatedtype Element

  mutating func next() async throws -> Self.Element?
}
```

### 自定义异步序列

下面是一个简单的自定义异步序列，用于打印一个字符串，每秒增加一个字符：

```swift
struct Typewriter: AsyncSequence {
  typealias Element = String

  let phrase: String

  func makeAsyncIterator() -> Iterator {
    return Iterator(phrase)
  }
}

extension Typewriter {
  struct Iterator: AsyncIteratorProtocol {
    typealias Element = String

    let phrase: String
    var index: String.Index

    init(_ phrase: String) {
      self.phrase = phrase
      self.index = phrase.startIndex
    }

    mutating func next() async throws -> String? {
      guard index < phrase.endIndex else {
        return nil
      }
      try await Task.sleep(nanoseconds: 1_000_000_000)
      defer {
        index = phrase.index(after: index)
      }
      return String(phrase[phrase.startIndex...index])
    }
  }
}
```

iterator 保存了字符串的副本。每次调用 `next()`，它都会返回初始字符串的子字符串，该子字符串比最后一个字符串长一个字符。

当它到达末尾时，无论是通过 `for wait` 循环还是直接调用 `next()` 的代码，`next()` 都返回 `nil` 以表示序列的结束。

使用 `Typewriter`：

```swift
for try await item in Typewriter(phrase: "Hello, world!") {
  print(item)
}
```

输出结果：

```
H
He
Hel
Hell
Hello
Hello,
Hello, 
Hello, w
Hello, wo
Hello, wor
Hello, worl
Hello, world
Hello, world!
```

### 使用 `AsyncStream` 简化自定义序列

为了简化异步序列的创建，苹果增加了 `AsyncStream` 的类型，其目的是使创建异步序列尽可能简单和快速。

它遵循 `AsyncSequence` 并从单个闭包生成值，在闭包中自定义序列的逻辑。

除了从 `AsyncSequence` 继承所有默认方法外，`AsyncStream` 有只有简单的几个接口：

- `init(:bufferingPolicy::)`：通过给定的闭包创建一个生成给定类型的值的新流。闭包可以通过一个称为 continuation 的结构来控制序列。生成但未使用的值保存在缓冲区中。如果不使用该选项设置该缓冲区的存储限制，则将缓冲所有未使用的值。

- `init(unfolding:onCancel:)`：创建一个新的流，该流通过从 `unfolding` 闭包返回值来生成值。它可以选择在取消时执行 `onCancel` 闭包。

使用 `AsyncStream` 重写 `Typewriter`：

```swift
let phrase = "Hello, world!"

var index = phrase.startIndex
let stream = AsyncStream<String> {
  guard index < phrase.endIndex else {
    return nil
  }

  do {
    try await Task.sleep(nanoseconds: 1_000_000_000)
  } catch {
    return nil
  }

  defer {
    index = phrase.index(after: index)
  }

  return String(phrase[phrase.startIndex...index])
}

for try await item in stream {
  print(item)
}
```
