## 基本布局

在刚创建好一个 SwiftUI 项目时，Xcode 给我们准备了模板代码：

```swift
struct ContentView : View {
    var body: some View {
        Text("Hello World")
    }
}
```

预览如下：

![](https://upload-images.jianshu.io/upload_images/2057254-3f4cf254f0c9de51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

View 层级关系如下：

![](https://upload-images.jianshu.io/upload_images/2057254-507dbd0deba0c763.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本质上来说，这个例子有三个 View：1）最底层的 Root View，也就是整个手机屏幕除去留海屏的部分；2）处于中间的 `ContentView`，预览图体现不出来，因为它和 `Text` 一样大；2）最顶层的 `Text`。但是因为 `ContentView` 的大小是有它的 Child View `Text` 决定的，所以这里我们可以把这个 View 层级简化成只有  Root View 和 `Text`。

我们就以这个为例子，讲解一下 SwiftUI 的基本布局步骤：
1. Parent View 给 Child View 提供一个 size，这个 size 是 Parent View 所能提供的最大 size。在本例中就相当于 Root View 对 `Text` 说：“Hey， Text，我可以把整个屏幕大小（除了留海）的空间给你”。

![](https://upload-images.jianshu.io/upload_images/2057254-39d3beb2cd289110.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. Child View 选择自己的 size。在本例中，Text 说我只需要这么大就够了。

![](https://upload-images.jianshu.io/upload_images/2057254-4159539295e95e78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. Parent View 把 Child View 放在自己的坐标空间里。在本例中，Root View 把 `Text` 放在中间。

![](https://upload-images.jianshu.io/upload_images/2057254-e7d004add3fe6bd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这就是一个完整的 SwiftUI 布局流程。在 SwiftUI 中所有的 Parent View 与 Child View 之间的布局逻辑都是跟这里讲解的一样的。

## HStack 和 VStack

借用官方的 Demo，UI代码如下：

```swift
HStack {
    VStack {
        Text("★★★★★")
        Text("5 stars")
    }.font(.caption)
    VStack(alignment: .leading) {
        HStack {
            Text("Avocado Toast").font(.title)
            Spacer()
            Image("20x20_avocado")
        }
        Text("Ingredients: Avocado, Almond Butter, Bread, Red Pepper Flakes")
            .font(.caption).lineLimit(1)
    }
}
```

预览效果：

![](https://upload-images.jianshu.io/upload_images/2057254-2dcdff5b888f1113.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Stack 可以自动为它的 Child Views 之间添加 spacing。如果用户使用的是阿拉伯语，SwiftUI 可以自动切换文字的方向，如下图：

![](https://upload-images.jianshu.io/upload_images/2057254-0ceeb8b64a4fece8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们拿 `HStack` 作为例子，讲解一下 Stack 是如果工作的。假设有以下代码：

```
HStack {
    Text("Delicious")
    Image("20x20_avocado")
    Text("Avocado Toast")
}
.lineLimit(1)
```

当水平方向上有足够空间的情况下，预览图如下：

![](https://upload-images.jianshu.io/upload_images/2057254-6c6e4957dddddddf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 首先 HStack 先预留合适的宽度给 spacing，所以 `HStack` 的可用空间为所有可用宽度减去 `S0 + S1`：

![](https://upload-images.jianshu.io/upload_images/2057254-cd69d6d4add3af21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![](https://upload-images.jianshu.io/upload_images/2057254-88031f0a2cd25327.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


2. 因为总共有三个 Child Views，所以 `HStack` 先把剩余可用宽度平均分配给三个 Child Views：

![](https://upload-images.jianshu.io/upload_images/2057254-d3dbba9290bdf4c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 因为 `Image` 的大小是固定的，所以 `Image` 先确定它的所需宽度，然后从可用宽度减去 `Image` 的宽度：

![](https://upload-images.jianshu.io/upload_images/2057254-63e233cf6d5cbfb4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/2057254-1c51f65d5342b77d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 剩下两个 Child Views，所以 `HStack` 又把剩余可用宽度平均分配给两个 Child Views：

![](https://upload-images.jianshu.io/upload_images/2057254-9150094bb5b956b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5.  `Delicious` 确定它的所需宽度，`HStack` 从可用宽度减去 `Delicious` 的宽度：

![](https://upload-images.jianshu.io/upload_images/2057254-85b2b0094944f5fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![](https://upload-images.jianshu.io/upload_images/2057254-8cbf17dbf82784c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6. 剩下的可用宽度全部分配给 `Avocado Toast`：

![](https://upload-images.jianshu.io/upload_images/2057254-5a90dda98ca35300.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

7. 所有 Child Views 水平方向的位置都确定了，那么剩下垂直方向。`HStack`垂直方向的对齐方式默认是 `center`，所以完整的代码如下：

```swift
// alignment 默认是 center
HStack(alignment: .center) {
    Text("Delicious")
    Image("20x20_avocado")
    Text("Avocado Toast")
}
.lineLimit(1)
```

### Layout Priority

如果水平方向的空间不够，那么就会造成下面这种效果：

![](https://upload-images.jianshu.io/upload_images/2057254-400eea3e3e38c0b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时我们觉得后面的文字比较重要，想让它优先显示完整，可以使用 Layout Priority。代码如下：

```swift
HStack {
    Text("Delicious")
    Image("20x20_avocado")
    Text("Avocado Toast").layoutPriority(1)
}
.lineLimit(1)
```

结果为：

![](https://upload-images.jianshu.io/upload_images/2057254-87a899458ac42e0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用了 Layout Priority 后，`HStack` 就会优先满足除了最大优先级外的所有 Child Views 的最小所需宽度。前面文字的所需最小宽度是 `...` 的宽度，图片所需最小宽度就是它的宽度，所以 `HStack` 把所有可用宽度减去前面文字和图片的所需最小宽度，剩下的可用宽度全部分配给 `Avocado Toast`。如果还有更多较低优先级的 Child Views，那么会重复使用同样的逻辑去进行宽度的分配。

### Alignment

上面的例子中，我们使用的是 `center`，我们可以改为 `bottom`，代码如下：

```swift
HStack(alignment: .bottom) {
    Text("Delicious")
    Image("20x20_avocado")
    Text("Avocado Toast")
}
.lineLimit(1)
```

结果如下：

![](https://upload-images.jianshu.io/upload_images/2057254-6e02c28255756ea6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果我们把前面文字的字体改小，代码和结果如下：

```swift
HStack(alignment: .bottom) {
    Text("Delicious").font(.caption)
    Image("20x20_avocado")
    Text("Avocado Toast")
}
.lineLimit(1)
```

![](https://upload-images.jianshu.io/upload_images/2057254-4d1162d7daeced90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样看起来三个 Child Views 的最低部不是在同一条线上。这时我们可以把 `alignment` 改为 `lastTextBaseline`。代码和结果如下：

```swift
HStack(alignment: .lastTextBaseline) {
    Text("Delicious").font(.caption)
    Image("20x20_avocado")
    Text("Avocado Toast")
}
.lineLimit(1)
```

![](https://upload-images.jianshu.io/upload_images/2057254-55951e04af50c953.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在我们把注意力放到图片里，我们可以看到这个图片里没有文字，但是 `lastTextBaseline` 默认是 View 的最底部，所以我们才能达到我们想要的对齐效果。但是我们的产品经理觉得图片的位置太靠上了，这样不好看，想要把图片往下移一点点，想要的效果如下：

![](https://upload-images.jianshu.io/upload_images/2057254-8f870db1ec3a1323.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时我们可以自定义  `lastTextBaseline` 的位置，代码如下：

```swift
HStack(alignment: .lastTextBaseline) {
    Text("Delicious")
        .font(.caption)
    Image("20x20_avocado")
        .alignmentGuide(.lastTextBaseline) { d in
            d[.bottom] * 0.927
    }
    Text("Avocado Toast")
}
.lineLimit(1)
```

`d[.bottom]` 这种写法是使用了 Swift 的 subscript 的特性。

## 自定义 VerticalAlignment

回到这一部分开头的例子，假设我们想让星星与 `Avocado Toast` 垂直方向上居中对齐，只设置 `alignment` 为 `center` 是不行的。

![](https://upload-images.jianshu.io/upload_images/2057254-8b1413c9082ed743.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

要达到想要的效果，我们需要自定义 VerticalAlignment。代码如下：

```swift
extension VerticalAlignment {
    private enum MidStarAndTitle: AlignmentID {
        static func defaultValue(in d: ViewDimensions) -> Length {
            return d[.bottom]
        }
    }
    static let midStarAndTitle = VerticalAlignment(MidStarAndTitle.self)
}
```

先定义一个遵循 `AlignmentID` 协议的 `MidStarAndTitle`，然后再用 `MidStarAndTitle` 实例化 `VerticalAlignment`。这样我们就可以把原有代码修改为：

```swift
HStack(alignment: .midStarAndTitle) {
    VStack {
        Text("★★★★★")
            .alignmentGuide(.midStarAndTitle) { d in d[.bottom] / 2 }
        Text("5 stars")
    }.font(.caption)
    
    VStack(alignment: .leading) {
        HStack {
            Text("Avocado Toast").font(.title)
                .alignmentGuide(.midStarAndTitle) { d in d[.bottom] / 2 }
            Spacer()
            Image("money")
        }
        Text("Ingredients: Avocado, Almond Butter, Bread, Red Pepper Flakes")
    }
    .font(.caption).lineLimit(1)
}
```

最终得到我们想要的效果：

![](https://upload-images.jianshu.io/upload_images/2057254-10f7c5608d116c30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至于代码中的 `alignmentGuide()` 是如果工作的，你把代码运行起来，修改 closure 里的值，看看具体效果。

## 参考资料

[Building Custom Views with SwiftUI - WWDC 2019 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2019/237/)
