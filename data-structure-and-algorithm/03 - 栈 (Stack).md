栈在开发中是很常见的，例如 iOS 中的 `UINavigationController` 就是通过栈数据结构来管理它的子控制器。

栈，非常简单而又很有用。当向栈中添加数据时，直接放在栈的上面；要移除数据时，直接把最上面的那个移除掉。它主要有两个操作：

- **push**：在栈顶上添加元素
- **pop**：把栈顶的元素移除

所以我们只能在栈的顶部对元素进行操作，在计算机的专业术语中，我们把栈的成为** LIFO** (last in first out) 的数据结构。简单地说就是后面进来的元素先出去；最先进去的最后出去。

### 实现

用数组来存储栈中的元素，把 `Stack` 定义如下：

```swift
struct Stack<Element> {
    private var elements: [Element] = []
    init() { }
}

extension Stack: CustomStringConvertible {
    var description: String {
        let topDivider = "====top====\n"
        let bottomDivider = "\n====bottom====\n"
        let stackElements = elements
            .reversed()
            .map { "\($0)" }
            .joined(separator: "\n")
        return topDivider + stackElements + bottomDivider
    }
}
```

另外还实现了 `CustomStringConvertible`。

#### push 和 pop 操作

```swift
extension Stack {
    mutating func push(_ element: Element) {
        elements.append(element)
    }
    
    @discardableResult
    mutating func pop() -> Element? {
        return elements.popLast()
    }
}
```

`push` 的元素直接添加到数组后面，`pop` 把数组最后的元素移除。`push` 和 `pop` 操作的时间复杂度都是**O(1)**。

#### 使用

```swift
var stack = Stack<Int>()
stack.push(1)
stack.push(2)
stack.push(3)
stack.push(4)

print(stack)

stack.pop()

print(stack)

// 结果
====top====
4
3
2
1
====bottom====

====top====
3
2
1
====bottom====
```

#### 添加一些实用的方法

```swift
// MARK: - Getters
extension Stack {
    var top: Element? {
        return elements.last
    }
    
    var isEmpty: Bool {
        return elements.isEmpty
    }
    
    var count: Int {
        return elements.count
    }
}
```

`top`：栈顶的元素，`isEmpty`：栈是否为空，`count`：栈元素个数。

#### 实现 Swift 的 Collection 协议？

在上一篇的链表中，实现了 Swift 的 Collection 协议。但是对于栈来说，我们是不能去实现 Collection 协议。因为栈数据结构的很大一个特性就是不能让我们可以通过很多方法去轻易访问里面的元素，如果实现 Collection 协议，就会违反这个规则了。

#### 总结

栈的实现相对来说比较简单，栈的元素是先进后出的。

### [完整代码 >>](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Stack/Stack.swift)

### 参考资料

> [Data Structures and Algorithms in Swift](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift) --- [raywenderlich.com](https://www.raywenderlich.com/)，如果想看原版书籍，请点击链接购买。