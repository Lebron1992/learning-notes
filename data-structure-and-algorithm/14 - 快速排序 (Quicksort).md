像归并排序一样，快速排序也是通过分组的思想来实现。

快速排序中最重要的环节是支点(pivot)的选择，也就是我们如何去把数组进行分组。支点把数组分成三个部分：`[小于支点的元素 | 支点 | 大于支点的元素]`。

这篇文章将会选择两种分组方式来实现快速排序：1）lomuto 划分，以最后一个元素为支点；2）hoare 划分，以第一个元素为支点。

### Lomuto 划分

#### 划分原理

Lomuto 划分是以最后一个元素为支点。

我们以下面这个乱序的数组为例：`[13, 2, 5, 8, 10, 32, 10]`。

首先，我们把第一个元素的索引命名为 `low`，最后一个元素的索引命名为 `high`；`i` 用来记录经过一轮划分后小于或等于支点的元素个数，`j`是数组当前遍历的位置。支点是最后一个元素 `10`，选定了这个支点后，在当前这一轮划分支点一直不变。

```
  0    1   2   3   4    5    6
[ 13,  2,  5,  8,  10,  32,  10]
  low                       high
  i
  j
```

1. 第一个元素`13` 大于支点`10`，`i` 的值也不变，`j` 加 `1`，结果为：

```
  0    1   2   3   4    5    6
[ 13,  2,  5,  8,  10,  32,  10]
  low                        high
  i
       j
```

2. 第二个元素 `2` 小于支点`10`，互换 `i` 和 `j` 对应的元素，`i` 加 `1`，`j` 加 `1`，结果为：

```
  0    1   2   3   4    5    6
[ 2,  13,  5,  8,  10,  32,  10]
  low                        high
       i
           j
```

3. 第三个元素 `5` 小于支点`10`，互换 `i` 和 `j` 对应的元素，`i` 加 `1`，`j` 加 `1`，结果为：

```
  0    1   2   3    4    5    6
[ 2,   5,  13,  8,  10,  32,  10]
  low                        high
            i
                j
```

4. 第四个元素 `8` 小于支点`10`，互换 `i` 和 `j` 对应的元素，`i` 加 `1`，`j` 加 `1`，结果为：

```
  0    1   2   3    4    5    6
[ 2,   5,  8,  13,  10,  32,  10]
  low                        high
                i
                     j
```

5. 第五个元素 `10` 等于支点`10`，互换 `i` 和 `j` 对应的元素，`i` 加 `1`，`j` 加 `1`，结果为：

```
  0    1   2   3    4    5    6
[ 2,   5,  8,  10,  13,  32,  10]
  low                        high
                     i
                          j
```

6. 第六个元素 `32` 大于支点`10`，`i` 不变，`j` 已经指向数组中除支点外的最后一个元素。结果为：

```
  0    1   2   3    4    5    6
[ 2,   5,  8,  10,  13,  32,  10]
  low                        high
                     i
```

7. 把支点跟 `i` 对应元素交换位置，第一轮的划分到此结束，i 左边的元素都小于等于支点 `10`；i 右边的元素，除最后一个元素外，其他都是没有排序的，这里因为数组比较短，所以只有一个 `32`，如果最开始在数组前面加多一个大于支点的元素 `11`，那么 `11` 也是在这个未排序的范围内。结果为：

```
  0    1   2   3    4    5    6
[ 2,   5,  8,  10,  10,  32,  13]
  low                        high
                     i
```

#### 划分的代码实现

```swift
func lomutoPartition<T: Comparable>(_ array: inout [T],
                                    low: Int,
                                    high: Int) -> Int {
    // 最后一个元素为支点
    let pivot = array[high]
    
    // i 的初始值为 low
    var i = low
    for j in low..<high {
        if array[j] <= pivot { // 当前遍历的值小于或等于支点
            array.swapAt(i, j) // 交换 i 和 j 对应的值
            i += 1 // i 加 1
        }
    }
    
    // 互换支点和 i 对应的值
    array.swapAt(i, high)
    return i
}
```

#### Lomuto 划分的排序实现

划分完成之后，实现排序的代码就非常简单了。代码如下：

```swift
func lomutoQuicksort<T: Comparable>(_ array: inout [T],
                                    low: Int,
                                    high: Int) {
    guard low < high else {
        return
    }
    let pivot = lomutoPartition(&array, low: low, high: high)
    lomutoQuicksort(&array, low: low, high: pivot - 1)
    lomutoQuicksort(&array, low: pivot + 1, high: high)
}
```

测试后，结果正确：

```swift
var list = [13, 2, 5, 8, 10, 32, 10]
lomutoQuicksort(&list, low: 0, high: list.count - 1)
print(list)

// 结果：
[2, 5, 8, 10, 10, 13, 32]
```

### Hoare 划分

#### 划分原理

Hoare 划分是以第一个元素为支点。

我们还是以刚刚那个数组为例：`[13, 2, 5, 8, 10, 32, 10]`。

首先，我们把第一个元素 `13` 作为支点，选定了这个支点后，在当前这一轮划分支点一直不变；`i` 的初始值为最小索引 减 `1`，`j` 的初始值为最大索引加 `1`。将 `i` 不断往中间移动，直到 `i` 对应的值不小于支点；将 `j` 不断往中间移动，直到 `j` 对应的值不大于支点；然后将 `i` 和 `j` 对应的值互换；重复以上步骤，直到 `i` 和 `j` 重叠，然后进入下一轮划分。

```
  0    1   2   3   4    5    6
[ 13,  2,  5,  8,  10,  32,  10]
i                                j
```

1. 将 `i` 不断往中间移动，`i` 移动到对应的元素为 `13`，不小于支点 `13`，所以 `i` 的值暂时为 `0`； 将 `j` 不断往中间移动，`j` 移动到对应的元素为 `10`，不大于支点 `13`，所以 `j` 的值暂时为 `6`，将 `i` 和 `j` 对应的值互换，结果如下：

```
  0    1   2   3   4    5    6
[ 10,  2,  5,  8,  10,  32,  13]
  i                           j
``` 

2. 将 `i` 继续往中间移动，`i` 移动到对应的元素为 `32`，不小于支点 `13`，所以 `i` 的值暂时为 `5`； 将 `j` 不断往中间移动，`j` 移动到对应的元素为 `10`，不大于支点 `13`，所以 `j` 的值暂时为 `4`；结果如下：

```
  0    1   2   3   4    5    6
[ 10,  2,  5,  8,  10,  32,  13]
                        i                        
                    j
``` 

`i` 的值为 `5`， `j` 的值为 `4`，说明他们已经重叠。所以 `j` 前面的元素都是小于或等于支点的元素，第一轮划分已经完成。

#### 划分的代码实现

```swift
func hoarePartition<T: Comparable>(_ array: inout [T],
                                   low: Int,
                                   high: Int) -> Int {
    // 把第一个元素作为支点
    let pivot = array[low]
    // 把 i 往左移动1位，方便使用下面的 repeat while 语句
    var i = low - 1
    // // 把 j 往左移动1位，方便使用下面的 repeat while 语句
    var j = high + 1
    
    while true {
        // `i += 1` 无论如何都会至少执行1次，如果 i 对应的值小于支点，继续往右移动
        repeat { i += 1 } while array[i] < pivot
        // `j -= 1` 无论如何都会至少执行1次，如果 i 对应的值大于支点，继续往左移动
        repeat { j -= 1 } while array[j] > pivot
        if i < j { // i 小于 j，互换他们对应的值
            array.swapAt(i, j)
        } else { // i 和 j 已经重叠，返回 j
            return j
        }
    }
}
```

#### Hoare 划分的排序实现

```swift
func hoareQuicksort<T: Comparable>(_ array: inout [T],
                                   low: Int,
                                   high: Int) {
    guard low < high else {
        return
    }
    let pivot = hoarePartition(&array, low: low, high: high)
    hoareQuicksort(&array, low: low, high: pivot)
    hoareQuicksort(&array, low: pivot + 1, high: high)
}
```

测试后，结果正确：

```swift
var list = [13, 2, 5, 8, 10, 32, 10]
hoareQuicksort(&list, low: 0, high: list.count - 1)
print(list)

// 结果：
[2, 5, 8, 10, 10, 13, 32]
```

### 最坏的情况分析

我们刚刚已经知道：1）lomuto 划分，以最后一个元素为支点；2）hoare 划分，以第一个元素为支点。

但是如果给定这个数组：`[5, 4, 3, 2, 1]`，如果使用 lomuto 划分，支点是最后一个元素 `1`，第一次划分后，将得到一下结果：

```
小于支点：[]
等于支点：[1]
大于支点: [5, 4, 3, 2]
```

对于这样的数组，排序的效率将会非常低，时间复杂度为 `O(n^2)`。所以选用第一个和最后一个元素都不是非常理想的支点。最理想的支点是可以把数组均匀的分成两个部分。

我们可以先通过下面这种方法来大概地找到数组的中间值：

```swift
func medianPivot<T: Comparable>(_ array: inout [T],
                                low: Int,
                                high: Int) -> Int {
    let center = (low + high) / 2
    if array[low] > array[center] {
        array.swapAt(low, center)
    }
    if array[low] > array[high] {
        array.swapAt(low, high)
    }
    if array[center] > array[high] {
        array.swapAt(center, high)
    }
    return center
}
```

执行完第一个 `if` 之后，将会满足`array[center] >= array[low]`；执行完第二个 `if` 之后，将会满足`array[high] >= array[low]`；执行完第三个 `if` 之后，将会满足`array[high] >= array[center]`。所以综合起来，执行完三个 `if` 之后，会得到 `array[high] >= array[center] >= array[low]`，我们就能大概把数组的中间值放在了数组中间。

优化 lomuto 划分，结果如下：

```swift
func medianQuicksort<T: Comparable>(_ array: inout [T],
                                    low: Int,
                                    high: Int) {
    guard low < high else {
        return
    }
    // 得到中间值所在的位置
    let center = medianPivot(&array, low: low, high: high)
    // 互换中间值和最后一个元素，lomuto 划分就能以中间值作为支点
    array.swapAt(center, high)
    let pivot = lomutoPartition(&array, low: low, high: high)
    lomutoQuicksort(&array, low: low, high: pivot - 1)
    lomutoQuicksort(&array, low: pivot + 1, high: high)
}
```

测试后，结果正确：

```swift
var list = [5, 4, 3, 2, 1]
hoareQuicksort(&list, low: 0, high: list.count - 1)
print(list)

// 结果：
[1, 2, 3, 4, 5]
```

### 非递归实现

前面的实现方法都是用了递归，如果要求不使用递归，我们可以用 [Stack](https://wp.me/p9EfvY-9d) 来模拟递归，因为递归的调用就是形成了一个 `Stack`。如果你还不了解  `Stack`，可以点击链接查看 `Stack` 的相关知识。

实现代码如下：

```swift
func nonrecursiveQuicksort<T: Comparable>(_ array: inout [T],
                                          low: Int,
                                          high: Int) {
    guard low < high else {
        return
    }
    var stack = Stack<Int>()
    stack.push(low)
    stack.push(high)
    
    while !stack.isEmpty {
        guard let end = stack.pop(),
            let start = stack.pop() else {
                continue
        }
        
        let pivot = lomutoPartition(&array, low: start, high: end)
        
        if (pivot - 1) > start {
            stack.push(start)
            stack.push(pivot - 1)
        }
        
        if (pivot + 1) < end {
            stack.push(pivot + 1)
            stack.push(end)
        }
    }
}
```

测试后，结果正确：

```swift
var list = [13, 2, 5, 8, 10, 32, 10]
nonrecursiveQuicksort(&list, low: 0, high: list.count - 1)
print(list)

// 结果：
[2, 5, 8, 10, 10, 13, 32]
```

### [完整代码 >>](https://github.com/Lebron1992/swift-algorithm-demo/blob/master/swift-algorithm/Sort/Quicksort.swift)

### 参考资料

> [Data Structures and Algorithms in Swift](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift) --- [raywenderlich.com](https://www.raywenderlich.com/)，如果想看原版书籍，请点击链接购买。
