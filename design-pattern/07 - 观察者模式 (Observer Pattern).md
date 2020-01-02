> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

观察者模式可以让一个对象观察另一个对象的变化。在iOS中，我们一般使用KVO来实现。

观察者模式主要有两个对象组成：

- **subject**：被观察的对象。
- **observer**：观察subject的对象。

### 什么时候使用

当我们想关注一个对象的变化时，可以使用观察者模式。

### 简单Demo 

创建一个`User`类，继承于`NSObject`：

```swift
@objcMembers class User: NSObject {
    dynamic var name: String
    public init(name: String) {
        self.name = name
    }
}
```

从Swift 4开始，继承自`NSObject`的类不会自动把他们的属性暴露给Objective-C runtime。因为`NSObject`使用Objective-C runtime来执行KVO，所以我们手动添加`@objcMembers`，它的作用相当于在每一个属性前面加上`@objc`。

`dynamic`意思是用Objective-C runtime动态调度系统来调用属性的getter和setter，意味着静态或者虚拟调用将永远不会被使用。我们要加上`dynamic`，KVO才能正常进行。

下面我们创建一个`User`实例，并且观察`name`的变化：

```swift
let user = User(name: "Lebron")

var nameObservation: NSKeyValueObservation? =
    user.observe(\.name, options: [.initial, .new]) { (user, change) in
        print("名字是：\(user.name)")
}

// 名字是：Lebron
```

这里我们使用Swift 4的KVO写法，`\.name`是`\KVOUser.name`的简写，是一个`KeyPath`；`options`参数是用来指定想要观察哪些类型的数据，这里传入`[.initial, .new]`，意味着可以观察到初始的实例时`name`的值和`name`变化时的值；最后一个参数是闭包，闭包有`user`和`change`两个对象，`user`是所有变化完成后的对象，如果`.new`事件被触发，`change`会包含一个旧值`oldValue`。

在`options`参数中，我们传入了`.initial`，所以我们能看到这个打印`名字是：Lebron`。

如果尝试把`name`改为`Love`：

```
user.name = "Love"

// 名字是：Love
```

控制台出现：`名字是：Lebron`。我们成功地观察了`name`的变化。

在Swift 4的KVO中，我们不需要手动的移除观察者或者闭包，因为观察者是被弱引用着的，当观察者变为`nil`时，闭包会自动移除。而在以前的Swift版本或者Objective-C中，我们必须调用`removeObserver`来移除，否则当我们访问被回收了的观察者时，应用会crash。

虽然Swift 4的KVO不需要我们手动移除观察则，但是被观察的对象必须继承自`NSObject`和使用Objective-C runtime。
