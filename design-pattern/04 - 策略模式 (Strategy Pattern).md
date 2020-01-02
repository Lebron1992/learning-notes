> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

策略模式定义了多个在运行中可以切换的对象。这个模式由三部分组成：


![策略模式UML图](http://upload-images.jianshu.io/upload_images/2057254-6c7ca6de0273bec0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **使用策略的对象**：在iOS开发中，通常是一个view controller，但是在理论上来说，可以是任何类型的对象。
- **策略协议**：定义了每一种策略需要实现的方法。
- **所有策略**：所有遵循策略协议的策略。

### 什么时候使用

当我们有多个不同的可切换的行为时，可以使用策略模式。

策略模式有点类似于代理模式，这两种模式都依赖于某个协议；在代理模式中，遵循了协议的对象就可以成为代理；在策略模式中，遵循了协议的就可以成一个策略。

但不同的是，策略模式有多个策略组成的集合，可以在运行时根据需要切换。

### 简单Demo

假设我们要做一个可以查看餐饮商家在不同平台的评分情况，例如大众点评、口碑等，我们就可以使用策略模式。

- 首先定义一个策略协议`FoodMerchantRatingStrategy`，要求有平台的名称`ratingServiceName`和获取评分的方法
- 定义大众点评和口碑两个平台：`DianPingClient`和`KouBeiClient`，并实现`FoodMerchantRatingStrategy`协议
- 在`FoodMerchantRatingViewController`中，我们就可以根据传入的`foodMerchantRatingClient`做相应的显示

代码如下：

```swift
import UIKit

protocol FoodMerchantRatingStrategy {
    var ratingServiceName: String { get }
    func fetchRatingforMerchant(named name: String,
                            success: (_ rating: String, _ review: String) -> Void)
}

class DianPingClient: FoodMerchantRatingStrategy {
    let ratingServiceName = "大众点评"
    func fetchRatingforMerchant(named name: String,
                            success: (String, String) -> Void) {
        let rating = "⭐️⭐️⭐️⭐️"
        let review = "还不错！！！"
        success(rating, review)
    }
}

class KouBeiClient: FoodMerchantRatingStrategy {
    let ratingServiceName = "口碑"
    func fetchRatingforMerchant(named name: String,
                            success: (String, String) -> Void) {
        let rating = "⭐️⭐️⭐️⭐️⭐️"
        let review = "非常好！"
        success(rating, review)
    }
}

class FoodMerchantRatingViewController: UIViewController {

    private let foodMerchantRatingClient: FoodMerchantRatingStrategy

    // MARK: - Initializers

    init(withRaingClient client: FoodMerchantRatingStrategy) {
        foodMerchantRatingClient = client
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    // MARK: - Views

    @IBOutlet private var merchantNameTextField: UITextField!
    @IBOutlet private var ratingServiceNameLabel: UILabel!
    @IBOutlet private var ratingLabel: UILabel!
    @IBOutlet private var reviewLabel: UILabel!

    // MARK: - View Lifecycles

    override func viewDidLoad() {
        super.viewDidLoad()
        ratingServiceNameLabel.text = foodMerchantRatingClient.ratingServiceName
    }

    // MARK: - Actions

    @IBAction private func searchButtonTapped() {
        guard let merchantName = merchantNameTextField.text,
            !merchantName.isEmpty  else {
                return
        }
        foodMerchantRatingClient
            .fetchRatingforMerchant(named: merchantName,
                                success: { [weak self] (rating, review) in
                                    self?.ratingLabel.text = rating
                                    self?.reviewLabel.text = review
            })
    }
}
```

### 总结

如果有多个不同的行为，可以使用策略模式。但如果只有一个行为，并且不会再发生改变，直接把逻辑写到View Controller里面就好了。
