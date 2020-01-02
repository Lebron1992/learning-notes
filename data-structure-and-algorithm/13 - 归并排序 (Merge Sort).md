归并排序算法是一个效率很高的排序算法，时间复杂度为 `O(n log n)`。它的算法思想是先分组，各小组排好序后，再把结果合并。

### 原理解析

以下面的数组为例来讲解一下归并排序的原理：

```
	+--------+    +--------+    +--------+    +--------+
	|        |    |        |    |        |    |        |
	|    8   |    |    7   |    |   14   |    |    5   |
	|        |    |        |    |        |    |        |
	+--------+    +--------+    +--------+    +--------+
```

1. 首先把数组分成两半：

```
	+--------+  +--------+        +--------+  +--------+
	|        |  |        |        |        |  |        |
	|    8   |  |    7   |        |   14   |  |    5   |
	|        |  |        |        |        |  |        |
	+--------+  +--------+        +--------+  +--------+
```

2. 继续把上面的两个子数组分别分成两半：

```
	+--------+        +--------+        +--------+        +--------+
	|        |        |        |        |        |        |        |
	|    8   |        |    7   |        |   14   |        |    5   |
	|        |        |        |        |        |        |        |
	+--------+        +--------+        +--------+        +--------+
```

到目前为止，每个子数组只有一个元素，不能再继续往下分组，分组的操作到此完成。

3. 子数组按照小到大的顺序合并：

```
	+--------+  +--------+        +--------+  +--------+
	|        |  |        |        |        |  |        |
	|    7   |  |    8   |        |    5   |  |   14   |
	|        |  |        |        |        |  |        |
	+--------+  +--------+        +--------+  +--------+
```

4. 再继续合并：

```
	+--------+    +--------+    +--------+    +--------+
	|        |    |        |    |        |    |        |
	|    5   |    |    7   |    |    8   |    |   14   |
	|        |    |        |    |        |    |        |
	+--------+    +--------+    +--------+    +--------+
```

完成排序。

### 实现

```swift
func mergeSort<Element: Comparable>(_ array: [Element]) -> [Element] {
    guard array.count > 1 else {
        return array
    }
    let middle = array.count / 2
    let left = mergeSort(Array(array[..<middle]))
    let right = mergeSort(Array(array[middle...]))
    return merge(left, right)
}

private func merge<Element: Comparable>(_ left: [Element],
                                        _ right: [Element]) -> [Element] {
    var leftIndex = 0
    var rightIndex = 0
    var result: [Element] = []
    
    while leftIndex < left.count && rightIndex < right.count {
        let leftElement = left[leftIndex]
        let rightElement = right[rightIndex]
        
        if leftElement < rightElement {
            result.append(leftElement)
            leftIndex += 1
        } else if leftElement > rightElement {
            result.append(rightElement)
            rightIndex += 1
        } else {
            result.append(leftElement)
            leftIndex += 1
            result.append(rightElement)
            rightIndex += 1
        }
    }
    
    if leftIndex < left.count {
        result.append(contentsOf: left[leftIndex...])
    }
    if rightIndex < right.count {
        result.append(contentsOf: right[rightIndex...])
    }
    
    return result
}
```

**`mergeSort`方法**：利用递归把数组分割到最小，然后调用 `merge`方法

**`merge`方法**：

  - `leftIndex` 和 `rightIndex` 记录左右两个子数组当前索引；`result` 用于存储左右两个子数组合并的结果
  - `while` 循环里面，判断当前左右子索引对应元素的大小，谁小就先把谁添加到 `result` 中，如果一样大，两个都添加
  - `while` 循环结束后，如果左右子数组还有剩下的元素，则直接添加到最后面。
  

#### 测试

```swift
let ints = [4,5,5467,73,234,678,87,989]
let sortedInts = mergeSort(ints)
print(sortedInts)

// 结果
[4, 5, 73, 87, 234, 678, 989, 5467]
```

结果正确。

### [完整代码 >>](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Sort/MergeSort.swift)

### 参考资料

> [Data Structures and Algorithms in Swift](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift) --- [raywenderlich.com](https://www.raywenderlich.com/)，如果想看原版书籍，请点击链接购买。
