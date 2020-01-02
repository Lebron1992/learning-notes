这篇文章我们来看一下在 SwiftUI 中如何将数据作为依赖连接起来，同时保持 UI 的显示是正确并可预测的。这里主要讲解 SwiftUI 中的五个数据流工具：`Property`、`@State`、`@Binding`、`@ObjectBinding` 和 `@EnvironmentObject`。

## 数据流工具

### Property

Property 是我们目前开发中最常见的，它就是一个简单的属性，没什么特别。例子：

```swift
struct ContentView : View {
    var body: some View {
        ChildView(text: "Demo")
    }
}

struct ChildView: View {
    let text: String
    
    var body: some View {
        Text(text)
    }
}
```

`ChildView` 需要 Parent View 给它传一个字符串，并且 `ChildView` 本省不需要对这个字符串进行修改，所以直接定义一个 Property，在使用的时候，直接让 Parent View 告诉它就好了。

### @State

我们先看一个官方给的错误例子：

```swift
struct PlayerView : View {
    let episode: Episode
    var isPlaying: Bool
    
    var body: some View {
        VStack {
            Text(episode.title)
            Text(episode.showTitle).font(.caption).foregroundColor(.gray)
            
            Button(action: {
                // 错误：Cannot use mutating member on immutable value: 'self' is immutable
                self.isPlaying.toggle()
            }) {
                Image(systemName: isPlaying ? "pause.circle" : "play.circle")
            }
        }
    }
}
```

上面的代码中，我们想在 `Button` 被点击后直接使用 `self.isPlaying.toggle()` 切换 `isPlaying` 的值，但这是不行的，因为 `PlayerView` 是 struct 类型，`self` 是不可变的，并且 `isPlaying` 是一个普通的属性。为了达到我们的需求，`@State` 的作用就来了。我们把上面的代码改成：

```swift
struct PlayerView : View {
    let episode: Episode
    @State private var isPlaying: Bool = false
    
    var body: some View {
        VStack {
            Text(episode.title)
            Text(episode.showTitle).font(.caption).foregroundColor(.gray)
            
            Button(action: {
                self.isPlaying.toggle()
            }) {
                Image(systemName: isPlaying ? "pause.circle" : "play.circle")
            }
        }
    }
}
```

我们用 `@State` 标记 `isPlaying` 属性，这样 `isPlaying`  就可以在 View 的内部被更改，并且被更改后，与 `isPlaying`  相关的 View 也会更新，本例中 `Image` 就会在 `pause.circle` 和 `play.circle` 之间切换。

总结：`@State` 的作用是让被它标记的属性可以在 View 内部修改，并且 View 也会重新渲染。

### @Binding

有时候我们想让 Child View 修改 Parent View 传给它的数据，并且数据修改后，Parent View 重新渲染。这时我们就得用到 `@Binding`。

我们把 `@State` 例子中的 `Button` 重构为 `PlayButton`，代码如下：

```swift
struct PlayerView : View {
    let episode: Episode
    @State private var isPlaying: Bool = false
    
    var body: some View {
        VStack {
            Text(episode.title)
            Text(episode.showTitle).font(.caption).foregroundColor(.gray)
            PlayButton(isPlaying: $isPlaying)
        }
    }
}

struct PlayButton : View {
    @Binding var isPlaying: Bool
    
    var body: some View {
        Button(action: {
            self.isPlaying.toggle()
        }) {
            Image(systemName: isPlaying ? "pause.circle" : "play.circle")
        }
    }
}
```

在 `PlayButton` 中，用 `@Binding` 标记 `isPlaying` 属性，意味着可以对传入的数据进行修改；在 `PlayerView` 使用时，传入的属性 `isPlaying` 需要有 `$` 前缀，并且被传入属性不能是普通的属性，而要求是可读可写的属性（被`@State` / `@Binding` /  `@ObjectBinding` 标记）。

`@Binding` 在很多系统自带的 View 中使用，如 `Toggle`、`TextField` 和 `Slider` 等等。

### @ObjectBinding 

其实在很多情况下，我们的数据来源于外部的数据模型。我们也想要在当外部数据发生变化时，能及时更新我们的 UI。而 `@ObjectBinding` 就是为这种需求而设计的。

对于 `@ObjectBinding`标记的属性，它必须遵循 `BindableObject`  协议，这个协议的定义如下：

```swift
public protocol BindableObject : AnyObject, DynamicViewProperty, Identifiable, _BindableObjectViewProperty {
    associatedtype PublisherType : Publisher where Self.PublisherType.Failure == Never
    var didChange: Self.PublisherType { get }
}
```

`Publisher` 是与 SwiftUI 一起推出的响应式编程框架 Combine 的一个协议。所以想要熟练使用 `BindableObject`， 学习 Combine 是必不可少的。

下面是 `@ObjectBinding` 的演示代码：

```swift
class MyModelObject : BindableObject {
    var didChange = PassthroughSubject<Void, Never>()
    
    func changeData() {
        // 修改数据
        // ...
        
        // 通知订阅者数据发生变化
        didChange.send()
    }
}

struct MyView : View {
    @ObjectBinding var model: MyModelObject
    
    // ...
}
```

当调用 `didChange.send()` 之后，`MyView` 接收到通知，View 就会重新渲染。

### @ EnvironmentObject

我们刚刚学习的 `Property` 和 `@Binding` 都只能从 Parent View 一层一层的往 Child View 传递。所以当我们的 View 层级关系比较复杂、有些属性只在很深层级的 View 才用到时，用 `Property` 和 `@Binding` 的方式就会非常麻烦。苹果使用 `@ EnvironmentObject` 来解决这个问题。

我们先看一个 demo，然后通过 demo 来讲解 `@ EnvironmentObject`  的使用。

```swift
class MyModelObject : BindableObject {
    var didChange = PassthroughSubject<Void, Never>()
    var count = 0
    
    func updateCount() {
        count += 1
        didChange.send()
    }
}

struct ContentView : View {
    var body: some View {
        RootView().environmentObject(MyModelObject())
    }
}

struct RootView: View {
    var body: some View {
        VStack(spacing: 20) {
            ChildView1()
            ChildView2()
        }
    }
}

struct ChildView1: View {
    @EnvironmentObject var model: MyModelObject

    var body: some View {
        Button(action: {
            self.model.updateCount()
        }) {
            Text("Button")
        }
    }
}

struct ChildView2: View {
    @EnvironmentObject var model: MyModelObject
    
    var body: some View {
        Text("\(model.count)")
    }
}
```

`RootView` 包含了 `ChildView1` 和 `ChildView2`，两个 Child View 都持有被 `@EnvironmentObject`  标记的 `MyModelObject` 类型的属性，当 `ChildView1` 的按钮被点击时，`MyModelObject` 的数据被更新，`ChildView2` 的 View 重新渲染。整个过程中两个 Child View 没有从 `RootView` 中直接接受参数，只有 `RootView`在初始化的时，通过 `environmentObject()` 方法把 `MyModelObject` 注入到整个 View 层级中，这个层级中所有的 View 都可以通过 `@Environment`的方式访问 `MyModelObject` 。需要注意的一点是，使用 `environmentObject()` 注入的对象必须是 `BindableObject` 类型。

## 五个数据流工具总结

- **Property**：当 View 所需要的属性只要求可读，则使用 Property。
- **@State**： 当 View 所需要的属性只在当前 View 和它的 Child Views 中使用，并且在用户的操作过程中会发生变化，然后导致 View 需要作出改变，那么使用 `@State`。 因为只在当前 View 和它的 Child Views 中使用，跟外界无关，所以被 `@State` 标记的属性一般在定义时就有初始值。
- **@Binding**：当 View 所需要的属性是从它的直接 Parent View 传入，在内部会对这个属性进行修改，并且修改后的值需要反馈给直接 Parent View，那么使用 `@Binding`。
- **@ObjectBinding**：用于直接绑定外部的数据模型和 View。
- **@EnvironmentObject**：Root View 通过 `environmentObject()` 把 `BindableObject` 注入到 View  层级中，其中的所有 Child Views 可以通过 `@EnvironmentObject` 来访问被注入的 `BindableObject`。

## 接收其他外部变化

有时我们的 View 需要监听外部的其他变化，并做出相应的改变，可以使用 `receive(on:)`，这里面的 closure 参数是在主线程执行的。

以下是官方的 Demo 代码：

```swift
struct PlayerView : View {
    let episode: Episode
    @State private var isPlaying: Bool = true
    @State private var currentTime: TimeInterval = 0.0
    
    var body: some View {
        VStack {
            // ...
            Text("\(playhead, formatter: currentTimeFormatter)")
        }
        .onReceive(PodcastPlayer.currentTimePublisher) { newCurrentTime in
            self.currentTime = newCurrentTime
        }
    }
}
```


## 总结

数据在整个 App 中是非常重要的一部分，在使用上面讲到的工具之前，先仔细研究自己的数据结构，然后选择合适的工具，把数据注入到 UI 中。

## 参考资料

[Data Flow Through SwiftUI - WWDC 2019 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2019/226/)