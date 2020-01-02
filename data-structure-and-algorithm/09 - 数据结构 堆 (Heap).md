堆，其实是一个完整的二叉树（除了叶节点外，其他节点都有左右子节点）。堆有以下两种形式：

- 最大堆：根部元素的值最大，越往下，值越小。例如

```
         10 
       /   \     
      7     9  
     / \   / \  
    6   4 5   3   
```

- 最小堆：根部元素的值最小，越往下，值越大。例如：

```
         1 
       /   \     
      2     3  
     / \   / \  
    6   8 9   7   
```

### 实现

虽然堆本质上是属于二叉树，但是我们不实用节点来实现，而是使用数组。因为堆中的元素有一定的优先级顺序，可以用数组来存储元素；并且在元素的添加移除等过程中需要频繁访问特定的位置的元素，数组正适合我们的需求。

根据以上分析，我们首先定义 `Heap` 如下：

```swift
struct Heap<Element: Equatable> {
    private(set) var elements: [Element] = []
    private let order: (Element, Element) -> Bool
    
    init(order: @escaping (Element, Element) -> Bool) {
        self.order = order
    }
    
    var isEmpty: Bool {
        return elements.isEmpty
    }
    
    var count: Int {
        return elements.count
    }
    
    var peek: Element? {
        return elements.first
    }
}

extension Heap: CustomStringConvertible {
    var description: String {
        return elements.description
    }
}
```

考虑到堆有两种形式，所以定义了一个 `order` 闭包，后续的相关操作可以根据它来判断堆的类型。另外还添加了一些常用属性和实现了 `CustomStringConvertible` 协议。

#### 数组如何表示堆

假设我们有以下堆数据结构：

```
        10       第一层
-----------------------
       /   \     
      7     9    第二层
-----------------------
     / \   / \  
    6   4 5   3  第三层
-----------------------
  /
 2               第四层
-----------------------
```

用数组可以表示如下：

```
索引     0      1     2     3     4     5     6     7
    	+-------------------------------------------------+
    	|  10  |  7  |  9  |  6  |  4  |  5  |  3  |  2   |
    	+-------------------------------------------------+
    	
    	|第一层 |   第二层   |         第三层         |第四层 |
    	|------|-----------|-----------------------|------|      

```

从上图我们可以总结出，如果给定一个元素的索引 `i`，那么：

- 这个元素的左子节点索引为 `2 * i + 1`，这个元素的右子节点索引为 `2 * i + 2`。
- 这个的父节点索引为 `(i - 1) / 2`

所以，添加获取左右子节点和父节点索引的方法，如下：

```swift
func leftChildIndex(ofParentAt index: Int) -> Int {
    return 2 * index + 1
}

func rightChildIndex(ofParentAt index: Int) -> Int {
    return 2 * index + 2
}

func parentIndex(ofChildAt index: Int) -> Int {
    return (index - 1) / 2
}
```

#### 移除元素

元素的移除分两种情况：1）移除根部元素；2）移除任意位置的元素。

##### 移除根部元素

以下面的最大堆为例，移除10：

```
  	 +------+
  	 |  10  |  
  	 +------+    
       /   \     
      7     9   
     / \   / \  
    6   4 5   3
  /
 2               
```

为了移除这个根部元素，我们首先将**根部元素**与最后一个元素的位置调换，变成：

```
      	+-----+
      	|  2  |  
      	+-----+    
         /   \     
        7     9   
       / \   / \  
      6   4 5   3
    /
+------+
|  10  |  
+------+    
```

调换成功后，就把最后一个元素移除，变成：

```
      	   2  
         /   \     
        7     9   
       / \   / \  
      6   4 5   3
```

这时需要检查当前的结构是否符合堆的特性。很明显，现在的根部元素是 `2`，小于它的子节点，所以我们把根部元素和它的左右子节点的较大者 `9` 位置调换，变成：

```
      	   9  
         /   \     
        7     2   
       / \   / \  
      6   4 5   3
```

再继续往下比较，这时2又比它的左右子节点小，继续把 `2` 和它的左右子节点的较大者位置 `5` 调换，变成：

```
      	   9  
         /   \     
        7     5   
       / \   / \  
      6   4 2   3
```

`2` 下面没有子节点了，所以这棵树最终调整完成，符合最大堆的要求。

实现代码如下：

```swift
extension Heap {
    mutating func removePeek() -> Element? {
        guard !isEmpty else { return nil }
        elements.swapAt(0, count - 1)
        defer {
            validate(from: 0)
        }
        return elements.removeLast()
    }
    
    private mutating func validateDown(from index: Int) {
        var parentIndex = index
        while true {
            let leftIndex = leftChildIndex(ofParentAt: parentIndex)
            let rightIndex = rightChildIndex(ofParentAt: parentIndex)
            var targetParentIndex = parentIndex
            
            if leftIndex < count &&
                order(elements[leftIndex], elements[targetParentIndex]) {
                targetParentIndex = leftIndex
            }
            
            if rightIndex < count &&
                order(elements[rightIndex], elements[targetParentIndex]) {
                targetParentIndex = rightIndex
            }
            
            if targetParentIndex == parentIndex {
                return
            }
            
            elements.swapAt(parentIndex, targetParentIndex)
            parentIndex = targetParentIndex
        }
    }
}
```

`removePeek()`：

- 首先判断堆是否为空，如果是直接返回 `nil`。
- 调换根元素和最后一个元素
- 移除最后一个元素
- `validate(from: 0)` 验证堆是否符合要求

`validate(from index: Int)`，while 循环里面：

- 首先得到左右子节点的索引
- `targetParentIndex` 用于记录最终需要跟父节点索引调换的索引
- 如果左子节点存在，并且左节点的值大于当前父节点的值，则更新 `targetParentIndex` 为 `leftIndex`
- 如果右子节点存在，并且右子节点的值大于当前父节点的值，则更新 `targetParentIndex` 为 `rightIndex`
- 如果 `targetParentIndex` 还是 `parentIndex`，意味着此时已经符合堆的特性，直接返回退出循环，否则替换 `parentIndex` 和 `targetParentIndex` 对应的值，并且把 `parentIndex` 替换为 `targetParentIndex`，继续循环

##### 移除任意元素

移除任意元素与移除根元素的思路类似，同样是把要移除的元素跟最后一个元素调换，让后在验证是否符合堆的特性。

实现代码如下：

```swift
mutating func remove(at index: Int) -> Element? {
    guard index < elements.count else { return nil }
    
    if index == elements.count - 1 {
        return elements.removeLast()
    } else {
        elements.swapAt(index, elements.count - 1)
        defer {
            validateDown(from: index)
            validateUp(from: index)
        }
        return elements.removeLast()
    }
}

private mutating func validateUp(from index: Int) {
    var childIndex = index
    var parentIndex = self.parentIndex(ofChildAt: childIndex)
    
    while childIndex > 0 &&
        order(elements[childIndex], elements[parentIndex]) {
            elements.swapAt(childIndex, parentIndex)
            childIndex = parentIndex
            parentIndex = self.parentIndex(ofChildAt: childIndex)
    }
}
```

`remove(at index: Int)`：

- 首先判断要移除的索引是否在数组范围内
- 如果要移除的是最后一个元素，则直接移除
- 如果是其他位置的元素，首先跟最后一个元素调换位置，接着移除最后一个元素
- 最后从上下两个方向进行验证是否符合堆的特性

`validateUp(from index: Int)`：

- 因为是向上验证，所以传入的 `index` 是一个子节点的索引，首先获得 `parentIndex`
- 在 while 循环的条件中，只有当子节点的索引存在，并且子节点的优先级高于父节点时，才需要调换子节点和父节点的值；在循环里面，先调换子节点和父节点的值，然后更新 `childIndex` 和 `parentIndex` 的值，进入下一次循环

#### 插入元素

有了上面的代码基础，插入元素就非常简单了，实现代码如下：

```swift
mutating func insert(_ element: Element) {
    elements.append(element)
    validateUp(from: elements.count - 1)
}
```

直接把插入的元素拼接到最后面，然后再向上验证堆的合法性就可以了。

#### 搜索元素

实现代码如下：

```swift
extension Heap {
    func index(of element: Element,
               searchingFrom index: Int = 0) -> Int? {
        if index >= count {
            return nil
        }
        if order(element, elements[index]) {
            return nil
        }
        if element == elements[index] {
            return index
        }
        
        let leftIndex = leftChildIndex(ofParentAt: index)
        if let i = self.index(of: element,
                              searchingFrom: leftIndex) {
            return i
        }
        
        let rightIndex = rightChildIndex(ofParentAt: index)
        if let i = self.index(of: element,
                              searchingFrom: rightIndex) {
            return i
        }
        
        return nil
    }
}
```

- 首先判断要查找的 `index` 是否在数组范围内，如果不在，直接返回 `nil`
- 因为搜索的实现要用到递归，所以方法中有一个 `searchingFrom` 参数，这个意思是告诉方法从哪个位置开始往后搜索。所以我们用 `order(element, elements[index])` 判断要查找的元素是否高于当前 `index` 对应元素的优先级，如果是，那么要查找的元素肯定不在后面，直接返回 `nil`
- 如果当前 `index` 对应的值等于要查找的值，则返回 `index`
- 从左分支用递归继续查找，如果找到则返回对饮的索引
- 从右分支用递归继续查找，如果找到则返回对饮的索引
- 以上步骤都没有找到，说明要查找的元素不存在，返回 `nil`

#### 测试

堆的代码实现就完成了，我们来测试一下：

##### 插入

```swift
var heap = Heap<Int>(order: >)
for i in 1...7 {
    heap.insert(i)
}
print(heap)

// 结果
[7, 4, 6, 1, 3, 2, 5]
```

对应的树结构为：

```
      	   7  
         /   \     
        4     6   
       / \   / \  
      1   3 2   5
```

符合堆的特性。

##### 移除根部元素

```swift
while !heap.isEmpty {
    print(String(describing: heap.removePeek()))
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

从最大的元素删除到最小的元素，符合堆的特性。

##### 移除任意元素

```swift
let index = heap.index(of: 3) ?? 0
print(String(describing: heap.remove(at: index)))
print(heap)

// 结果
Optional(3)
[7, 5, 6, 1, 4, 2]
```

移除元素3，并且移除后，依然符合堆的特性。

### 总结

堆数据结构非常适合用来跟踪最大值或者最小值，因为获取 `peek` 的时间复杂度为 `O(1)`。在搜索性能方面，时间复杂度为 `O(n)`， 比二叉搜索树的 `O(log n)` 要差很多。另外堆的插入和移除的时间复杂度都为 `O(log n)`。

### [完整代码 >>](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Heap/Heap.swift)

### 参考资料

> [Data Structures and Algorithms in Swift](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift) --- [raywenderlich.com](https://www.raywenderlich.com/)，如果想看原版书籍，请点击链接购买。
