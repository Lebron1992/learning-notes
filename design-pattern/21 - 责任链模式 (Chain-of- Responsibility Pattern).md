> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

责任链模式属于行为模式，它可以允许事件被一个或多个 handler 处理。涉及三个类型：

![](http://upload-images.jianshu.io/upload_images/2057254-d5bce6b6dbfeda36.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **Client**：接受事件并把事件传给 HandlerProtocol 实例
- **HandlerProtocol**：定义了 Concrete Handler 需要实现的属性和方法。HandlerProtocol 不一定是协议，也可以是一个基类。
- **Concrete Handler**：第一个 Concrete Handler 直接被 Client 持有，当接收到事件的时候，首先使用第一个 Concrete Handler 处理，如果无法处理，则把这个事件传给下一个 handler：`next`。

在整个模式中，Client 可以把所有Concrete Handlers 看做是一个实例，在底层实现中，每一个Concrete Handler 决定是否能处理事件，如果不能，则传给下一个 handler。如果没有一个 handler 可以处理事件，那么最后一个 handler 可以根据需要返回 `nil`、抛出错误或者什么都不做。

### 什么时候使用

当我们有一组用来处理类似事件的对象，但这些对象因为它所处理的事件不同而有差异时，使用责任链模式。

### 简单demo

自动售货机在大城市到处可见。在移动支付流行之前，我们所见的自动售货机基本只能接受纸币或者硬币；如今的自动售货机基本都是扫码支付的了。

这里我们以只能接受5元、10元、20元纸币的自动售货机为例：

- 1）自动售货机`VendingMachine`属于 Client 角色，接受纸币；
- 2）Handler Protocol 要求有一个`next`属性和`handleRMBValidation(_:)`方法；
- 3）Concrete Handler 就是各类纸币的验证器 `RMBHandler`。

另外，为了简便，我们只通过验证纸币的宽高来判断纸币的面值（实际情况肯定复杂很多）。通过查询，5元、10元、20元纸币的长宽如下：

| 面值         | 大小（单位：毫米）   |
| ------------ |:--------------------:|
| 5元          | 长135 宽63           |
| 10元         | 长140 宽70           |
| 20元         | 长145 宽70           |

#### RMB 基类

```swift
class RMB {
    class var standardWidth: CGFloat {
        return 0
    }
    class var standardHeight: CGFloat {
        return 0
    }
    
    var rmbValue: Double {
        return 0
    }
    
    final let width: CGFloat
    final let height: CGFloat
    
    required init(width: CGFloat, height: CGFloat) {
        self.width = width
        self.height = height
    }
    
    convenience init() {
        let width = type(of: self).standardWidth
        let height = type(of: self).standardHeight
        self.init(width: width, height: height)
    }
}

extension RMB: CustomStringConvertible {
    var description: String {
        return "长：\(width)，宽：\(height)，面值：\(rmbValue)元"
    }
}
```

创建一个 RMB 基类：

- 1）`standardWidth`和`standardHeight`只读属性表示纸币的标准宽高，单位是毫米，让子类实现；
- 2）`rmbValue`表示纸币面值，让子类实现；
- 3）`width`和 `height`两个存储属性，表示纸币的实际长宽，单位是毫米，因为纸币在使用过程等中，肯定会有一些磨损，实际的长宽与标准长宽有差别；
- 4）`required`标记的初始化函数，我们需要通过这个函数来初始化；
- 5）`convenience`初始化函数，用于创建标准的 RMB；
- 6）最后实现了`CustomStringConvertible`，打印的时候更好看。

#### 5元、10元和20元

```swift
final class FiveYuan: RMB {
    
    override class var standardWidth: CGFloat {
        return 135
    }
    override class var standardHeight: CGFloat {
        return 63
    }
    override var rmbValue: Double {
        return 5
    }
}

final class TenYuan: RMB {
    
    override class var standardWidth: CGFloat {
        return 140
    }
    override class var standardHeight: CGFloat {
        return 70
    }
    override var rmbValue: Double {
        return 10
    }
}

final class TwentyYuan: RMB {
    
    override class var standardWidth: CGFloat {
        return 145
    }
    override class var standardHeight: CGFloat {
        return 70
    }
    override var rmbValue: Double {
        return 20
    }
}
```

分别创建了`FiveYuan`、`TenYuan`和`TwentyYuan`。

#### RMBHandler

```swift
protocol RMBHandlerProtocol {
    var next: RMBHandlerProtocol? { get }
    func handleRMBValidation(_ unknownRMB: RMB) -> RMB?
}
```

首先创建了`RMBHandlerProtocol`，规定`next`属性和`handleRMBValidation(_:)`方法。

```swift
final class RMBHandler: RMBHandlerProtocol {
    var next: RMBHandlerProtocol?
    let rmbType: RMB.Type
    let widthRange: ClosedRange<CGFloat>
    let heightRange: ClosedRange<CGFloat>
    
    init(rmbType: RMB.Type,
         widthTolerance: CGFloat = 1,
         heightTolerance: CGFloat = 1) {
        
        self.rmbType = rmbType
        
        let standardWidth = rmbType.standardWidth
        self.widthRange = (standardWidth - widthTolerance) ...
            (standardWidth + widthTolerance)
        
        let standardHeight = rmbType.standardHeight
        self.heightRange = (standardHeight - heightTolerance) ...
            (standardHeight + heightTolerance)
    }
}

extension RMBHandler {
    func handleRMBValidation(_ unknownRMB: RMB) -> RMB? {
        guard let rmb = createRMB(from: unknownRMB) else {
            return next?.handleRMBValidation(unknownRMB)
        }
        return rmb
    }
    
    private func createRMB(from unknownRMB: RMB) -> RMB? {
        print("尝试创建RMB：\(rmbType)")
        guard widthRange.contains(unknownRMB.width) else {
            print("长度不符合要求")
            return nil
        }
        guard heightRange.contains(unknownRMB.height) else {
            print("宽度不符合要求")
            return nil
        }
        let rmb = rmbType.init(width: unknownRMB.width,
                               height: unknownRMB.height)
        print("已创建纸币，\(rmb)")
        return rmb
    }
}
```

创建具体的 Handler，`RMBHandler`：
- 1）`next`，下一个 `RMBHandler`；
- 2）`rmbType`，人民币类型，这样我们就不必为每一类纸币都创建一个Handler，例如`FiveYuanHandler`等；
- 3）`widthRange`和`heightRange`是有效的长宽范围；
- 4）在初始化函数中，我们默认地允许长宽的误差为1毫米；
- 5）实现`handleRMBValidation(_:)`，判断传入的`unknownRMB`的长宽是否符合要求，若符合，创建具体的 RMB 实例，否则让下一个 handler 继续处理。

#### VendingMachine

```swift
final class VendingMachine {
    
    private let rmbHandler: RMBHandler
    private(set) var rmbs: [RMB] = []
    
    init(rmbHandler: RMBHandler) {
        self.rmbHandler = rmbHandler
    }
    
    func insertRMB(_ unknownRMB: RMB) {
        guard let rmb = rmbHandler
            .handleRMBValidation(unknownRMB) else {
                print("无效的纸币")
                return
        }
        print("纸币已接收")
        rmbs.append(rmb)
        
        let totalValue = rmbs.reduce(0, { $0 + $1.rmbValue })
        print("已投币总金额：\(totalValue)元")
    }
}
```

创建了 Client 角色，`VendingMachine`：

- 1）`rmbHandler`：handler 链的第一个handler，自动售货机不需要知道 handler 链的所有 handlers；
- 2）`rmbs`：所有已投的有效人民币；
- 3）简单的初始化函数；
- 4）`insertRMB(_:)`插入未知的人民币，首先验证是否是有效的人民币，如果有效，添加到`rmbs`中，最后打印已投币的总额。

#### 使用

##### 准备

```swift
let fiveYuanHandler = RMBHandler(rmbType: FiveYuan.self)
let tenYuanHandler = RMBHandler(rmbType: TenYuan.self)
let twentyYuanHandler = RMBHandler(rmbType: TwentyYuan.self)

fiveYuanHandler.next = tenYuanHandler
tenYuanHandler.next = twentyYuanHandler

let vendingMachine = VendingMachine(rmbHandler: fiveYuanHandler)
```

创建三种纸币 Handler，并且给`next`属性赋值；最后创建自动售货机。

##### 插入标准的5元纸币

```swift
let fiveYuan = FiveYuan()
vendingMachine.insertRMB(fiveYuan)

// 结果
尝试创建RMB：FiveYuan
已创建纸币，长：135.0，宽：63.0，面值：5.0元
纸币已接收
已投币总金额：5.0元
```

投入标准的5元纸币，成功插入。

##### 插入标准的20元纸币

```swift
let twentyYuan = TwentyYuan()
vendingMachine.insertRMB(twentyYuan)

// 结果
尝试创建RMB：FiveYuan
长度不符合要求
尝试创建RMB：TenYuan
长度不符合要求
尝试创建RMB：TwentyYuan
已创建纸币，长：145.0，宽：70.0，面值：20.0元
纸币已接收
已投币总金额：20.0元
```

投入标准的20元纸币，刚开始5元和10元的 handler 不能处理20元的纸币，最后20元的 handler 可以处理，成功插入。

##### 投入无效的纸币

```swift
let rmb = RMB(width: TwentyYuan.standardWidth,
              height: FiveYuan.standardHeight)
vendingMachine.insertRMB(rmb)

// 结果
尝试创建RMB：FiveYuan
长度不符合要求
尝试创建RMB：TenYuan
长度不符合要求
尝试创建RMB：TwentyYuan
宽度不符合要求
无效的纸币: 长：145.0，宽：63.0，面值：0.0元
```

纸币无效。

### 总结

责任链模式，对于可以很快决定是否能处理事件的handler非常适合，所以要尽量避免做决定的过程非常耗时的 handler。另外，还要考虑如果所有的 handlers 都不能处理事件时，我们该怎么做？
