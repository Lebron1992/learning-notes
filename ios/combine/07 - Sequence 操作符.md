> 本文是阅读 [Combine: Asynchronous Programming with Swift](https://store.raywenderlich.com/products/combine-asynchronous-programming-with-swift) 后的学习笔记，有需要的请点击链接购买。

# 07 - Sequence 操作符

这里所记录的操作符跟 Swift 标准库中的几乎一样，可以直接类比过来理解。

## `min`

这个操作符可以用来找出 publisher 所发出的全部值中的最小值。这也就意味着它要等 publisher 结束之后，才会把最小值发出来。

例如：

```swift
let publisher = [1, -50, 246, 0].publisher
publisher
    .print("publisher")
    .min()
    .sink(receiveValue: { print("Lowest value is \($0)") })
```

运行结果如下：

```
publisher: receive subscription: ([1, -50, 246, 0])
publisher: request unlimited
publisher: receive value: (1)
publisher: receive value: (-50)
publisher: receive value: (246)
publisher: receive value: (0)
publisher: receive finished
Lowest value is -50
```

可以看到 `publisher` 不断发出值，但是 `min` 要等 `publisher` 结束之后，才发出最小值 `-50`。

那么问题来了，`min` 操作符是根据什么进行比较的呢？答案是根据 `Comparable` 协议。这个例子中 `Int` 类型已经支持 `Comparable` 协议，所以可以直接调用 `min()`。如果比较的值类型没有遵循这个协议，可以调用 `min(by:)`，参数是一个返回 `Bool` 类型闭包，可以在闭包里面自定义比较的逻辑。

## `max`

这个操作符可以用来找出 publisher 所发出的全部值中的最大值。用法跟 `min` 一样。

例如：

```swift
let publisher = [1, -50, 246, 0].publisher
publisher
    .print("publisher")
    .min()
    .sink(receiveValue: { print("Lowest value is \($0)") })
```

运行结果如下：

```
publisher: receive subscription: ([1, -50, 246, 0])
publisher: request unlimited
publisher: receive value: (1)
publisher: receive value: (-50)
publisher: receive value: (246)
publisher: receive value: (0)
publisher: receive finished
Highest value is 246
```

## `first`

这个操作符用来发出 publisher 的第一个值，然后就马上结束，并取消对 publisher 的订阅。

例如：

```swift
let publisher = ["A", "B", "C"].publisher
publisher
    .print("publisher")
    .first()
    .sink(receiveValue: { print("First value is \($0)") })
```

运行结果如下：

```
publisher: receive subscription: (["A", "B", "C"])
publisher: request unlimited
publisher: receive value: (A)
publisher: receive cancel
First value is A
```

当然，`first` 也可以接收一个返回 `Bool` 类型的闭包参数，这个在 《04 - Filter 操作符》已经讲过。

## `last`

这个操作符用来发出 publisher 的最后一个值，这也就意味着它要等 publisher 结束之后才发这最后一个值。

例如：

```swift
let publisher = ["A", "B", "C"].publisher
publisher
    .print("publisher")
    .last()
    .sink(receiveValue: { print("Last value is \($0)") })
```

运行结果如下：

```
publisher: receive subscription: (["A", "B", "C"])
publisher: request unlimited
publisher: receive value: (A)
publisher: receive value: (B)
publisher: receive value: (C)
publisher: receive finished
Last value is C
```

## `output(at:)`

这个操作符用来发出在 publisher 中指定索引的值。

例如：

```swift
let publisher = ["A", "B", "C"].publisher
publisher
    .print("publisher")
    .output(at: 1)
    .sink(receiveValue: { print("Value at index 1 is \($0)") })
```

运行结果如下：

```
publisher: receive subscription: (["A", "B", "C"])
publisher: request unlimited
publisher: receive value: (A)
publisher: request max: (1) (synchronous)
publisher: receive value: (B)
Value at index 1 is B
publisher: receive cancel
```

## `output(in:)`

这个操作符用来发出在 publisher 中指定索引范围的值。

例如：

```swift
let publisher = ["A", "B", "C", "D", "E"].publisher
publisher
    .output(in: 1...3)
    .sink(receiveCompletion: { print($0) },
          receiveValue: { print("Value in range: \($0)") }
)
```

运行结果如下：

```
Value in range: B
Value in range: C
Value in range: D
finished
```

## `count`

这个操作符用来发出一个 publisher 发出的所有值的个数。

例如：

```swift
let publisher = ["A", "B", "C"].publisher
publisher
    .print("publisher")
    .count()
    .sink(receiveValue: { print("I have \($0) items") })
```

运行结果如下：

```
publisher: receive subscription: (["A", "B", "C"])
publisher: request unlimited
publisher: receive value: (A)
publisher: receive value: (B)
publisher: receive value: (C)
publisher: receive finished
I have 3 items
```

## `contains`

这个操作符用来发出一个 publisher 发出的值中是否包含某个值。如果在 publisher 结束之前就找到这个值，那么直接发出 `true`，然后结束，并且取消对 publisher 的订阅；如果 publisher 结束之后都没有找到，则发出 `false`。

例如：

```swift
let publisher = ["A", "B", "C", "D", "E"].publisher
let letter = "C"
publisher
    .print("publisher")
    .contains(letter)
    .sink(receiveValue: { contains in
        print(contains ? "Publisher emitted \(letter)!"
            : "Publisher never emitted \(letter)!")
    })
```

运行结果如下：

```
publisher: receive subscription: (["A", "B", "C", "D", "E"])
publisher: request unlimited
publisher: receive value: (A)
publisher: receive value: (B)
publisher: receive value: (C)
publisher: receive cancel
Publisher emitted C!
```

另外，`contains` 也可以接收一个返回 `Bool` 类型的闭包参数，用于自定义判断条件。

## `allSatisfy`

这个操作符用判断一个 publisher 发出的所有值是否符合某个条件，如果全部都满足，那么它会在 publisher 结束时发出 `true`；如果 publisher 结束之前就发现一个不满足条件的值，那么就会发出 `false`，并且取消对 publisher 的订阅。

例如：

```swift
let publisher = stride(from: 0, to: 5, by: 2).publisher // [0, 2, 4]
publisher
    .print("publisher")
    .allSatisfy { $0 % 2 == 0 }
    .sink(receiveValue: { allEven in
        print(allEven ? "All numbers are even"
            : "Something is odd...")
    })
```

运行结果如下：

```
publisher: receive subscription: (Sequence)
publisher: request unlimited
publisher: receive value: (0)
publisher: receive value: (2)
publisher: receive value: (4)
publisher: receive finished
All numbers are even
```

## `reduce`

这个操作符跟 Swift 标准库的 `reduce` 是同一个逻辑，他接收两个参数：初始值和闭包。这个闭包也接收两个参数，第一个是上一次闭包返回的结果，第二个当前 publisher 发出的值。当 publisher 结束后，它会发出最后一次闭包的运算结果。

例如：

```swift
let publisher = ["Hel", "lo", " ", "Wor", "ld", "!"].publisher
publisher
    .print("publisher")
    .reduce("") { accumulator, value in
        accumulator + value
}
.sink(receiveValue: { print("Reduced into: \($0)") })
```

运行结果如下：

```
publisher: receive subscription: (["Hel", "lo", " ", "Wor", "ld", "!"])
publisher: request unlimited
publisher: receive value: (Hel)
publisher: receive value: (lo)
publisher: receive value: ( )
publisher: receive value: (Wor)
publisher: receive value: (ld)
publisher: receive value: (!)
publisher: receive finished
Reduced into: Hello World!
```

`reduce` 和 《03 - Transform 操作符》 讲到的 `scan` 非常类似，唯一的区别是 `reduce` 只在 publisher 结束后才发出最后一次闭包运算的结果，而 `scan` 会在每次 publisher 发出值时都发出经过闭包运算的结果。
