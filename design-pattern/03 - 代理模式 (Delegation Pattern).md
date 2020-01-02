> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

代理模式可以让一个对象用另外一个对象来提供一些数据或者执行某些任务。这种模式有三部分：

- **需要代理的对象**：也就是有一个`delegate`属性的对象，通常使用`weak`修饰，以避免引用循环（需要代理的对象引用着代理，而代理又引用着需要代理的对象）。
- **代理协议**：定义了代理需要实现的方法。
- **代理**：实现代理协议的对象。

有了代理协议，那么代理协议的实现变得非常灵活，任何对象都可以实现这个协议，成为代理。

### 什么时候使用

当我们要创建一些通用的、可重复利用的组件时，可以考虑使用代理模式。代理模式在iOS的框架中普遍使用，特别是`UIKit`，例如`xxxDataSource`和`xxxDelegate`都是代理模式。

为什么在iOS的框架中，要把`xxxDataSource`和`xxxDelegate`分开呢？

其实是为了更好的管理，`xxxDataSource`专门用来提供数据，`xxxDelegate`用来接收数据或者事件。我们在自定义控件的时候也可以用这种思想。

### 简单Demo

假设我们有一个侧滑菜单，用户点击菜单中的每一项，然后显示不同的View Controller，我们就可以通过代理来实现。

- 我们用一个枚举`MenuItem`列举出菜单有哪些选项
- 实现UITableViewDataSource
- 当用户点击了某一项之后，我们代理协议来告诉代理用户点击了哪一项，所以定义`MenuViewControllerDelegate`
- 定义`delegate`属性，然后在`didSelectRowAt`中调用代理方法
- 当其他对象成为`MenuViewController`的代理之后，它就能收到用户的点击事件

```swift
protocol MenuViewControllerDelegate: class {
    func menuViewController(_ menuViewController: MenuViewController,
                            didSelectItem item: MenuItem)
}

enum MenuItem: String {
    case home = "首页"
    case me = "我的"
    case settings = "设置"
}

class MenuViewController: UIViewController {

    weak var delegate: MenuViewControllerDelegate?

    @IBOutlet var tableView: UITableView! {
        didSet {
            tableView.dataSource = self
            tableView.delegate = self
        }
    }

    private let items: [MenuItem] = [.home, .me, .settings]
}

// MARK: - UITableViewDataSource & UITableViewDelegate
extension MenuViewController: UITableViewDataSource, UITableViewDelegate {
    func tableView(_ tableView: UITableView,
                   numberOfRowsInSection section: Int) -> Int {
        return items.count
    }
    
    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
            cell.textLabel?.text = items[indexPath.row].rawValue
            return cell
    }

    func tableView(_ tableView: UITableView,
                   didSelectRowAt indexPath: IndexPath) {
        let item = items[indexPath.row]
        delegate?.menuViewController(self, didSelectItem: item)
    }
}
```

### 总结

代理模式在开发中经常使用，但是不要过度使用。如果一个对象定义很多delegate，说明我们这个类做了太多的事情，要考虑把它拆分了。
