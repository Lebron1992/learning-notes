> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

状态模式属于行为模式，可以让一个对象在运行中改变他自己的行为，而行为的改变时通过改变状态来实现的。**State**指定是描述一个对象如何表现的数据，跟**Status**的意思是不同的；**Status**可以理解为你正在做一件事，这件事要多个步骤才能完成，你可以用Status来描述现在进行到哪个步骤了。

这个模式涉及到三个部分：

- **Context**：持有当前的状态
- **State Protocol**：状态协议，规定需要的方法和属性。也可以是一个基类。
- **Concrete State**：具体的实现类，遵循状态协议（或者继承于基类），描述Context如何表现自己的行为。Context持有当前的状态，但是他不知道具体的状态类型，通过多态特性来改变他的行为。

### 什么时候使用

通常我们使用这个模式来创建有多个状态系统，并且状态会改变这个系统的行为。如果你的类里面大量使用了`swtich`或者`if-else`，可以考虑是否可以使用这个模式。

### 简单demo

下面以模拟红绿灯作为例子，最终我们做成这个样子，红黄绿三种颜色的灯自动切换：

![](http://upload-images.jianshu.io/upload_images/2057254-96ad3c37ae12a888.gif?imageMogr2/auto-orient/strip)

根据上面提到的，这个模式涉及三个部分，所以在这个例子中：1）底部的整个View应该是**Context**，他持有当前的状态和所有状态，定义为`TrafficLightView`，继承自`UIView`；2）每一种颜色的灯属于一种状态，所以把**State Protocol**定义为`TrafficLightState`；3）把**Concrete State**定义为`SolidTrafficLightState`。

#### TrafficLightView

在`TrafficLightView`中，我们默认用三个light container layers来分别放置不同颜色的light layer：

```swift
final class TrafficLightView: UIView {

    // light container layers数组
    var lightContainerLayers: [CAShapeLayer] = []
    
    // 当前的状态
    private var currentState: TrafficLightState
    
    // 所有状态
    private var states: [TrafficLightState]

    // MARK: - Initializers

    init(frame: CGRect,
         lightsCount: Int = 3,
         states: [TrafficLightState]) {

        guard !states.isEmpty else { fatalError("states不能为空")}

        self.currentState = states.first!
        self.states = states
        super.init(frame: frame)

        backgroundColor = UIColor(red: 0.8, green: 0.6, blue: 0.3, alpha: 1)
        createLightContainerLayers(count: lightsCount)
        transition(to: currentState)
    }

    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    // MARK: - Public

    // 下一个状态
    var nextState: TrafficLightState {
        guard let currentIndex = states.index(where: { $0 === currentState }),
            currentIndex + 1 < states.count else {
                return states.first!
        }
        return states[currentIndex + 1]
    }

    // 切换到指定的状态
    func transition(to state: TrafficLightState) {
        removeLightLayers()
        currentState = state
        currentState.apply(to: self)
        nextState.apply(to: self, after: currentState.delay)
    }

    // MARK: - Private

    // 创建light containers
    private func createLightContainerLayers(count: Int) {
        let yTotalPadding = 0.2 * frame.height
        let containerHeight = (frame.height - yTotalPadding) / CGFloat(count)

        let yPadding = yTotalPadding / CGFloat(count + 1)
        let xPadding = (frame.width - containerHeight) / 2.0

        var containerFrame = CGRect(x: xPadding,
                                   y: yPadding,
                                   width: containerHeight,
                                   height: containerHeight)

        for _ in 0 ..< count {
            let containerShape = CAShapeLayer()
            containerShape.path = UIBezierPath(ovalIn: containerFrame).cgPath
            containerShape.fillColor = UIColor.black.cgColor
            layer.addSublayer(containerShape)
            lightContainerLayers.append(containerShape)
            containerFrame.origin.y += (containerHeight + yPadding)
        }
    }
  
    // 移除所有light containers里面的light layers
    private func removeLightLayers() {
        lightContainerLayers.forEach {
            $0.sublayers?.forEach {
                $0.removeFromSuperlayer()
            }
        }
    }
}
```

有些方法目前还没有创建，可以先忽略，下面会讲到。

### TrafficLightState

我们都知道，每个颜色的灯都会持续一段时间，所以规定需要一个`delay`属性；然后是把当前灯应用到context，所以规定`apply(to context: TrafficLightView)`方法。另外还通过扩展添加了一个方法，指定多少秒之后把当前状态应用到context，这样我们可以实现自动切换到下一个颜色的灯。

```swift
protocol TrafficLightState: class {
    var delay: TimeInterval { get }
    func apply(to context: TrafficLightView)
}

extension TrafficLightState {
    func apply(to context: TrafficLightView, after delay: TimeInterval) {
        DispatchQueue.main
            .asyncAfter(deadline: .now() + delay) { [weak self, weak context] in
                guard let strongSelf = self, let context = context else { return }
                context.transition(to: strongSelf)
        }
    }
}
```

### SolidTrafficLightState

Solid在这里的意思指的是纯色，`index`表示在红绿灯中所在的位置。在`apply(to context: TrafficLightView)`的实现中: 先找到对应的container，然后创建light layer并设置颜色，最后把light layer加到对应的container中。

另外还定义了默认的红灯、黄灯和绿灯。

```swift
final class SolidTrafficLightState: TrafficLightState {
    let index: Int
    let color: UIColor
    let delay: TimeInterval

    init(index: Int, color: UIColor, delay: TimeInterval) {
        self.index = index
        self.color = color
        self.delay = delay
    }

    func apply(to context: TrafficLightView) {
        let containerLayer = context.lightContainerLayers[index]
        let lightShape = CAShapeLayer()
        lightShape.path = containerLayer.path
        lightShape.fillColor = color.cgColor
        lightShape.strokeColor = color.cgColor
        containerLayer.addSublayer(lightShape)
    }
}

// MARK: - Lights
extension SolidTrafficLightState {
    class func redLight(index: Int = 0,
                        color: UIColor = .red,
                        delay: TimeInterval = 10) -> SolidTrafficLightState {
        return SolidTrafficLightState(index: index, color: color, delay: delay)
    }

    class func yellowLight(index: Int = 1,
                           color: UIColor = .yellow,
                           delay: TimeInterval = 3) -> SolidTrafficLightState {
        return SolidTrafficLightState(index: index, color: color, delay: delay)
    }

    class func greenLight(index: Int = 2,
                          color: UIColor = .green,
                          delay: TimeInterval = 10) -> SolidTrafficLightState {
        return SolidTrafficLightState(index: index, color: color, delay: delay)
    }
}
```

#### 使用

把上面三个类结合起来理解后，我们把`TrafficLightView`添加到一个View Controller的root view上，就能看到效果。

```swift
let frame = CGRect(x: 100, y: 100, width: 150, height: 400)
let lights: [SolidTrafficLightState] =
    [.greenLight(), .yellowLight(), .redLight()]
let trafficLight = TrafficLightView(frame: frame, states: lights)
view.addSubview(trafficLight)
```

### 总结

在作为Context的`TrafficLightView`中，我们定义的`currentState`和`states`都是跟`TrafficLightState`协议相关的，而不是跟具体的State相关联，这样能重复使用这个Context。另外如果我们还想把State应用到其他Context中，State类中的Context就不应该直接使用`TrafficLightView`，而是创建一个Context Protocol，然后State类中使用Context Protocol。
