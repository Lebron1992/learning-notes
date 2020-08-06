# 02 - TDD 循环

> 本文是阅读 [iOS Test-Driven Development by Tutorials](https://store.raywenderlich.com/products/ios-test-driven-development) 后的学习笔记，有需要的请点击链接购买。

在上一篇文章已经提到，TDD 的流程有四个步骤：
1. 写一个失败的测试
2. 使测试通过
3. 重构
4. 重复

下面通过例子来演示每个步骤。

## 写一个失败的测试

在 playground 中编写以下测试用例：

```swift
class CashRegisterTests: XCTestCase {
    func testInit_createsCashRegister() {
        XCTAssertNotNil(CashRegister())
    }
}

// 调用这个可以在 playground 运行测试
CashRegisterTests.defaultTestSuite.run()
```

在 iOS 中，测试用例的方法都是以 `test` 开头。因为没有定义 `CashRegister`，所以编译器报错。而对于 TDD 循环来说，编译报错可以看做是测试失败，所以到此我们完成第一步：写一个失败的测试。

## 使测试通过

如果编写最少的代码使得上面的测试通过？当然是定义 `CashRegister`。

```swift
class CashRegister {

}
```

执行 playground 后，得到以下输出：

```
Test Suite 'CashRegisterTests' started at 2020-08-06 02:17:19.397
Test Case '-[__lldb_expr_1.CashRegisterTests testInit_createsCashRegister]' started.
Test Case '-[__lldb_expr_1.CashRegisterTests testInit_createsCashRegister]' passed (0.210 seconds).
Test Suite 'CashRegisterTests' passed at 2020-08-06 02:17:19.608.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.210 (0.212) seconds
```

可以看到测试通过，所以到此我们完成第二步：使测试通过。

## 重构

在这一步中，我们将清理应用程序代码和测试代码。通过这样做，可以不断地维护和改进代码。以下是一些可以重构的内容：

- **重复的逻辑**：反问自己，能抽出任何属性、方法或类来消除重复吗？

- **注释**：注释应该解释为什么要做某件事，而不是怎么做的。尽量消除解释代码工作原理的注释。应该通过将复杂方法分解为几个命名良好的方法，重命名属性和方法以使其更清晰，或者有时只是简单地将代码结构更好地表达出来。

- **错误代码**：有时某个特定的代码块似乎是错误的。相信你的直觉，试着消除这些错误代码。例如，你可能有做过多假设的逻辑，使用硬编码字符串或有其他问题。

现在， `CashRegister` 和 `CashRegisterTests` 没有太多的逻辑，也没有什么可以重构的。所以到此我们完成第三步：重构。


## 重复

前面已经完成了第一个 TDD 周期，现在我们重复这个周期。在这内容，将为 `CashRegister` 添加以下方法：
- 接受可用资金参数的初始化器
- 用于添加 item 的方法

### 接受可用资金参数的初始化函数

首先写测试用例：

```swift
func testInitAvailableFunds_setsAvailableFunds() {
    // given
    let availableFunds: Decimal = 100
    
    // when
    let sut = CashRegister(availableFunds: availableFunds)
    
    // then
    XCTAssertEqual(sut.availableFunds, availableFunds)
}
```

编译错误，因为 `CashRegister` 还没有那个初始化函数。下面编写这个函数：

```swift
class CashRegister {
    var availableFunds: Decimal
    
    init(availableFunds: Decimal = 0) {
        self.availableFunds = availableFunds
    }
}
```

编译错误消失，执行 playground 后，得到以下输出：

```
Test Suite 'CashRegisterTests' started at 2020-08-06 09:42:40.794
Test Case '-[__lldb_expr_3.CashRegisterTests testInit_createsCashRegister]' started.
Test Case '-[__lldb_expr_3.CashRegisterTests testInit_createsCashRegister]' passed (0.174 seconds).
Test Case '-[__lldb_expr_3.CashRegisterTests testInitAvailableFunds_setsAvailableFunds]' started.
Test Case '-[__lldb_expr_3.CashRegisterTests testInitAvailableFunds_setsAvailableFunds]' passed (0.008 seconds).
Test Suite 'CashRegisterTests' passed at 2020-08-06 09:42:40.978.
	 Executed 2 tests, with 0 failures (0 unexpected) in 0.182 (0.184) seconds
```

可以看到测试通过。到此我们完成了两个步骤：1. 写一个失败的测试；2. 使测试通过。

第三步是重构，这一步中我们将清理应用程序代码和测试代码。

对于测试代码，可以发现 `testInit_createsCashRegister` 是没必要的，所以可以把它删掉。

对于应用代码，初始化器 `init(availableFunds: Decimal = 0)` 的参数有一个默认值，我们得思考这个默认值是否有必要？这将会产生以下两种情况：

- 如果保留默认值，那么就得考虑为这个默认值添加测试
- 如果不保留默认值，那么就把它删掉

在这里，我们把它删掉，变成：

```swift
init(availableFunds: Decimal) {
    self.availableFunds = availableFunds
}
```

重构完成后，继续执行 playground，发现还能测试通过。通过这个例子，可以感觉到遵循 TDD 能让我们在重构时更有信心。

### 用于添加 item 的方法

首先写测试用例：

```swift
func testAddItem_oneItem_addsCostToTransactionTotal() {
    // given
    let availableFunds: Decimal = 100
    let sut = CashRegister(availableFunds: availableFunds)
    let itemCost: Decimal = 42
    
    // when
    sut.addItem(itemCost)
    
    // then
    XCTAssertEqual(sut.transactionTotal, itemCost)
}
```

编译错误，因为 `CashRegister` 还没有 `addItem` 方法和 `transactionTotal` 属性。下面编写 `addItem` 方法和 `transactionTotal` 属性：

```swift
class CashRegister {
    var transactionTotal: Decimal = 0
    var availableFunds: Decimal
    
    init(availableFunds: Decimal = 0) {
        self.availableFunds = availableFunds
    }
    
    func addItem(_ cost: Decimal) {
        transactionTotal = cost
    }
}
```

在 `addItem` 方法的实现中，直接把 `cost` 赋值给 `transactionTotal` 很明显是不对的。但对于这个测试用例来说，这么做也能让编译错误消失。所以我们暂时先这么做。执行 playground 后，可以看到测试通过。

接下来进行重构。

对于测试代码，可以发现一下的代码在测试用例中重复了：

```swift
let availableFunds: Decimal = 100
let sut = CashRegister(availableFunds: availableFunds)
```

所以我们可以把这两个变量抽出来，作为 `CashRegisterTests` 的属性，并且在 `setUp()` 和 `tearDown()` 方法中初始化和重置他们的值。最终重构后的代码如下：

```swift
class CashRegisterTests: XCTestCase {
    
    var availableFunds: Decimal!
    var sut: CashRegister!
    
    override func setUp() {
        super.setUp()
        availableFunds = 100
        sut = CashRegister(availableFunds: availableFunds)
    }

    override func tearDown() {
        availableFunds = nil
        sut = nil
        super.tearDown()
    }
    
    func testInitAvailableFunds_setsAvailableFunds() {
        XCTAssertEqual(sut.availableFunds, availableFunds)
    }
    
    func testAddItem_oneItem_addsCostToTransactionTotal() {
        // given
        let itemCost: Decimal = 42
        
        // when
        sut.addItem(itemCost)
        
        // then
        XCTAssertEqual(sut.transactionTotal, itemCost)
    }
}
```

在 `setUp()` 中初始哈变量的值；在 `tearDown()` 方法中重置变量的值。另外，我们应该总是在 `tearDown()` 中把变量设置为 `nil`，因为 `XCTestCase` 类只在所有测试完成之后才会释放变量占用的内存，所以如果我们有很多测试用例并且不在 `tearDown()` 中把变量设置为 `nil`的话，那么测试的性能可能会受到影响。

#### 添加两个 items

`testAddItem_oneItem` 测试用例证明了 `addItem()` 在添加一个 item 时是正确的。如果添加两个或更多 items 时会怎样呢？我们来测试一下。

添加测试用例如下：

```swift
func testAddItem_twoItems_addsCostsToTransactionTotal() {
    // given
    let itemCost: Decimal = 42
    let itemCost2: Decimal = 20
    let expectedTotal = itemCost + itemCost2
    
    // when
    sut.addItem(itemCost)
    sut.addItem(itemCost2)
    
    // then
    XCTAssertEqual(sut.transactionTotal, expectedTotal)
}
```

执行后，发现刚刚添加的测试用例失败了。这证明 `addItem()` 方法的实现有问题，我们很快找到问题所在，把实现改为：

```swift
transactionTotal += cost
```

再次执行后，所有测试通过。

接下来是重构：仔细查看测试代码，发现 `itemCost` 可以抽取出来，重构后代码为：

```swift
class CashRegisterTests: XCTestCase {
    
    var availableFunds: Decimal!
    var sut: CashRegister!
    var itemCost: Decimal!
    
    override func setUp() {
        super.setUp()
        availableFunds = 100
        sut = CashRegister(availableFunds: availableFunds)
        itemCost = 42
    }

    override func tearDown() {
        availableFunds = nil
        sut = nil
        itemCost = nil
        super.tearDown()
    }
    
    func testInitAvailableFunds_setsAvailableFunds() {
        XCTAssertEqual(sut.availableFunds, availableFunds)
    }
    
    func testAddItem_oneItem_addsCostToTransactionTotal() {
        // when
        sut.addItem(itemCost)
        
        // then
        XCTAssertEqual(sut.transactionTotal, itemCost)
    }
    
    func testAddItem_twoItems_addsCostsToTransactionTotal() {
        // given
        let itemCost2: Decimal = 20
        let expectedTotal = itemCost + itemCost2
        
        // when
        sut.addItem(itemCost)
        sut.addItem(itemCost2)
        
        // then
        XCTAssertEqual(sut.transactionTotal, expectedTotal)
    }
}
```

## 总结

本文用简单的例子模拟了 TDD 循环的流程。