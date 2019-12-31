### 【Swift 3.1】20 - 嵌套类型 (Nested Types)

枚举通常是用来支持类或者结构的某些功能。同样地，在一些复杂类型里面，可以定义一些工具类和结构。Swift可以让我们定义嵌套类型。

#### 嵌套类型实践 (Nested Typeds in Action)

```swift
struct BlackjackCard {
    
    // nested Suit enumeration
    enum Suit: Character {
        case spades = "♠", hearts = "♡", diamonds = "♢", clubs = "♣"
    }
    
    // nested Rank enumeration
    enum Rank: Int {
        case two = 2, three, four, five, six, seven, eight, nine, ten
        case jack, queen, king, ace
        struct Values {
            let first: Int, second: Int?
        }
        var values: Values {
            switch self {
            case .ace:
                return Values(first: 1, second: 11)
            case .jack, .queen, .king:
                return Values(first: 10, second: nil)
            default:
                return Values(first: self.rawValue, second: nil)
            }
        }
    }
    
    // BlackjackCard properties and methods
    let rank: Rank, suit: Suit
    var description: String {
        var output = "suit is \(suit.rawValue),"
        output += " value is \(rank.values.first)"
        if let second = rank.values.second {
            output += " or \(second)"
        }
        return output
    }
}
```

`Suit`枚举描述了四个常见的扑克牌，并且有一个字符代表各自的符号。

`Rank`枚举定义了13个扑克牌等级，并有一个`Int`类型的原始值代表他们的牌面数字(除了J/Q/K和A)。`Rank`里面嵌套了一个`Values`结构。

因为`BlackjackCard`结构没有自定义初始化器，所以他有一个默认地逐一成员初始化器：

```swift
let theAceOfSpades = BlackjackCard(rank: .ace, suit: .spades)
print("theAceOfSpades: \(theAceOfSpades.description)")
// Prints "theAceOfSpades: suit is ♠, value is 1 or 11"
```
初始化器中可以直接使用case的名字`.ace`和`.spades`来引用枚举的case。

#### 引用嵌套类型 (Referring to Nested Types)

```swift
let heartsSymbol = BlackjackCard.Suit.hearts.rawValue
// heartsSymbol is "♡"
```
