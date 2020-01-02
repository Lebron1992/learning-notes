> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

备忘录模式可以用来存储或者恢复一个对象，它包含以下三个部分：

![](http://upload-images.jianshu.io/upload_images/2057254-b9d3b4caa9ad8590.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **Originator**：被存储或者恢复的对象
- **Memento**：备忘录，代表一个已经存储的状态
- **CareTaker**：发起保存Originator的请求，然后收到一个Memento。CareTaker的作用对备忘录进行管理，负责保存备忘录，然后提供已保存的备忘录给Originator，用于恢复Originator的状态。

在iOS中，我们使用`Encoder`把originator的状态编码到一个备忘录中，然后使用`Decoder`解码备忘录，生成一个originator。例如，我们可以把一个遵循`Codable`的类型，用`JSONEncoder`和`JSONDecoder`进行编码和解码。

### 什么时候使用

当我们需要保存一个对象的状态，后续要恢复这个对象时，使用备忘录模式。

### 简单Demo

我们简单地设计一个游戏系统：

- **Originator**：`Game`，记录游戏的状态和动作
- **Memento**：在本例中，是`Data`，用`JSONEncoder`把`Game`编码成数据，再用`JSONDecoder`解码成`Game`对象
- **CareTaker**：`GameSystem`，执行存储和恢复的过程

代码如下：

#### Game

记录游戏的分数和级别，并执行一些有些的操作

```swift
class Game: Codable {
    class State: Codable {
        var score = 0
        var level = 1
    }
    lazy var state = State()

    func enterNextLebrl() {
        state.level += 1
    }

    func earnSomePoints() {
        state.score += 100
    }
}
```

#### GameSystem

定义了`encoder`和`decoder`，用`encoder`把`game`编码成数据，用`UserDefaults`保存；用`decoder`把数据转化成`Game`对象。

```swift
class GameSystem {
    private let encoder = JSONEncoder()
    private let decoder = JSONDecoder()

    func save(_ game: Game, for player: String) throws {
        let data = try encoder.encode(game)
        UserDefaults.standard.set(data, forKey: player)
    }

    func loadGame(for player: String) throws -> Game? {
        guard let data = UserDefaults.standard.data(forKey: player),
            let game = try? decoder.decode(Game.self, from: data) else {
                return nil
        }
        return game
    }
}
```

#### 游戏模拟

第一个玩家开始玩游戏，赚了100分，进入下一等级；然后把这个玩家的数据以`player1`作为key保存。

```swift
var game1 = Game()
game1.earnSomePoints()
game1.enterNextLebrl()

let gameSystem = GameSystem()
try gameSystem.save(game1, for: "player1")
```

第二个玩家开始玩游戏：

```swift
var game2 = Game()
print("第二个玩家的分数: \(game2.state.score)") // 第二个玩家的分数: 0
```

如果这时第二个玩家不想玩了，第一个玩家接着玩，恢复第一个玩家的数据：

```swift
var restoredGame1 = try! gameSystem.loadGame(for: "player1")
print("第一个玩家的分数: \(restoredGame1?.state.score ?? "nil")")
// 第一个玩家的分数: 100
```
