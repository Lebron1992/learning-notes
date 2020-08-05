**### 1. 你如何描述 Swift 这门语言？**

Swift 是一种语法简洁、面向协议编程、类型安全的现代化语言。以下是 Swift 的一些特性：

- 每行代码末尾不需要 `;`

```swift
var name = "Lebron"
name = Davis
```

- 自动类型推断

```swift
// 自动推断为 `String` 类型
var name = "Lebron"
```

- 定义属性时直接设置默认值

```swift
struct Player {
    var name: String
    var highScore: Int = 0 // 设置默认值
    var history: [Int] = []

    init(_ name: String) {
        self.name = name
    }
}

var player = Player("Tomas")
```

- 使用 `extension` 向已有类型添加功能

```swift
extension Player {
    mutating func updateScore(_ newScore: Int) {
        history.append(newScore)
        if highScore < newScore {
            print("\(newScore)! A new high score for \(name)! 🎉")
            highScore = newScore
        }
    }
}

player.updateScore(50)
```

- 快速扩展自定义类型以利用强大的 Swift 语言功能，如自动JSON编码和解码

```swift
extension Player: Codable, Equatable {}

import Foundation
let encoder = JSONEncoder()
try encoder.encode(player)

print(player)
```

- 协议扩展让编写通用代码更容易
- 元组(`Tuple`)和多个返回值
- 区别于 `class` 类型的 `struct`
- 枚举可以携带关联值并支持模式匹配
- `Optional` 类型
- ……

**### 2. `var` 和 `let` 有什么区别？你会在 `struct` 中选择哪个作为 property？为什么？**

`var` 用于定义变量，可以随时更改变量的值；`let` 用于定义常量，常量一旦赋值后，就无法再更改。

在 `struct` 中选择哪个作为 property要视情况而定：通常 Data Model 的所有属性都定义为 `let`，其他的要视情况而定。


**### 3. 什么是 lazy property？**

Lazy property 是指只有在第一次用到它时才会初始化的属性，用于提高运行效率。通常在以下情况下使用：1）属性初始值依赖于外部条件；2）属性的初始值需要经过复杂的计算，而这个计算只有在用到它时才需要执行。

Lazy property 只能用 `var` 定义，因为它的初始值在实例完成初始化后不一定能最终确定，而 `let` 定义的属性在实例完成初始化后必须有值，所以只能用 `var`。

另外，如果 lazy property 在初始化前在多个线程中同时被访问，那就无法保证它只被初始化一次，有可能是多次。

**### 4. 什么是 `Optional`？其背后的机制是什么**

`Optional` 是可选类型，其本质是一个枚举，有 `none` 和 `some(Wrapped)` 两个 cases。

**### 5. 如何 unwrap 一个 `Optional` 值？附加问题：什么是，optional binding以及nil-coalescing operator？举例说明你会在何种情况下选用哪种方法。这道题很简单，但目的只在于看你是否知道其中的区别，而guard并不总是首选。**

假设有这个 `Optional` 值：`let number = Int("42")`，可以通过一下方式实现 unwrap：

- Unconditional Unwrapping，强制解包：

```swift
let number = Int("42")!
```

- Optional Binding，可选绑定，其实就是 `if let` 和 `guard let`：

```swift
if let number = number {

}

guard let number = number else {
  return
}
```

- Nil-Coalescing Operator，空合运算符，也就是 `??`：

```swift
let number = Int("42") ?? 0
```

****你会在何种情况下选用哪种方法？****

- 当确定一个 `Optional` 类型的实例有值时，可以使用强制解包
- 当不确定是否有值，并且在不能设置默认值的情况下，使用可选绑定，通常使用 `if let`；而 `guard let` 一般在有值才继续执行后面所有代码的情况下使用。
- 当不确定是否有值，并且在可以设置默认值的情况下，使用空合运算符

**### 6. 什么是 Optional Chaining？**

Optional Chaining 是指使用 `?` 来访问 `Optional` 实例的成员(属性、方法和下标)。例如：

```swift
class Person {
    var residence: Residence?
}

class Residence {
    var numberOfRooms = 1
}

let john = Person()
let roomCount = john.residence?.numberOfRooms
```

**### 7. class 和 struct 有什么区别？举例说明分别什么情况下应该选用。**

****class 比 struct 多了以下特性：****

- 支持继承
- 支持在运行时通过类型转换检查实例的具体类型
- 可以在 `deinit` 方法释放资源
- 引用计数允许对一个 class 实例有多个引用

****其他区别：****

- 如果不自定义初始化器，struct会自动获得一个成员初始化器，而 class 没有
- struct 是值类型，而 class 是引用类型。

****如何选择？****

- 优先考虑 struct：因为 struct 是值类型，不需要考虑它在应用中的传递，使用起来会更简单，写出更自信的代码。
- 与 Objective-C 混编时，只能选择 class
- 需要共享一个实例时，使用 class；其他情况使用 struct
- 

**### 8. 什么是 closure？什么是 autoclosure？**

Closure 是包含特定功能的代码块，可以在代码中传递和使用，类似于 Objective-C 的 block。在它被定义的时，可以捕获当时环境中的变量。closure 属于引用类型。

Closure 有三种表现形式：全局方法、嵌套方法和 closure 表达式。

Autoclosure 用于标记参数类型，他可以把传入的一行代码语句自动变成 closure。例如：

```swift
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0))
```

Autoclosure 一般很少使用，会影响性能，更重要的是影响代码可读性。

**### 9. escaping 是什么意思？**

Escaping 用于标记 closure 类型的参数，被它标记的 closure 会在 closure 所在的方法返回之后才会执行。例如：

```swift
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler) // completionHandler 没有马上执行
}
```

在调用含有 escaping closure 参数的方法时，在传入的 closure 参数中访问实例变量，需要明确使用 self：

```swift
func someFunctionWithNonescapingClosure(closure: () -> Void) {
    closure()
}

class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 } // 必须使用 self
        someFunctionWithNonescapingClosure { x = 200 } // 可有可无，通常不写 self
    }
}
```

**### 10. weak 和 unowned 是什么意思？二者有什么不同？**

Weak 和 unowned 都是用于解决 class 之间的循环应用问题。

两者的区别：造成循环应用的两个 class，如果他们的生命周期完全不同，则使用 weak；如果生命周期相同或者其中一个class 的生命周期更长，则使用 unowned。例如：假设有两个 class 实例 A 和 B，如果他们之间任何一个销毁，都不会造成对方被销毁，也就是生命周完全不同，这时使用 weak；如果有其中一个销毁，会导致对方被销毁，那么使用 unwoned。

虽然每次使用 weak 可能也没问题，但还是应该知道它们的区别。

**### 11. mutating 关键字是什么意思？**

在值类型的实例方法中修改 self 或者属性值，那么这个实例方法必须使用 mutating 关键字标记。

**### 12. 你听说过 method swizzling 吗？是什么意思？在 Swift 中可以用吗？**

Method swizzling 用来修改已存在的 selector 映射的方法。

在 Swift 中也可以用，但必须满足以下条件：

- 包含 Swizzle 方法的类需要继承自 NSObject
- 需要 Swizzle 的方法必须有动态属性（dynamic attribute）

**### 13. Array 和 Set 有什么区别？**

Set 的元素是唯一的，并且是无序的。

Array 可以根据索引访问，并且便于遍历元素；Set的插入和删除速度更快。

**### 14. 什么是 KVO 和 KVC？**

- KVO 全称 Key-Value Observing，是苹果提供的一套事件通知机制。允许对象监听另一个对象特定属性的改变，并在改变时接收到事件。
- KVC 全称 Key-Value Coding，可以允许开发者通过 key 名直接访问对象的属性，或者给对象的属性赋值。而不需要调用明确的 getter 和 setter 方法。这样就可以在运行时动态地访问和修改对象的属性。而不是在编译时确定。

都需要集成 NSObject。

**### 15. 什么是 REST？**

REST 是一种设计 API 的模式

**### 16. GCD 和 NSOperation 之间有什么异同？**

- 从易用性角度，GCD 由于采用 C 风格的 API，在调用上比使用面向对象风格的 NSOperation 要简单一些
- 从对任务的控制性来说，NSOperation 好于 GCD，NSOperation 支持 Cancel 操作，支持任务之间的依赖关系，支持同一个队列中任务的优先级设置，同时还可以通过 KVO 来监控任务的执行情况

**### 17. 什么是串行/并行队列？**

- 串行/并行队列中的任务都是按顺序执行
- 串行队列中，上一个任务执行完成后，下一个任务才开始执行，执行开始的顺序和完成的顺序一致。
- 并行队列中，无需等待上一个任务执行完成，就可以开始执行下一个任务，任务之间不会等待谁，而是自己执行自己的。

**### 18. 什么是 DispatchGroup？**

DispatchGroup 可以用于分组管理异步任务。

**### 19. 什么是 defer？**

Defer 语句用于在它所在的作用域即将执行完毕、切换到其他作用域之前执行一些代码。通常用于一些资源回收操作。

53. 什么是泛化（generics）？描述一下泛化是如何改善我们的生活的，并从Swift标准库里举例说明。

**### copy 和 retain 之间有什么区别？**

- copy 与 strong 类似，不过它会复制对象的值，原对象不发生任何改变，一般 copy 用在 NSString、NSArray、NSDictionary 等属性字段的修饰符
- retain 与 strong 为同样的功能，retain 为 ARC 出现之前的关键字

区别在于 copy 会复制原对象的值。

**### Objective-C中的 atomic/nonatomic 是什么？**

- atomic 系统自动生成的 getter/setter 方法会进行加锁操作
- nonatomic 系统自动生成的 getter/setter 方法不会进行加锁操作

Atomic 可以用于多线程，加锁保证了属性在多线程的情况下不出现数据错误。但是有了加锁这一层操作，就会消耗更多的系统性能和资源。nonatomic 不适合用在多线程的情况下，相比 atomic 性能更好。
