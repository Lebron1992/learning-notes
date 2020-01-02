二叉搜索树 (**BST**)可以提高查找、插入和删除的效率，时间复杂度都为`O(log n)`。成为二叉搜索树必须满足下面两个条件：

- 左子节点的值必须小于它的父节点的值
- 右子节点的值必须大于或等于它的父节点的值

### 二叉搜索树和数组的对比

我们先来看看二叉搜索树和数组在查找、插入和删除操作的效率对比。假设我们有下列数组和二叉搜索树：

![](http://upload-images.jianshu.io/upload_images/2057254-50e5520955518b02.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 查找

假设我们要在这两个数据结构中找到 6：
- **数组**：需要拿数组的每个元素与6去对比，时间复杂度为`O(n)`。
- **二叉搜索树**：首先拿树的根节点去对比，6大于3，所以可以确定6肯定不在左子节点，而在右子节点，根据这个思路，我们只需要对比三次就能找到6，时间复杂度为`O(log n)`。

#### 插入

假设我们要在这两个数据结构中插入 -1：
- **数组**：在数组的最前面插入-1，已有的元素需要往后移动，时间复杂度为`O(n)`。
- **二叉搜索树**：首先拿树的根节点去对比，-1小于3，所以可以确定-1要插入到左子节点这边，根据这个思路，我们只需要对比三次就能在正确的位置插入-1，时间复杂度为`O(log n)`。

![](http://upload-images.jianshu.io/upload_images/2057254-d365672a12b57d4d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 移除

假设我们要在这两个数据结构中移除 4：
- **数组**：与插入一样，移除其中的元素，后面的元素要往前移动，时间复杂度为`O(n)`。
- **二叉搜索树**：图中二叉树的4没有子节点，直接把它移除即可。如果他有子节点，会稍微复杂一些，等会会讲到这个，但是时间复杂度还是`O(log n)`。

![](http://upload-images.jianshu.io/upload_images/2057254-f1af4a05deb6e2a1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 实现

首先简单定义`BinarySearchTree`：

```swift
struct BinarySearchTree<Element: Comparable> {
    private(set) var root: BinaryTreeNode<Element>?
    init() {  }
}
```

因为二叉搜索树的元素有大小之分，所以`Element`必须遵循`Comparable`。这里我们用到了[【数据结构与算法 - Swift实现】06 - 二叉树 (Binary Tree)](https://wp.me/p9EfvY-9W)的`BinaryTreeNode`，可以到我在 github 的 [demo](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Tree/BinaryTree.swift) 查看。

#### 查找

判断树中是否包含某个值：

```swift
extension BinarySearchTree {
    func contains(_ value: Element) -> Bool {
        var current = root
        while let node = current {
            if node.value == value {
                return true
            }
            if value < node.value {
                current = node.leftChild
            } else {
                current = node.rightChild
            }
        }
        return false
    }
}
```

使用`while`循环，从 `root` 开始，如果当前节点的值等于要查找的值，则返回 `true`；如果要查找的值小于当前节点的值，把当前节点的左节点作为下一轮循环的节点，否则把当前节点的右节点作为下一轮循环的节点。

#### 插入

```swift
extension BinarySearchTree {
    mutating func insert(_ value: Element) {
        root = insert(from: root, value: value)
    }
    
    private func insert(from node: BinaryTreeNode<Element>?,
                        value: Element) -> BinaryTreeNode<Element> {
        guard let node = node else {
            return BinaryTreeNode(value)
        }
        if value < node.value {
            node.leftChild = insert(from: node.leftChild, value: value)
        } else {
            node.rightChild = insert(from: node.rightChild, value: value)
        }
        return node
    }
}
```

在私有的`insert`方法实现中：

- 首先判断`node`是否为 `nil`，如果是 `nil`，返回一个新的 `node`
- 如果要插入的值小于当前 node 的值，利用递归把它插到左子节点；否则插到右子节点
- 最后返回当前 `node`

我们来使用一下这个插入方法：

```swift
var bst1 = BinarySearchTree<Int>()
bst1.insert(0)
bst1.insert(2)
bst1.insert(3)
bst1.insert(5)
bst1.insert(7)
bst1.insert(9)
```

上面的代码创建了这样一个树结构：

![](http://upload-images.jianshu.io/upload_images/2057254-445590c2f9ea16d8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们在来看一个例子：

```swift
var bst2 = BinarySearchTree<Int>()
bst2.insert(4)
bst2.insert(2)
bst2.insert(6)
bst2.insert(1)
bst2.insert(3)
bst2.insert(5)
bst2.insert(7)
```

上面的代码创建了这样一个树结构：

![](http://upload-images.jianshu.io/upload_images/2057254-5a3468675222cc66.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对比刚刚创建的两个树结构，可以发现`bst1`是不平衡的，而`bst2`则是完美地平衡。下一篇文章我们再讨论是否平衡的问题。

#### 移除

移除操作比较复杂，需要考虑这几种情况：1）移除叶节点；2）移除只有一个子节点的节点；3）移除有两个子节点的节点。下面我们一个个来看。

##### 移除叶节点

这是最简单的情况，直接把它移除即可

![](http://upload-images.jianshu.io/upload_images/2057254-26f5e0c45b3bd1e4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 移除只有一个子节点的节点

移除只有一个子节点的节点后，要它的子节点连接到它的父节点。

![](http://upload-images.jianshu.io/upload_images/2057254-1baa8d0bcf27c957.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 移除有两个子节点的节点

如下图，要移除3：直接移除3之后，1和5就没有父节点了，这时我们应该怎么办？

如果把1和5连接到9的话，9就有三个子节点了，不符合二叉树的要求。

![](http://upload-images.jianshu.io/upload_images/2057254-27f564b9723e5e9a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通常我们用3右边的最小值4来代替3的位置，替代之后，依然符合二叉搜索树的要求。树结构变为：

![](http://upload-images.jianshu.io/upload_images/2057254-033720e7e1b31d2d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过分析之后，我们来看看具体实现：

```swift
extension BinaryTreeNode {
    var minNode: BinaryTreeNode {
        return leftChild?.minNode ?? self
    }
}

extension BinarySearchTree {
    mutating func remove(_ value: Element) {
        root = remove(node: root, value: value)
    }
    
    private func remove(node: BinaryTreeNode<Element>?,
                value: Element) -> BinaryTreeNode<Element>? {
        
        guard let node = node else { return nil }
        
        if node.value == value {
            // 左右子节点都为空
            if node.leftChild == nil && node.rightChild == nil {
                return nil
            }
            // 左右子节点中，有其中一个为空
            if node.leftChild == nil {
                return node.rightChild
            }
            if node.rightChild == nil {
                return node.leftChild
            }
            // 左右子节点都不为空
            node.value = node.rightChild!.minNode.value // 把当前 `node` 的值更新为右子节点的最小值
            node.rightChild = remove(node: node.rightChild, value: node.value) // 把右子节点的最小值删除
            
        } else if value < node.value {
            node.leftChild = remove(node: node.leftChild, value: value)
        } else {
            node.rightChild = remove(node: node.rightChild, value: value)
        }
        
        return node
    }
}
```

- 1）因为我们要得到被删除节点右边的最小值，所以通过扩展定义了一个计算属性，`minNode`；
- 2）在私有的`remove`方法实现中，实现思路跟插入操作很类似，都是通过递归来实现，我们主要看看当`node.value == value`时的思路：
  - `node.leftChild == nil && node.rightChild == nil`：左右子节点都为空，当前节点为叶节点，直接通过返回`nil`来删除
  - `node.leftChild == nil` 和 `node.rightChild == nil`：左右子节点中，有其中一个为空，返回另外一个不为空的子节点，当前的 `node` 就会被删除
  - 左右子节点都不为空：把当前 `node` 的值更新为右子节点的最小值，然后把右子节点的最小值删除
  
验证一下我们刚刚的实现：

```swift
var bst = BinarySearchTree<Int>()
(0...12).forEach {
    bst.insert($0)
}
bst.remove(3)
bst.root?.traverseInOrder { print($0) }

// 结果
0
1
2
4
5
6
7
8
9
10
11
12
```

先插入0到12，接着把3移除，最后用中序遍历(左根右)，3被移除，并且所有的节点还是按照对小到大排列，说明我们写的移除方法没错。

### 总结

二叉搜索树是一个非常强大的数据结构，在处理有序的数据时有很高的效率。查找、插入和移除的时间复杂度都为`O(log n)`。

### [完整代码 >>](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Tree/BinarySearchTree.swift)

### 参考资料

> [Data Structures and Algorithms in Swift](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift) --- [raywenderlich.com](https://www.raywenderlich.com/)，如果想看原版书籍，请点击链接购买。
