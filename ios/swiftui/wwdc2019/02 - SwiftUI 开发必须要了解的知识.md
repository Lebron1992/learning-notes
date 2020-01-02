SwiftUI 是一个全新的框架，它是为了以最快的路径开发 App 而设计的。虽然它是全新的，但是它包含了大量常见的组件，如下图：


![](https://upload-images.jianshu.io/upload_images/2057254-057546b4bc36c552.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 UIKit 时代， 我们需要花大量的时间来写基本的 UI；而有了 SwiftUI 之后，我们可以把更多的时间花在自定义功能和业务逻辑上，借用了苹果的图，对比如下：

![](https://upload-images.jianshu.io/upload_images/2057254-a3d51d8d71f33189.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![](https://upload-images.jianshu.io/upload_images/2057254-7f81ea5fb0121104.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为什么说 SwiftUI 简化基本 UI 的编写？这里以显示一组数据为例：1）在 UIKit 中，我们使用 `UITableView`，需要实现 `UITableViewDataSource`；2）而在 SwiftUI 中，只需要一个 `List` 就可以搞定。下面看一下 `List`的 demo：

```swift
struct ContentView : View {
    let ints = [1, 2, 3, 4, 5]
    
    var body: some View {
        List(ints) { int in
            Text("\(int)")
        }
    }
}
```

## View

在 UIKit 中有 `UIView`，在 AppKit 中有 `NSView`，而在 SwiftUI 中也有类似的概念，叫做 `View`。

在 View 的层级管理上，SwiftUI 有很大的优势。假设有以下层级关系：

![](https://upload-images.jianshu.io/upload_images/2057254-eab6d1d56b8078ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在用 UIKit 实现时，我们需要先定义一堆的 `UIView`实例，然后再通过 `addSubview()` 去组成 View 的层级关系。而且从代码上看，我们很难一眼看出 View 之间的层级关系。但是在 SwiftUI 就不一样了，下面是 SwiftUI 的实现代码：

```swift
VStack {
    Text("Avocado Toast").font(.title)
    
    Toggle(isOn: $order.includeSalt) {
        Text("Include Salt")
    }
    Toggle(isOn: $order.includeRedPepperFlakes) {
        Text("Include Red Pepper Flakes")
    }
    Stepper(value: $order.quantity, in: 1...10) {
        Text("Quantity: \(order.quantity)")
    }
    
    Button(action: submitOrder) {
        Text("Order")
    }
}
```

在 SwiftUI 中，直接把 subview 放在 superview 初始化方法的一个 closure 参数里，而不需要再调用  `addSubview()`，View 之间的层级关系一目了然。

到这里，我们可以总结出：1）UIKit 的实现过程，其实是属于命令式编程。你需要一步步告诉程序做什么和怎么做，例如先把各种 view 初始化好，然后把这个 subview 添加到那个 superview 里面等；2）SwiftUI 的实现过程是声明式编程，你只需要告诉程序你需要什么、这个东西放置在哪里。举个简单的例子，假设你想让朋友做一个番茄炒蛋：命令式编程就像是你教朋友做番茄炒蛋，告诉他第一步把番茄洗干净，第二步把番茄切片 …… ；而声明式编程就是你直接告诉朋友，我要吃番茄炒蛋，就这么简单，至于怎么做就交给你朋友了。 

## Object Binding

从上面的代码我们看到有些传入的参数前面有 `$` ， `$`意思是我们传入的对象是一个 binding，而不是一个普通的对象。下面是 Binding 的语法：

```swift
struct OrderForm: View {
    @State private var order: Order
    
    var body: some View {
        Stepper(value: $order.quantity, in: 1...10) {
            Text("Quantity: \(order.quantity)")
        }
    }
}
```

Binding 属性被 `@State` 标记。当你看到一个属性携带着 `$` 被传入，那么意味着它是一个 Binding，允许被另一个 View 更改。在这个例子中，说明 `order.quantity` 的值可以被 `Stepper` 内部修改。

## Modifier

以上面讲解 View 的其中一行代码为例：

```swift
Text("Avocado Toast").font(.title)
```

前半部分 `Text("Avocado Toast")` 是一个 View，而后半部分 `font(.title)` 就是一个 Modifier。我们看一下这个 Modifier 的方法定义：`public func font(_ font: Font?) -> Text` ，很明显，Modifier 就是一个根据已经存在的 View 重新创建一个新的 View。另外还可以通过链式语法修改更多属性：

```swift
VStack {
    Text("Avocado Toast")
        .font(.title)
        .foregroundColor(.green)
		
	   ...
}
```

上面的代码会创建这样一个 View 层级：

![](https://upload-images.jianshu.io/upload_images/2057254-585ce9fd7a77fd73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每添加一个 Modifier，View层级就会不断变得复杂。但是，SwiftUI 可以很熟练根据 View 层级把我们的 Views 进行渲染。所以即使我们把 `Text`包装在多个 Wrapper Views，SwiftUI 会在底层整理成一个高效的数据结构，然后被渲染系统使用。

当我们不需要担心性能时，你会发现链式语法语法可以给我们带来好处。

第一，Modifier Chains 可以以可视化的效果执行确定顺序。 我们来看这组代码：

```swift
Text("🍞🥑")
    .background(Color.green, cornerRadius: 12)
    .padding(.all)

Text("🍞🥑")
    .padding(.all)
    .background(Color.green, cornerRadius: 12)
```

他们的渲染结果分别为：

![](https://upload-images.jianshu.io/upload_images/2057254-13483f12242ee485.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![](https://upload-images.jianshu.io/upload_images/2057254-3dd1334ce7d7c2fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


发现什么区别没？第一个例子看起来好像 `padding` Modifier 没有效果，但实际上是有的，只是周围的 padding 是白色背景而已。仔细分析代码，我们可以看到，第一个例子先执行 `background` 后执行 `padding` ，所以 `background` 对 `padding` 没有任何作用；而第二个例子，先执行 `padding` 后执行 `background` ，`background` 就能应用到 `padding`。

如果把 `background` 和 `padding` 定义为 `Text` 的属性，而不是独立的 Modifier，那么在不借助文档或者不断尝试运行代码的情况下，我们就无法知道哪个属性先应用到 `Text` 上。而通过链式语法把多个 Modifiers 链接在一起，就可以很明确知道对 `Text` 修改的顺序。

第二，Modifier 可以同时对多个 View 进行修改。例如：

我们可以把下面的代码：
```swift
VStack(alignment: .leading) {
    Toggle(isOn: $order.includeSalt) { ... }
        .opacity(0.5)
    Stepper(value: $order.quantity, in: 1...10) { ... }
        .opacity(0.5)
    Button(action: submitOrder) { ... }
        .opacity(0.5)
}
```

改为：

```swift
VStack(alignment: .leading) {
    Toggle(isOn: $order.includeSalt) { ... }
    Stepper(value: $order.quantity, in: 1...10) { ... }
    Button(action: submitOrder) { ... }
}
.opacity(0.5)
```

## 自定义 View

在使用 UIKit 时，所有的自定义 View，都需要继承自 `UIView`。在 SwiftUI 中，也是类似的思想，不同的是从 `UIView` （类）变成了 `View` （协议）。

我们仔细看一下 SwiftUI 中的 `View`定义：

```swift
public protocol View : _View {
    associatedtype Body : View
    var body: Self.Body { get }
}
```

不像 UIKit 中的 `UIView`存储了大量的属性，SwiftUI 中的 `View` 变得非常轻量化，只需要实现一个 `body` 属性即可，View 的属性全部通过 Modifiers 去修改。下面看一个官方视频中的一个自定义 View：

```swift
struct OrderHistory : View {
    let previousOrders: [CompletedOrder]
    
    var body: some View {
        List(previousOrders) { order in
            VStack(alignment: .leading) {
                Text(order.summary)
                Text(order.purchaseDate)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
        }
    }
}
```

在 SwiftUI 中，所有的自定义 View 都是通过 struct 去定义的，并实现 `View` 协议即可。我们前面讲过，SwiftUI 的核心思想是声明式编程，所以 SwiftUI 中的 View 不再是一个我们持续修改并永久存在的对象，而更像是一个纯函数，外部给我怎样的数据，我就给你怎样的 View，数据一样，返回的 View 就一样，这也是为什么 `View` 协议只有一个只读属性 `body`的原因。

上面代码中的 `List` 是一个演示声明式编程如此强大的例子：当 `previousOrders` 发生变化时，SwiftUI 会把新数据和旧数据进行对比，然后根据数据的变化非常高效地更新 UI，并且有对应的删除和增加的动画。这一整个复杂的过程，无需写一行代码；而在 UIKit 中，我们需要写很多代码去应对数据的变化。

## 参考资料

[SwiftUI Essentials - WWDC 2019 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2019/216/)