> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

MVC (model-view-controller) 把所有类分成三种类型：模型、视图和控制器。

MVC在iOS开发中是非常常见的，因为这是Apple重度使用的一种设计模式。

**Model**：保存应用数据，通常是Struct和Class类型。在Swift中，我们优先选择Struct，因为Struct是值类型，可以避免不必要的循环引用问题。
**View**：在屏幕上显示应用界面，通常继承于`UIView`。
**Controller**：控制和协调Model和View，通常继承于`UIViewContorller`。

Controller可以强引用Model和View，所以可以直接访问Model和View。但是反过来Model和View不能强引用Controller，否则会造成引用循环。

Model可以通过属性观察者来告诉Controller数据发生变化，例如`willSet`和`didSet`；而View可以通过手势或者`valueChanged`事件等告诉Controller。

### 简单Demo

需求是展示一个用户的名字和年龄：

1. **Model**：定义了一个Person，有`name`和`age`属性
2. **View**：`nameLabel`和`ageLabel`
3. **Controller**：`PersonInfoViewController`，强引用了两个views，`nameLabel`和`ageLabel`；还强引用了`person`。

在`viewDidLoad`我们从服务器获取数据，获取到数据之后，我们修改`person`模型的值；模型被修改后告诉控制器，控制器更新`nameLabel`和`ageLabel`。这就是一个简单的MVC。

```swift
import UIKit

struct Person {
    var name: String
    var age: Int
}

final class PersonInfoViewController: UIViewController {

    @IBOutlet var nameLabel: UILabel!
    @IBOutlet var ageLabel: UILabel!

    var person: Person? {
        didSet {
            nameLabel.text = person?.name
            ageLabel.text = person?.age
        }
    }
 
    override func viewDidLoad() {
        super.viewDidLoad()
        fetchPersonInfo()
    }

    private func fetchPersonInfo() {
        // 假设name和age是从服务器获取的
        let name = "Lebron"
        let age = 25
        person = Person(name: name, age: age)
    }
}
```

### 总结

在我们刚刚开始学iOS开发时，一般都是从MVC模式开始的。但是MVC有个问题，如果是比较大型的项目，Controller的代码将会变得非常复杂，不方便维护。所以在实际开发中，大型的项目，我们一般考虑使用其他设计模式，例如MVVM等。
