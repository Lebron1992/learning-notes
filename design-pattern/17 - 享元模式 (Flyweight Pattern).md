> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

享元模式属于创建型模式，通过复用内存中的对象，减少内存的占用。

这个模式涉及两个部分：1）flyweight对象；2）创建 flyweight 的方法。

```
   +---------------------------------------------+ 
   |                Flyweight                    |
   +---------------------------------------------+ 
   |  static flyweight: SomeObjectType           |
   +---------------------------------------------+ 
   |  static flyweight(for:) -> SomeObjectType   |
   +---------------------------------------------+ 
```

其实，享元模式很像单例模式，他们都有让整个程序共享某个对象的特性。但是，在享元模式中，可以拥有多个同一类型的对象。例如，我们`UIColor`中的各种系统自带的颜色，每一种颜色都是一个单例。

### 什么时候使用

在需要使用多个不同配置的单例时使用。

### 简单demo

#### iOS 中用到的享元模式

上面提到了`UIColor`其实是使用了享元模式，我们来验证一下：

```swift
let black1 = UIColor.black
let black2 = UIColor.black
print(black1 === black2)  // true
```

结果是 `true`，很明显`black`是一个单例。

来看看我们自己创建的黑色：

```swift
let black3 = UIColor(red: 0, green: 0, blue: 0, alpha: 1)
let black4 = UIColor(red: 0, green: 0, blue: 0, alpha: 1)
print(black3 === black4)  // false
```

都是黑色，但是他们是不同的对象。

我们再看看`UIFont`：

```swift
let font1 = UIFont.systemFont(ofSize: 15)
let font2 = UIFont.systemFont(ofSize: 15)
print(font1 === font2)  // true
```

结果也是 `true`，说明`UIFont`也是使用了享元模式。

#### Demo

在开发中，难免会遇到自定义字体的需求，这里以自定义字体为例，假设在项目中要用到**SFUIText**这个系列的字体：

```swift
extension UIFont {
    
    enum SFUITextFontStyle: String {
        case medium = "SFUIText-Medium"
        case regular = "SFUIText-Regular"
        case heavy = "SFUIText-Heavy"
        case bold = "SFUIText-Bold"
        case semibold = "SFUIText-Semibold"
    }
    
    static var SFUITextFonts: [String: UIFont] = [:]
    
    static func SFUITextFont(style: SFUITextFontStyle,
                             size: CGFloat) -> UIFont? {
        
        let key = "\(style.rawValue)\(size)"
        if let font = SFUITextFonts[key] {
            return font
        }
        
        let font = UIFont(name: style.rawValue, size: size)
        SFUITextFonts[key] = font
        return font
    }
}
```

首先用枚举列举了字体的粗细程度；然后定义`SFUITextFonts`来存储创建过的字体；最后定义一个创建字体的方法，如果已经创建过的字体，则直接在缓存中取出。

我们来验证一下：

```swift
let font1 = UIFont.SFUITextFont(style: .heavy, size: 20)
let font2 = UIFont.SFUITextFont(style: .heavy, size: 20)
print(font1 == font2)  // true
```

结果是 `true`。


### 总结

在使用享元模式的过程中，需要考虑缓存数据是否很增长得很大。虽然我们用缓存来保存了一些对象，避免重复创建，以减少内存的使用，但是也有可能缓存会占用比较大的内存。所以我们可能要考虑设定缓存的最大占用内存，如果超过了设定值，则把以前的数据删掉，保留最近的数据。

另外还要注意的是，享元模式只能用于 Class 类型，不能用于 Struct 类型，因为 Struct 类型是值类型，
