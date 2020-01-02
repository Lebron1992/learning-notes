> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

单例模式限制一个类只能有一个实例，每一个对这个类的引用，都指向同一个实例。

### 什么时候使用

当一个类有多个实例会出现问题的时候，我们使用单例模式。

### 简单Demo

iOS中我们经常会用到一些单例模式，例如：

```swift
UIApplication.shared

NotificationCenter.default

UserDefaults.standard

URLSession.shared

FileManager.default
```

通常以`shared`、`default`、`standard`命名的，都是用单例。

我们可以创建自己的单例：

```swift
class MySingleton {
    static let shared = MySingleton()
    private init() { }
}

let singleton = MySingleton.shared
//let singleton2 = MySingleton() // 报错
```

在上面这个例子中，我们把`MySingleton`的初始化函数定义成了`private`，保证外部不能再自己创建另外一个`MySingleton`实例。

### 另外一种“单例模式”

我们上面讲到的单例，是一种严格的单例模式，保证一个类只能有一个实例，外部不能再创建第二个实例。

实际上我们还可以有另外一种类似的单例模式，同样有一个`shared`实例，但是它允许再创建另外一个实例。例如iOS中的`FileManager`：

```swift
let defaultFileManager = FileManager.default
let customFileManager = FileManager()
```

在上面的那个`MySingleton`例子中，我们把初始化函数的`private`关键字去掉，就是我们的这部分讲到的单例模式：

```swift
class MySingleton {
    static let shared = MySingleton()
    init() { }
}

let singleton = MySingleton.shared
let singleton2 = MySingleton()
```

### 总结

单例模式非常容易滥用。

当遇到某些需求想要使用单例模式来实现的时候，要优先考虑是否有其他方式可以实现？

当确定要使用单例模式时，考虑下是否可以让外部自己创建另外一个实例？

单例模式最大的问题是测试。如果有多个状态存储在一个单例中，测试案例的顺序跟单例的状态有密切的联系，那么在测试的时候就会比较麻烦。
