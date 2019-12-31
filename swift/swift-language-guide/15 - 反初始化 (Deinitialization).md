### 反初始化 (Deinitialization)

反初始化器在一个类的实例被释放之前调用。反初始化器只适用于class类型。

#### 反初始化器如果工作 (How Deinitialization Works)

当一个类的实例不再需要的时候，Swift会自动释放。Swift是通过自动引用计数(automatic reference counting，简称ARC)来管理内存的，不需要手动管理。然后，在默写情况下是需要自己去释放资源的。例如，如果我们创建了一个自定义的类去打开一个文件或者把数据写入到文件，我们必须在这个类的实例被释放之前手动关闭文件。

每个类最多只有一个反初始化器：

```swift
deinit {
    // perform the deinitialization
}
```

反初始化器在一个类的实例被释放之前调用，我们不能自己调用反初始化器。父类的反初始化器会被子类继承，并且父类的反初始化器会在子类反初始化器的实现后面被调用。即使子类没有提供自己的反初始化器，父类的反初始化器也会被调用。

因为在反初始化器被调用之前，实例还没有被释放，所以在反初始化器里面可以访问所有实例属性，并用这些属性来执行一些相关的代码。

#### 反初始化器实践 (Deinitializers in Action)

创建`Bank`和`Player`两个类：

```swift
class Bank {
    static var coinsInBank = 10_000
    static func distribute(coins numberOfCoinsRequested: Int) -> Int {
        let numberOfCoinsToVend = min(numberOfCoinsRequested, coinsInBank)
        coinsInBank -= numberOfCoinsToVend
        return numberOfCoinsToVend
    }
    static func receive(coins: Int) {
        coinsInBank += coins
    }
}

class Player {
    var coinsInPurse: Int
    init(coins: Int) {
        coinsInPurse = Bank.distribute(coins: coins)
    }
    func win(coins: Int) {
        coinsInPurse += Bank.distribute(coins: coins)
    }
    deinit {
        Bank.receive(coins: coinsInPurse)
    }
}
```

创建一个`Player`实例：

```swift
var playerOne: Player? = Player(coins: 100)
print("A new player has joined the game with \(playerOne!.coinsInPurse) coins")
// Prints "A new player has joined the game with 100 coins"
print("There are now \(Bank.coinsInBank) coins left in the bank")
// Prints "There are now 9900 coins left in the bank"
```

执行`winCoins(_:)`方法：

```swift
playerOne!.win(coins: 2_000)
print("PlayerOne won 2000 coins & now has \(playerOne!.coinsInPurse) coins")
// Prints "PlayerOne won 2000 coins & now has 2100 coins"
print("The bank now only has \(Bank.coinsInBank) coins left")
// Prints "The bank now only has 7900 coins left"
```

把`playerOne`设置为nil，意味着`playerOne`不再引用`Player`实例，那么反初始化器被调用，`receive(coins:)`方法被执行，玩家所有的游戏币返回给银行。
