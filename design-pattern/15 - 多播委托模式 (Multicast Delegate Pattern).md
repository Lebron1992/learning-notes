> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

多播委托模式属于行为模式，属于代理模式的变种。我们可以通过这种模式来创建一对多的代理模式。

这个模式涉及四个部分：

- **需要代理的对象**：这个对象可以有一个或者多个delegates
- **代理协议**：规定代理需要实现的方法
- **代理**：实现代理协议的对象
- **多播委托**：持有多个代理，并对代理进行管理

多播委托模式和代理模式的主要区别在于：多播委托模式多了一个多播委托对象。

### 什么时候使用

使用这个模式来创建一对多的代理模式。

### 简单demo

在这个demo中，我们实现这个例子：特朗普大厦发生火灾，大厦的安保中心及时通知消防中心和医院的急救中心。

从这个例子可以分析得出：1）**需要代理的对象**是大厦的安保中心`SecurityCenter`；2）**代理**是消防中心`FireStation`和医院的急救中心`HospitalEmergencyCenter`；3）Swift默认没有提供**多播委托**给我们，需要自己创建`MulticastDelegate`；4）**代理协议**：`FireEmergencyResponding`。

#### MulticastDelegate

```swift
final class MulticastDelegate<ProtocolType> {

    // MARK: - Helper Types

    private final class DelegateWrapper {
        weak var delegate: AnyObject?
        init(delegate: AnyObject) { self.delegate = delegate }
    }

    // MARK: - Properties

    private var delegateWrappers: [DelegateWrapper]
    private var delegates: [ProtocolType] {
        delegateWrappers = delegateWrappers.filter { $0.delegate != nil }
        return delegateWrappers.map { $0.delegate } as! [ProtocolType]
    }

    // MARK: - Initializers

    init(delegates: [ProtocolType] = []) {
        delegateWrappers = delegates
            .map { DelegateWrapper(delegate: $0 as AnyObject)}
    }

    // MARK: - Delegate Management

    func addDelegate(_ delegate: ProtocolType) {
        let wrapper = DelegateWrapper(delegate: delegate as AnyObject)
        delegateWrappers.append(wrapper)
    }

    func removeDelegate(_ delegate: ProtocolType) {
        guard let index = delegateWrappers
            .index(where: { $0.delegate === (delegate as AnyObject)}) else {
                return
        }
        delegateWrappers.remove(at: index)
    }

    func notifyDelegates(_ closure: (ProtocolType) -> Void) {
        delegates.forEach { closure($0) }
    }
}
```

为了保证`MulticastDelegate`的通用性，把它定义为泛型。

另外，为了不让`MulticastDelegate`强引用代理，定义了一个内部类`DelegateWrapper`来把代理包装起来，内部类里面弱引用代理。在内部类里面，`delegate`被定义为`AnyObject?`，而不是`ProtocolType?`，因为`weak`只能用于Class类型。

还添加了管理代理的方法。

#### 代理协议

```swift
protocol FireEmergencyResponding: class {
    func notifyFire(at location: String)
}
```

火灾紧急响应，只有一个方法，通知在某个地方发生了火灾。

#### 代理

```swift
final class FireStation: FireEmergencyResponding {
    func notifyFire(at location: String) {
        print("已经通知消防员在\(location)发生了火灾")
    }
}

final class HospitalEmergencyCenter: FireEmergencyResponding {
    func notifyFire(at location: String) {
        print("已经通知医护人员在\(location)发生了火灾")
    }
}
```

消防中心和医院的急救中心。

#### 需要代理的对象

```swift
final class SecurityCenter {
    static let shared = SecurityCenter()
    let multicastDelegate = MulticastDelegate<FireEmergencyResponding>()
}
```

大厦的安保中心，因为安保中心通常只有一个，所以这里使用了单例。

#### 使用

```swift
let securityCenter = SecurityCenter.shared
var fireStation: FireStation! = FireStation()
let hospital = HospitalEmergencyCenter()

securityCenter.multicastDelegate.addDelegate(fireStation)
securityCenter.multicastDelegate.addDelegate(hospital)

securityCenter.multicastDelegate.notifyDelegates {
    $0.notifyFire(at: "特朗普大厦")
}

print("======分割线======")

fireStation = nil

securityCenter.multicastDelegate.notifyDelegates {
    $0.notifyFire(at: "特朗普大厦")
}

// 结果
已经通知消防员在特朗普大厦发生了火灾
已经通知医护人员在特朗普大厦发生了火灾
======分割线======
已经通知医护人员在特朗普大厦发生了火灾
```

首先创建了两个代理：`fireStation`和`hospital`。把`fireStation`定义为`FireStation!`，这样后面我们可以把`fireStation`设置为`nil`。然后把两个代理添加到`multicastDelegate`，接着通知特朗普大厦发生火灾，得到了我们预期的打印结果。

把`fireStation`设置为`nil`之后，只有医院接收到了通知。

### 总结

多播委托模式非常适合用于把信息告诉代理的场景。如果是需要多个代理提供数据，这种模式就不适合了。
