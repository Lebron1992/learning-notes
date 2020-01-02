> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

组合模式属于结构型模式，可以把多个对象整理成一个树状结构，把这些对象当做一个对象来处理。涉及以下三种类型：

- **Component 协议**：这个协议保证了可以使用同样的方式来处理整棵树中的对象
- **Leaf**：树结构中的一个没有子元素的组件
- **Composite**：用来存储 Leaf 对象和 Composite 的容器

所有的 Leaf 和 Composite 都遵循 Component 协议，所以我们可以在 Composite 中存储不同类型的 Leaf 对象。例如，数组是一个 Composite，Component可以是 String、Int 等，也可以是一个数组。

### 什么时候使用

如果类的层级结构形成了一个分支模式，我们分别创建分支和节点两种类型来处理的话，这些类之间将会变得难以交互。这种情况下，我们可以使用组合模式，通过让分支和节点遵循同一个协议，就可以把他们当做一个对象来处理。Component 协议在这个模式中，起到一个抽象的层的作用，减少他们的复杂程度。

### 简单demo

仅从上面的理论描述，是比较难理解这个模式的。这里以我们常见的电脑中的文件的层级关系来 demo 一下这个模式。

文件夹中，可以存储某一类具体类型的文件，例如`.pdf`、`.mp3`等，还可以存储文件夹。不管是具体的文件还是文件夹，在他们上面点击右键有一些共同的操作，例如**打开**、**删除**和**重命名**等。我们可以在文件夹中存储不同类型的文件和文件夹，就是因为他们都遵循了 Component 协议。

下面来看看代码。

#### Component 协议

首先定义 Component 协议：

```swift
protocol File {
    var name: String { get set }
    func open()
}
```

定义了`File`协议，所有的 Leaf 和 Composite 对象都要遵循这个协议。

#### Leaf

```swift
final class PDF: File {
    var name: String
    init(name: String) {
        self.name = name
    }
    
    func open() {
        print("正在打开\(name)")
    }
}

final class Music: File {
    var name: String
    var artist: String
    
    init(name: String, artist: String) {
        self.name = name
        self.artist = artist
    }
    
    func open() {
        print("正在播放\(artist)的\(name)")
    }
}
```

定义了两个 Leaf 类型，分别是`PDF`和`Music`，并且遵循`File`协议，各自实现了`open()`方法。

#### Composite

```swift
final class Folder: File {
    var name: String
    private(set) var files: [File] = []
    
    init(name: String) {
        self.name = name
    }
    
    func addFile(_ file: File) {
        files.append(file)
    }
    
    func open() {
        print("\n")
        print("正在显示以下文件：")
        files.forEach { print("--- \($0.name)") }
    }
}
```

`Folder`属于 Composite，实现了`File`协议，另外有一个数组可以存储其他 `File` 类型，也就意味着 `Folder`不仅可以存储`PDF`和`Music`，也可以存储其他`Folder`对象。

#### 使用

```swift
// 创建两个文件夹
let desktop = Folder(name: "桌面")
let musicFolder = Folder(name: "我最爱的音乐")

// 创建具体的文件
let iOSDesignPattern = PDF(name: "iOS设计模式")

let diYiCi = Music(name: "第一次", artist: "光良")
let liXiang = Music(name: "理想", artist: "赵雷")

// 桌面文件夹添加音乐文件夹和 PDF 文件
desktop.addFile(musicFolder)
desktop.addFile(iOSDesignPattern)

// 把两个音乐添加到音乐文件夹
musicFolder.addFile(diYiCi)
musicFolder.addFile(liXiang)

// 打开文件
iOSDesignPattern.open()
liXiang.open()

// 打开文件夹
desktop.open()
musicFolder.open()


// 结果
正在打开iOS设计模式
正在播放赵雷的理想

正在显示以下文件：
--- 我最爱的音乐
--- iOS设计模式

正在显示以下文件：
--- 第一次
--- 理想
```

从例子中可以看到，我们可以对不同的对象调用同一个方法，让程序变得非常简单。

### 总结

利用组合模式，我们可以对不同的对象以相同的方式处理。想象一下，如果没有 Component 协议，要创建一个文件的容器会变得有多复杂。但是在使用组合模式之前，首先要保证应用有分支结构。
