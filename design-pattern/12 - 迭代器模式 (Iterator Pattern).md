> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

迭代器模式提供了一种遍历集合的方式。这个模式设计两种类型：1） **迭代器协议**；2） **自定义的可遍历的类型**。

Swift里面的集合可以通过`for-in`来遍历的类型，就是利用了迭代器协议`IteratorProtocol`。

### 什么时候使用

当我们的class或者struct拥有一组排好序的对象，并且我们想通过`for-in`来遍历这组对象时，使用迭代器模式。

### 简单demo

我们实现一个队列，然后通过遵循`Sequence`协议，实现使用`for-in`来遍历队列里面的元素。

#### 队列

跟数组和字典一样，队列也是一种数据结构。队列遵循的规则是**先进先出**。

```swift
struct Queue<T> {
    private var array: [T?] = []
    private var head = 0
    
    var isEmpty: Bool {
        return count == 0
    }
    
    var count: Int {
        return array.count - head
    }
    
    mutating func enqueue(_ element: T) {
        array.append(element)
    }
    
    mutating func dequeue() -> T? {
        guard head < array.count,
            let element = array[head] else {
                return nil
        }
        array[head] = nil
        head += 1
        return element
    }
}

extension Queue: Sequence {
    func makeIterator() -> IndexingIterator<ArraySlice<T?>> {
        let values = array[head..<array.count]
        return values.makeIterator()
    }
}
```

在队列的实现里面，我们用一个数组来存储队列的元素，并实现了入队和出队的方法。

为了可以使用`for-in`来遍历队列中的元素，我们要遵循`Sequence`协议。`Sequence`协议有两个要求：1）关联值类型，也就是迭代器的类型，在上面的代码中，关联值类型是`IndexingIterator`，这是所有集合类型的默认迭代器类型。2）实现`makeIterator()`方法。

#### 使用

```swift
struct Person {
    let name: String
}

let lebron = Person(name: "Lebron James")
let love = Person(name: "Kevin Love")
let korver = Person(name: "Kyle Korver")

// 入队
var queue = Queue<Person>()
queue.enqueue(lebron)
queue.enqueue(love)
queue.enqueue(korver)

// 出队
queue.dequeue()

for person in queue {
    print(person?.name ?? "此人不存在")
}

// 结果
Kevin Love
Kyle Korver
```

这里我们定义了一个`Person`类型，用于定义`Person`队列。添加了三个人，然后将第一个人剔除，最终剩下两个。

### 总结

Swift里面有迭代器协议`IteratorProtocol`，但是我们不需要直接去遵循这个协议，而是通过遵循`Sequence`来实现，好处是我们不需要自己自定义一个迭代器，而且还可以获得集合的很多方法，例如`filter`、`map`和`sort`等等。
