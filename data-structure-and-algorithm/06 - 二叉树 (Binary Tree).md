二叉树中的每个节点最多只有两个子节点，左子节点和右子节点：

![](http://upload-images.jianshu.io/upload_images/2057254-f73dcf3a9248fc05.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 实现

根据上面描述的二叉树的特点，创建`BinaryTreeNode`：

```swift
final class BinaryTreeNode<T> {
    var value: T
    var leftChild: BinaryTreeNode?
    var rightChild: BinaryTreeNode?
    
    init(_ value: T) {
        self.value = value
    }
}
```

- `value`：存储当前节点的值
- `leftChild`：左子节点
- `leftChild`：右子节点

我们来尝试使用一下：

```swift
let zero = BinaryTreeNode(0)
let one = BinaryTreeNode(1)
let two = BinaryTreeNode(2)
let three = BinaryTreeNode(3)
let four = BinaryTreeNode(4)
let five = BinaryTreeNode(5)
let six = BinaryTreeNode(6)

three.leftChild = one
three.rightChild = five

one.leftChild = zero
one.rightChild = two

five.leftChild = four
five.rightChild = six
```

上面的代码创建了这样一个树结构：

![](http://upload-images.jianshu.io/upload_images/2057254-95e0da30975ee22f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 遍历算法

二叉树的遍历算法有三种：1）中序遍历；2）前序遍历；3）后序遍历。

#### 中序遍历

遍历顺序：对当前节点的左子节点进行中序遍历 --> 当前节点 --> 对当前节点的右子节点进行中序遍历，简称**左根右**。

代码实现：

```swift
func traverseInOrder(_ closure: (T) -> Void) {
    leftChild?.traverseInOrder(closure)
    closure(value)
    rightChild?.traverseInOrder(closure)
}
```

对上面创建的树形结构，用中序遍历:

```swift
three.traverseInOrder { print($0) }

// 结果
0
1
2
3
4
5
6
```

#### 前序遍历

遍历顺序：当前节点 --> 对当前节点的左子节点进行前序遍历 --> 对当前节点的右子节点进行前序遍历，简称**根左右**。

代码实现：

```swift
func traversePreOrder(_ closure: (T) -> Void) {
    closure(value)
    leftChild?.traversePreOrder(closure)
    rightChild?.traversePreOrder(closure)
}
```

对上面创建的树形结构，用前序遍历:

```swift
three.traversePreOrder { print($0) }

// 结果
3
1
0
2
5
4
6
```

#### 后序遍历

遍历顺序：对当前节点的左子节点进行后序遍历 --> 对当前节点的右子节点进行后序遍历  --> 当前节点，简称**左右根**。

代码实现：

```swift
func traversePostOrder(_ closure: (T) -> Void) {
    leftChild?.traversePostOrder(closure)
    rightChild?.traversePostOrder(closure)
    closure(value)
}
```

对上面创建的树形结构，用后序遍历:

```swift
three.traversePostOrder { print($0) }

// 结果
0
2
1
4
6
5
3
```

如何区分这三种遍历方法？看根部遍历的顺序即可：根部在前，则为前序遍历；根部在中，则为中序遍历；根部在后，则为后序遍历。

### [完整代码 >>](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Tree/BinaryTree.swift)

### 参考资料

> [Data Structures and Algorithms in Swift](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift) --- [raywenderlich.com](https://www.raywenderlich.com/)，如果想看原版书籍，请点击链接购买。
