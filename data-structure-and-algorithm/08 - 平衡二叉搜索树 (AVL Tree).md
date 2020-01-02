AVL树，第一个自平衡的二叉搜索树，是Adelson-Velsky 和 Evgenii Landis 在1962年提出的，所以 **AVL**来自于他们名字的简称。

### 怎样才算是平衡？

下图演示了三种平衡状态：1）完美平衡；2）不错的平衡；3）不平衡

![](http://upload-images.jianshu.io/upload_images/2057254-ba73b5b0423f0f91.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **完美平衡**：从上到下，除了最下面一层节点外，其他节点都有左右子节点。这是最理想的平衡状态。在实际中是比较难达到的。
- **不错的平衡**：除了底层的节点外，其他节点都有左右子节点。这种状态是我们多数情况下能实现的最好的状态。
- **不平衡**：除了底层的节点外，还有其他节点的子节点没有填满。这种状态的性能会很差。

### 实现

AVL树的本质还是一个二叉搜索树，只是多了一个平衡条件而已，所以这里用到的代码可以直接从平衡搜索树那里拿过来用：把[BinaryTree.swift](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Tree/BinaryTree.swift)里面的 `BinaryTreeNode` 重命名为 `AVLNode`；把[BinarySearchTree.swift](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Tree/BinarySearchTree.swift)里面的 `BinarySearchTree` 重命名为 `AVLTree`。

另外，为了能方便观看树的结构，给 `AVLNode` 和 `AVLTree` 实现了 `CustomStringConvertible`：

```swift
extension AVLNode: CustomStringConvertible {
    var description: String {
        return diagram(for: self)
    }
    
    private func diagram(for node: AVLNode?,
                         top: String = "",
                         root: String = "",
                         bottom: String = "") -> String {
        guard let node = node else {
            return root + "nil\n"
        }
        if node.leftChild == nil && node.rightChild == nil {
            return root + "\(node.value)\n"
        }
        return diagram(for: node.rightChild,
                       top: top + " ",
                       root: top + "┌───",
                       bottom: top + "| ")
        + root + "\(node.value)\n"
        + diagram(for: node.leftChild,
                  top: bottom + "| ",
                  root: bottom + "└───",
                  bottom: bottom + " ")
    }
}

extension AVLTree: CustomStringConvertible {
    var description: String {
        return root?.description ?? "empty tree"
    }
}


// 打印的效果如下
 ┌───6
┌───5
| └───4
3
| ┌───2
└───1
 └───0
```

#### 二叉搜索树平衡的条件

首先我们给 `AVLNode` 添加一个 `height` 属性，它指的是从当前节点到叶节点的最长距离。

```swift
var height = 0
```

例如以下各节点的高度为：

![](http://upload-images.jianshu.io/upload_images/2057254-cdfd688347138df9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

二叉搜索树平衡的条件是树中所有节点的左子节点高度和右子节点高度的差的绝对值不大于1，我们把这个条件称为 **balance factor**，平衡因子。根据以上分析，添加以下属性：

```swift
var leftHeight: Int {
    return leftChild?.height ?? -1
}
var rightHeight: Int {
    return rightChild?.height ?? -1
}
var balanceFactor: Int {
    return leftHeight - rightHeight
}
```

如果子节点为空，则默认为-1。

例如以下各节点的平衡因子和高度，左边的数字是平衡因子，右边的是高度：

![](http://upload-images.jianshu.io/upload_images/2057254-9fef38b5fc068806.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图中20和40的平衡因子绝对值大于1，所以这棵树不平衡。下面我们看看如何通过旋转使它达到平衡。

#### 旋转

总共有四种类型的旋转：左旋转、左右旋转、右旋转、右左旋转。

##### 左旋转

我们先看一个左旋转示意图，图中的字母表示节点的名称，不是节点的值：

![](http://upload-images.jianshu.io/upload_images/2057254-e75b28800c25fcf9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在旋转后，我们要保证用中序遍历时，不改变节点的遍历顺序。

下面是代码实现：

```swift
private func leftRotate(_ node: AVLNode<Element>) -> AVLNode<Element> {
    guard let pivot = node.rightChild else { return node }
    node.rightChild = pivot.leftChild
    pivot.leftChild = node
    node.height = max(node.leftHeight, node.rightHeight) + 1
    pivot.height = max(pivot.leftHeight, pivot.rightHeight) + 1
    return pivot
}
```

- 把被旋转的节点(A)的右节点(B)作为轴，这个右节点最终将替代被旋转的节点作为根节点
- 被旋转的节点(A)的右节点更新为轴的左节点(X)
- 轴(B)的左节点更新为被旋转的节点(A)
- 最后更新被旋转节点和轴的高度

我们之前那个不平衡的例子可以通过左旋转达到平衡状态：

![](http://upload-images.jianshu.io/upload_images/2057254-215b64fef41894e2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 右旋转

右旋转跟左旋转相反。我们先看一个右旋转示意图，图中的字母表示节点的名称，不是节点的值：

![](http://upload-images.jianshu.io/upload_images/2057254-6862d092fb92e7f8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面是代码实现：

```swift
private func rightRotate(_ node: AVLNode<Element>) -> AVLNode<Element> {
    guard let pivot = node.rightChild else { return node }
    node.leftChild = pivot.rightChild
    pivot.rightChild = node
    node.height = max(node.leftHeight, node.rightHeight) + 1
    pivot.height = max(pivot.leftHeight, pivot.rightHeight) + 1
    return pivot
}
```

代码实现与左旋转类似，只不过是`leftChild`和`rightChild`对调了而已。

##### 右左旋转

上面的左旋转和右旋转，旋转的节点都是在左边或者右边。如果我们有以下第一个不平衡的树，通过右左旋转后，可以达到平衡：

![](http://upload-images.jianshu.io/upload_images/2057254-5dea6548a3e092cc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 首先对25进行右旋转，变成图二
- 图二中导致不平衡的节点都在右边，所以可以对20进行左旋转，最终达到图三的平衡状态

代码实现如下：

```swift
private func rightLeftRotate(_ node: AVLNode<Element>) -> AVLNode<Element> {
    guard let rightChild = node.rightChild else { return node }
    node.rightChild = rightRotate(rightChild)
    return leftRotate(node)
}
```

##### 左右旋转

左右旋转与右左旋转相反，对右左旋转的例子稍作修改：

![](http://upload-images.jianshu.io/upload_images/2057254-f760010ef2142c29.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 首先对10进行左旋转，变成图二
- 图二中导致不平衡的节点都在左边，所以可以对20进行右旋转，最终达到图三的平衡状态

代码实现如下：

```swift
private func leftRightRotate(_ node: AVLNode<Element>) -> AVLNode<Element> {
    guard let leftChild = node.leftChild else { return node }
    node.leftChild = leftRotate(leftChild)
    return rightRotate(node)
}
```

#### 平衡的处理

我们已经写好了四种旋转的实现，下面根据平衡因子来决定何时调用哪一种旋转。

- 平衡因子为2，意味着左边的节点比右边多，需要使用右旋转或者左右旋转。
- 平衡因子为-2，意味着右边的节点比左边多，需要使用左旋转或者右左旋转。

根据以上分析，平衡相关代码如下：

```swift
func balanced(_ node: AVLNode<Element>) -> AVLNode<Element> {
    switch node.balanceFactor {
    case 2:
        if let leftChild = node.leftChild,
            leftChild.balanceFactor == -1 {
            
            return leftRightRotate(node)
        } else {
            return rightRotate(node)
        }
    case -2:
        if let rightChild = node.rightChild,
            rightChild.balanceFactor == 1 {
            
            return rightLeftRotate(node)
        } else {
            return leftRotate(node)
        }
    default:
        return node
    }
}
```

#### 更新insert方法

```swift
private func insert(from node: AVLNode<Element>?,
                    value: Element) -> AVLNode<Element> {
    guard let node = node else {
        return AVLNode(value)
    }
    if value < node.value {
        node.leftChild = insert(from: node.leftChild, value: value)
    } else {
        node.rightChild = insert(from: node.rightChild, value: value)
    }
    let balancedNode = balanced(node)
    balancedNode.height = max(balancedNode.leftHeight,
                              balancedNode.rightHeight) + 1
    return balancedNode
}
```

每插入一个元素之后，重新平衡一下当前节点，并更新平衡后的节点的高度。

测试一下：

```swift
var tree = AVLTree<Int>()
for i in 0...6 {
    tree.insert(i)
}
print(tree)

// 结果
 ┌───6
┌───5
| └───4
3
| ┌───2
└───1
 └───0
```

无论插入的顺序如何，自己自动平衡。

#### 更新 remove 方法

跟 insert 类型，每移除一个元素之后，重新平衡一下当前节点，并更新平衡后的节点的高度：

```swift
private func remove(node: AVLNode<Element>?,
                    value: Element) -> AVLNode<Element>? {
    
    guard let node = node else { return nil }
    
    if node.value == value {
        
        if node.leftChild == nil && node.rightChild == nil {
            return nil
        }
        if node.leftChild == nil {
            return node.rightChild
        }
        if node.rightChild == nil {
            return node.leftChild
        }
        node.value = node.rightChild!.minNode.value
        node.rightChild = remove(node: node.rightChild, value: node.value)
        
    } else if value < node.value {
        node.leftChild = remove(node: node.leftChild, value: value)
    } else {
        node.rightChild = remove(node: node.rightChild, value: value)
    }
    
    let balancedNode = balanced(node)
    balancedNode.height = max(balancedNode.leftHeight,
                              balancedNode.rightHeight) + 1
    return balancedNode
}
```

测试一下：

```swift
var tree = AVLTree<Int>()
for i in 0...6 {
    tree.insert(i)
}
tree.remove(3)
tree.remove(5)
print(tree)

// 结果
┌───6
4
| ┌───2
└───1
 └───0
```

移除元素后，自动平衡。

### 总结

AVL 树的自动平衡功能比较复杂，需要一定的时间去理解。它的效率很高，插入和移除操作的时间复杂度都是 `O(log n)`。AVL 树是第一个自动平衡的树结构，与 AVL 类似的树还有**红黑树**和**伸展树**，有兴趣的话，可以自己去了解下！

### [完整代码 >>](https://github.com/Lebron1992/swift-algorithm-demo/tree/master/swift-algorithm/Tree/AVL%20Tree)

### 参考资料

> [Data Structures and Algorithms in Swift](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift) --- [raywenderlich.com](https://www.raywenderlich.com/)，如果想看原版书籍，请点击链接购买。
