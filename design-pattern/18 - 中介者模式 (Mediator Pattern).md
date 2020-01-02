> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

中介者模式属于行为模式，封装了对象之间的交互，使得对象之间不需要直接沟通，减少对象之间的耦合性。

它主要涉及以下四种类型：

- **Colleague**：需要互相交互的对象，实现了 Colleague 协议
- **Colleague Protocol**：定义了 Colleague 需要实现的方法和属性
- **Mediator**：控制 Colleague 之间的交互，实现了 Mediator Protocol
- **Mediator Protocol**：定义了 Mediator 需要实现的方法和属性

每个 Colleague 通过Mediator Protocol引用着同一个 Mediator，Colleague 之间的交互都是通过 Medaitor 间接进行交互。

![Mediator Pattern](http://upload-images.jianshu.io/upload_images/2057254-3c2021125addd8ab.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 什么时候使用

当我们想把多个对象之间复杂的交互关系，简化成通过一个对象进行处理时，使用这个模式。

### 简单demo

首先我们要搭建好中介者模式的框架：Mediator 协议、Colleague 协议、Mediator基类。

#### Mediator协议 & Colleague协议

```swift
protocol Colleague: class {
    func colleague(_ colleague: Colleague?,
                   didSendMessage message: String)
}

protocol MediatorProtocol: class {
    func addColleague(_ colleague: Colleague)
    func didSendMessage(_ message: String, by colleague: Colleague)
}
```

对于 Colleague 而言，他需要通过 Mediator 知道其他 Colleague 给他发送了什么消息，所以在 Colleague 协议中，定义了一个`colleague(_:didSendMessage:)`，具体的 Colleague 类通过实现这个方法，就知道其他 Colleague 说了什么。

`MediatorProtocol`规定 Mediator 需要：1）添加 Colleague 的方法；2）通知其他Colleagues 当前的 Coleague 说了什么。

#### Mediator基类

```swift
class Mediator<ColleagueType> {
    private(set) var colleagues: [ColleagueType] = []
    
    // MARK: - Manage Colleague
    
    func addColleague(_ colleague: ColleagueType) {
        colleagues.append(colleague)
    }
    
    func removeColleague(_ colleague: ColleagueType) {
        guard let index = colleagues
            .index(where: { ($0 as AnyObject) === (colleague as AnyObject)}) else {
                return
        }
        colleagues.remove(at: index)
    }
    
    func invokeColleagues(closure: (ColleagueType) -> Void) {
        colleagues.forEach(closure)
    }
    
    func invokeColleagues(by colleague: ColleagueType,
                          closure: (ColleagueType) -> Void) {
        colleagues.forEach {
            guard ($0 as AnyObject) !== (colleague as AnyObject) else { return }
            closure($0)
        }
    }
}
```

首先 Mediator 需要知道 Colleague 的类型，所以这里使用了泛型，把Colleague 的类型定义为`ColleagueType`：1）用一个数组来保存所有的 Colleagues；2）提供了添加和移除 Colleague 的方法；3）`invokeColleagues(closure:)`通过传入的 closure 参数，让每一个 Colleague 去执行 closure 里面的任务；4）`invokeColleagues(by:closure:)`某一个 Colleague 通过 closure 告诉其他 Colleagues 执行closure 里面的任务。

#### 具体的Colleague和Mediator

这里我们以一个大厦的安保人员为例。

##### SecurityStaff

```swift
final class SecurityStaff {
    var name: String
    weak var mediator: MediatorProtocol?
    
    init(name: String, mediator: MediatorProtocol) {
        self.name = name
        self.mediator = mediator
        mediator.addColleague(self)
    }
    
    func sendMessage(_ message: String) {
        print("\(name) 发送：\(message)")
        mediator?.didSendMessage(message, by: self)
    }
}

extension SecurityStaff: Colleague {
    func colleague(_ colleague: Colleague?,
                   didSendMessage message: String) {
        print("\(name) 收到: \(message)")
    }
}
```

Colleague 需要持有 Mediator，所以我们必须定义了一个`mediator`属性，并且使用`weak`避免循环引用。在初始化函数中，调用了 mediator 的`addColleague`方法，把 Colleague 添加到 Mediator 中；在`sendMessage`方法的实现里面，当前的安保人员通过调用 Mediator 的`didSendMessage(_:by:)`告诉 Mediator 发生了什么；最后还实现了`Colleague`协议，用来接受其他安保人员发出来的信息。

#### SecurityStaffMediator

```swift
final class SecurityStaffMediator: Mediator<Colleague> { }

extension SecurityStaffMediator: MediatorProtocol {
    func didSendMessage(_ message: String, by colleague: Colleague) {
        invokeColleagues(by: colleague) {
            $0.colleague(colleague, didSendMessage: message)
        }
    }
}
```

`SecurityStaffMediator`继承自`Mediator<Colleague>`，然后实现`MediatorProtocol`，在`didSendMessage(_:by:)`方法中，通知其他 Colleagues 发生了什么。

#### 使用

```swift
let mediator = SecurityStaffMediator()
let zhangSan = SecurityStaff(name: "张三", mediator: mediator)
let liSi = SecurityStaff(name: "李四", mediator: mediator)
let wangWu = SecurityStaff(name: "王五", mediator: mediator)


zhangSan.sendMessage("大厅有人抢劫")
print("")

liSi.sendMessage("我马上过去看看")
print("")

wangWu.sendMessage("我马上过去看看")

// 结果
张三 发送：大厅有人抢劫
李四 收到: 大厅有人抢劫
王五 收到: 大厅有人抢劫

李四 发送：我马上过去看看
张三 收到: 我马上过去看看
王五 收到: 我马上过去看看

王五 发送：我马上过去看看
张三 收到: 我马上过去看看
李四 收到: 我马上过去看看
```

首先创建`SecurityStaffMediator`对象，然后分别创建三个安保人员。张三说：大厅有人抢劫，李四和王五收到张三的信息后，都回复：我马上过去看看。

从打印的结果看，张三发送信息后，李四和王五都能收到，然后李四和王五的分别回复，另外两个人也能收到对应的信息。

### 总结

这个模式很适合用于断开 Colleagues 之间的联系，不让他们直接交互。如果一个 Mediator 变得非常庞大而且复杂，需要考虑把它分成多个小的中介者模式，或者看能不能与其他模式结合使用。
