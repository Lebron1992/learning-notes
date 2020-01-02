> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

原型模式属于创建型模式，允许我们对一个对象进行复制。它包含两个部分：1) **Copying**协议；2) 遵循**Copying**协议的Class类型。

复制又分为两种类型：浅复制和深复制：

- **浅复制**：创建了一个新的实例，但是不复制原实例的属性，而是所有属性都指向原实例。

- **深复制**：创建新的实例，并且复制原有实例的属性。

### 什么时候使用

当想要复制一个对象的时候，使用这个模式。

### 简单demo

在Objective-C的时候，有一个`NSCopying`协议，虽然在Swift还是可以使用，但使用起来不是非常友好，所以这里我们就自己定义一个`Copying`协议：

```swift
protocol Copying: class {
    init(_ prototype: Self)
}

extension Copying {
    func copy() -> Self {
        return type(of: self).init(self)
    }
}
```

首先我们定义了一个`Copying`协议，并规定了一个把当前类型的实例作为参数初始化函数；然后通过扩展实现了`copy()`方法。

接下来定义一个宠物类，并实现`Copying`协议：

```swift
class Pet: Copying {
    let name: String
    let weight: Double

    init(name: String, weight: Double) {
        self.name = name
        self.weight = weight
    }

    // MARK: - Copying

    required convenience init(_ pet: Pet) {
        self.init(name: pet.name, weight: pet.weight)
    }
}
```

试试我们刚刚定义的宠物类：

```swift
let pet1 = Pet(name: "Lili", weight: 10)
let pet2 = pet1.copy()
print("pet1====name: \(pet1.name)====weight: \(pet1.weight)")
print("pet2====name: \(pet2.name)====weight: \(pet2.weight)")

// 结果
pet1====name: Lili====weight: 10.0
pet2====name: Lili====weight: 10.0
```

结果完全一样，这样我们就完成了对一个实例的复制。
