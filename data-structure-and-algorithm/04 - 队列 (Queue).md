 队列在生活中非常常见。排队等位吃饭、在火车站买票、通过高速路口等，这些生活中的现象很好的描述了队列的特点：先进先出 (FIFO, first in first out)，排在最前面的先出来，后面来的只能排在最后面。

### 实现

这里我们利用 Swift 的数组来实现。

```swift
struct Queue<Element> {
    
    private var elements: [Element] = []
    
    init() { }
    
    // MARK: - Getters
    
    var count: Int {
        return elements.count
    }
    
    var isEmpty: Bool {
        return elements.isEmpty
    }
    
    var peek: Element? {
        return elements.first
    }
    
    // MARK: - Enqueue & Dequeue
    
    mutating func enqueue(_ element: Element) {
        elements.append(element)
    }
    
    @discardableResult
    mutating func dequeue() -> Element? {
        return isEmpty ? nil : elements.removeFirst()
    }
}

extension Queue: CustomStringConvertible {
    var description: String {
        return elements.description
    }
}
```

用数组实现队列比较简单：

- 1) 用数组存储所有队列的元素
- 2) 常用的一些属性: `count`、`isEmpty`、`peek` (访问第一个元素)
- 3) `enqueue(_:)`: 在队列最后添加元素
- 4) `dequeue()`: 移除队列的第一个元素
- 5) 遵循 `CustomStringConvertible`，直接使用`elements`的`description`

#### 使用

```swift
var queue = Queue<Int>()
queue.enqueue(1)
queue.enqueue(2)
queue.enqueue(3)
queue.enqueue(4)

print(queue)

queue.dequeue()

print(queue)

// 结果
[1, 2, 3, 4]
[2, 3, 4]
```

上面代码先插入四个元素，然后再把第一个移除。

#### 性能分析

|方法名            | 方法概述                   | 时间复杂度    |
| -----------------|:---------------------------|:-------------:|
| `enqueue(_:)`    | 在队列最后添加元素         |   O(1)        | 
| `dequeue()`      | 移除队列的第一个元素       |   O(n)        | 

- 入队的时间复杂度为`O(1)`，如果队列比较大，可能造成刚开始开辟的内存不够用，需要重新开辟内存空间。重新开辟空间的时间复杂度为`O(n)`，因为要把元素复制到新的内存。但是 Swift 在每次增加内存空间时，都会增加到原来的两倍，重新开辟内存操作的次数一般比较少，所以我们还是认为入队的时间复杂度为`O(1)`。
- 而出队为`O(n)`，因为在内存中，移除数组的第一个元素之后，后面的元素要往前移动（就像排队付款，前面的付完款之后，后面的往前走），所以造成时间复杂度为`O(n)`。
 

#### 总结

用数组来实现队列是非常简单的，但是`dequeue()`的时间复杂度是`O(n)`，有没有办法可以让解决这个缺点呢？有，我们可以使用双向链表来实现队列，但是双向链表的节点要记录前后节点的地址，所以占用的内存比较大。所以在实际使用中，要根据情况来选择。

### [完整代码 >>](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Queue/Queue.swift)

### 参考资料

> [Data Structures and Algorithms in Swift](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift) --- [raywenderlich.com](https://www.raywenderlich.com/)，如果想看原版书籍，请点击链接购买。
