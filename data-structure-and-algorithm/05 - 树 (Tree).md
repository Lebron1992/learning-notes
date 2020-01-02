虽然我们在iOS开发中，很少接触树结构，但他是一种极其重要的数据结构。树也分很多种类：1）二叉树；2）二叉搜索树；3）平衡搜索树等。

在这篇文章里，我们先实现一个简单的树。

### 专业术语

在写代码之前，先学习一些树结构中的专业术语。

#### 节点

树，是由节点组成的，每个节点存储了一些数据和他的子节点。图中的每个圆圈都是节点。

![](http://upload-images.jianshu.io/upload_images/2057254-b3cf9d500c0bcdb7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 父节点和子节点

在观察树的时候，我们是从上往下看的。除了最顶部的节点以外，其他节点的上面都连接着一个节点，这个节点就叫父节点，而且每一个节点只有一个父节点；下面的节点就叫子节点。

![](http://upload-images.jianshu.io/upload_images/2057254-8ba38c56a99d7ecf.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 根

树中最顶部的节点称为**根**，唯一一个没有父节点的节点。

![](http://upload-images.jianshu.io/upload_images/2057254-1f1abd130c7ad320.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 叶节点

没有子节点的节点。 

![](http://upload-images.jianshu.io/upload_images/2057254-f223c13178ef83d5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 实现

#### TreeNode

根据刚刚的学习的知识点，我们可以写一个简单的`TreeNode`：

```swift
final class TreeNode<T> {
    var value: T
     private(set) var children: [TreeNode] = []
    
    init(_ value: T) {
        self.value = value
    }
    
    func addChild(_ child: TreeNode) {
        children.append(child)
    }
}
```

- `value`：存储当前节点的值
- `children`：存储所有子节点
- `addChild(_:)`：添加子节点的方法

我们来尝试使用一下：

```swift
let products = TreeNode("Products")

let phone = TreeNode("Phone")
let computer = TreeNode("Computer")

products.addChild(phone)
products.addChild(computer)
```

上面的代码创建了这样一个树结构：

![](http://upload-images.jianshu.io/upload_images/2057254-eb5ba108593e9863.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 遍历算法

不同于遍历数组，遍历树结构会比较复杂一些：数组从头到尾遍历，但是树结构没有头和尾之说。在树结构中，我们可以使用**深度优先遍历**和**层次顺序遍历**。

##### 深度优先遍历

从根开始，从最左边的树枝一直往下遍历，遍历完之后再遍历右边的树枝，直到所有树枝便利完为止。实现的代码如下：

```swift
func traverseDepthFirst(_ closure: (TreeNode) -> Void) {
    closure(self)
    children.forEach {
        $0.traverseDepthFirst(closure)
    }
}
```

我们来看一个例子：

```swift
let products = TreeNode("Products")

let phone = TreeNode("Phone")
let computer = TreeNode("Computer")

products.addChild(phone)
products.addChild(computer)

let iPhone8 = TreeNode("iPhone 8")
let iPhone8Plus = TreeNode("iPhone 8 Plus")
let iPhoneX = TreeNode("iPhone X")

let macBookPro = TreeNode("MacBook Pro")
let iMac = TreeNode("iMac")
let iMacPro = TreeNode("iMacPro")

phone.addChild(iPhone8)
phone.addChild(iPhone8Plus)
phone.addChild(iPhoneX)

computer.addChild(macBookPro)
computer.addChild(iMac)
computer.addChild(iMacPro)
```

上面的代码创建了这样一个树结构：

![](http://upload-images.jianshu.io/upload_images/2057254-8b59206d9e668da5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

遍历：

```swift
products.traverseDepthFirst { print($0.value) }

// 结果
Products
Phone
iPhone 8
iPhone 8 Plus
iPhone X
Computer
MacBook Pro
iMac
iMacPro
```

#### 层次顺序遍历

下图演示了如何区分层次：

![](http://upload-images.jianshu.io/upload_images/2057254-4213ae833dae85c0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码实现如下：

```swift
func traverseLevelOrder(_ closure: (TreeNode) -> Void) {
    closure(self)
    var queue = Queue<TreeNode>()
    children.forEach { queue.enqueue($0) }
    while let node = queue.dequeue() {
        closure(node)
        node.children.forEach { queue.enqueue($0) }
    }
}
```

把每一层的元素添加到队列中，保证树中的节点按层次顺序遍历。`Queue`的实现代码在上一篇文章: [【数据结构与算法 - Swift实现】04 - 队列 (Queue)](https://wp.me/p9EfvY-9F)，或者查看我在 GitHub 的 [demo](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Queue/Queue.swift)。

遍历：

```swift
products.traverseLevelOrder { print($0.value) }

// 结果
Products
Phone
Computer
iPhone 8
iPhone 8 Plus
iPhone X
MacBook Pro
iMac
iMacPro
```

### 总结

这篇文章介绍了树结构的一些专业术语；演示了用代码如何创建树结构；用两种方式实现对树的遍历。但是文章中介绍的树包含的范围非常广，对树的结构没有要求，接下来的文章我们将一起看看一些特别的、有实际用途的树。

### [完整代码 >>](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Tree/Tree.swift)

### 参考资料

> [Data Structures and Algorithms in Swift](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift) --- [raywenderlich.com](https://www.raywenderlich.com/)，如果想看原版书籍，请点击链接购买。
