> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

门面模式属于结构模式，它给一个复杂的系统提供了简单的接口。它涉及到两个部分：

- **Facade**：提供了与系统交互的简单方法，它让使用者直接使用facade，而不需要知道系统里的类，也不需要跟他们交互。
- **Dependencies**：Facade持有的对象，每个dependency处理一部分复杂的任务。

### 什么时候使用

当我们有一个由很多组件组成的系统时，并且需要提供一个简单的方式给用户执行复杂的任务时，使用这个模式。例如电商的下单系统，他会涉及到顾客、产品、库存清单等等。

### 简单demo

这里我们就以刚刚提到的电商下单系统为例。首先我们需要`Customer`和`Product`数据模型；然后是库存`InventoryDatabase`；用户下单后，把已购买的产品存到待发货数据库`ShippingDatabase`中。实际开发中，肯定还需要顾客数据库和发票的处理等等，这里为了简便，我们就不考虑这些了。

#### `Customer`和`Product`数据模型

```swift
// MARK: - Customer
struct Customer {
    let id: String
    var address: String
    var name: String
}
extension Customer: Hashable {
    var hashValue: Int {
        return id.hashValue
    }

    static func ==(lhs: Customer, rhs: Customer) -> Bool {
        return lhs.id == rhs.id
    }
}

// MARK: - Product
struct Product {
    let id: String
    var name: String
    var price: Double
}
extension Product: Hashable {
    var hashValue: Int {
        return id.hashValue
    }

    static func ==(lhs: Product, rhs: Product) -> Bool {
        return lhs.id == rhs.id
    }
}
```

这里简单地创建了`Customer`和`Product`。因为后面要把他们作为字典的key，所以都实现了`Hashable`协议。

#### `InventoryDatabase`和`ShippingDatabase`数据库

```swift
final class InventoryDatabase {
    var inventory: [Product: Int] = [:]
    init(inventory: [Product: Int]) { self.inventory = inventory }
}

final class ShippingDatabase {
    var pendingShipments: [Customer: [Product]] = [:]
}
```

`InventoryDatabase`记录了某个产品对应的数量有多少个；`ShippingDatabase`记录了某个客户对应的待发货的产品。

#### Facade

上面已经准备好了一系列的工作，下面该创建我们的门面了。

```swift
final class OrderFacade {
    let inventoryDatabase: InventoryDatabase
    let shippingDatabase: ShippingDatabase

    init(inventoryDatabase: InventoryDatabase,
         shippingDatabase: ShippingDatabase) {
        self.inventoryDatabase = inventoryDatabase
        self.shippingDatabase = shippingDatabase
    }

    func placeOrder(for product: Product, by customer: Customer) {
        let productCount = inventoryDatabase.inventory[product, default: 0]

        guard productCount > 0 else {
            print("\(product.name)缺货")
            return
        }

        inventoryDatabase.inventory[product] = productCount - 1

        var pendingShipmentProducts = shippingDatabase.pendingShipments[customer, default: []]
        pendingShipmentProducts.append(product)
        shippingDatabase.pendingShipments[customer] = pendingShipmentProducts

        print("\(customer.name)购买了\(product.name)")
    }
}
```

在订单门面里，持有了`InventoryDatabase`和`ShippingDatabase`实例。还提供了一个下单的方法：先查询要下单的产品的库存，如果库存为0，则提示缺货；如果库存足够，把库存减一，并加到待发货数据库中。

#### 使用

```swift
let cola = Product(id: UUID().uuidString, name: "可乐", price: 3)
let sprite = Product(id: UUID().uuidString, name: "雪碧", price: 3)

let inventoryDatabase = InventoryDatabase(inventory: [cola: 10, sprite: 10])
let shippingDatabase = ShippingDatabase()

let orderFacade = OrderFacade(inventoryDatabase: inventoryDatabase,
                              shippingDatabase: shippingDatabase)

let customer = Customer(id: UUID().uuidString,
                        address: "深圳南山区xx街道xx小区10栋",
                        name: "Lebron James")

orderFacade.placeOrder(for: cola, by: customer)

// 打印
Lebron James购买了可乐
```

这里，存入了10瓶可乐和10瓶雪碧到库存中，然后用`orderFacade`去下单。

至此，我们完成了一个订单门面。

### 总结

在实际开发中，我们可能会创建一个或多个门面。如果一个门面有多个方法分别在不同的类使用，那么需要考虑把这个门面分成多一个门面。
