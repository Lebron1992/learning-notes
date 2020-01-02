在[【数据结构与算法 - Swift实现】04 - 队列 (Queue)](http://zengwenzhi.com/2018/07/08/%E3%80%90%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95-swift%E5%AE%9E%E7%8E%B0%E3%80%9104-%E9%98%9F%E5%88%97-queue/)文章里面，我们已经讲过队列，采用**先进先出**的规则。这边文章我们要学习的是优先队列，根据优先级的高低来决定出队的顺序。

### 实现

在上一篇文章[【数据结构与算法 – Swift实现】09 – 数据结构 堆 (Heap)](http://zengwenzhi.com/2018/07/23/%E3%80%90%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95-swift%E5%AE%9E%E7%8E%B0%E3%80%9109-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84-%E5%A0%86-heap/)中，我们了解了堆的特点，最大堆能在 `O(1)` 时间内得到最大值，最小堆能在 `O(1)` 时间内得到最小值。优先队列是根据优先级的高低来决定出队的顺序，堆的这个特点非常适合用来实现优先队列。

采用堆实现优先队列比较简单，直接给出实现代码如下：

```swift
struct PriorityQueue<Element: Equatable> {
    private var heap: Heap<Element>
    
    init(order: @escaping (Element, Element) -> Bool) {
        heap = Heap(order: order)
    }
    
    var isEmpty: Bool {
        return heap.isEmpty
    }
    
    var peek: Element? {
        return heap.peek
    }
    
    mutating func enqueue(_ element: Element) {
        heap.insert(element)
    }
    
    mutating func dequeue() -> Element? {
        return heap.removePeek()
    }
}

extension PriorityQueue: CustomStringConvertible {
    var description: String {
        return heap.description
    }
}
```

下面我们来测试一下：

```swift
var priorityQueue = PriorityQueue<Int>(order: >)
for i in 1...7 {
    priorityQueue.enqueue(i)
}

while !priorityQueue.isEmpty {
    print(String(describing: priorityQueue.dequeue()))
}

// 结果
Optional(7)
Optional(6)
Optional(5)
Optional(4)
Optional(3)
Optional(2)
Optional(1)
```

数值较大的先出列，符合我们传入的 `order` 参数。

### [完整代码 >>](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Priority%20Queue/PriorityQueue.swift)

### 参考资料

> [Data Structures and Algorithms in Swift](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift) --- [raywenderlich.com](https://www.raywenderlich.com/)，如果想看原版书籍，请点击链接购买。
