Kickstarter-iOS 把 MVVM 模式贯彻地非常彻底。MVVM 的全称是 Model-View-ViewModel，所以我们可能会觉得要有 View 存在的地方，才可以用 ViewModel。但是 Kickstarter-iOS 在 AppDelegate 中也使用了 ViewModel，把很多在 AppDelegate 处理的逻辑剥离到 AppDelegateViewModelType 中。

这篇文章主要讲一下 Kickstarter-iOS 在使用 MVVM 架构时用到的一些很好的技巧。

## 所需知识

- **什么是 MVVM 架构**： 我以前写过一篇文章简单讲解了 MVVM，不了解的可以去看看。[【设计模式】09 - MVVM模式 (Model-View-ViewModel Pattern)](https://www.jianshu.com/p/1e22cf049cb3)
- **[ReactiveSwift](https://github.com/ReactiveCocoa/ReactiveSwift)**：一个响应式编程的库，与 RxSwift 类似，这两个库非常适用于 MVVM 架构。至于要选择哪一种，可以先去了解下他们的差别，然后再决定，[How does ReactiveSwift relate to RxSwift?](https://github.com/ReactiveCocoa/ReactiveSwift/blob/master/Documentation/RxComparison.md)。

## 默认调用 bindViewModel()

在 MVVM 架构中，一般来说 ViewModel 是被 `UIView` 和 `UIViewController` 持有，而持有 ViewModel 的对象就需要绑定到 ViewModel，这样就能响应 ViewModel 中数据的变化，从而更新 UI。一般我们都会在持有 ViewModel 的对象中定义一个方法 `bindViewModel()`，并且在这个方法里面做绑定。

Kickstarter 分别在  `UIView` 和 `UIViewController`  做了一些处理，让程序在启动的时候就默认在各自内部的方法调用了 `bindViewModel()`，这样可以避免在很多的 View 和 ViewController 中写重复的代码。

### UIView

对于 `UIView`，Kickstarter 通过扩展重写 `awakeFromNib()`，在内部调用 `bindViewModel()`。代码如下：

```swift
extension UIView {
  open override func awakeFromNib() {
    super.awakeFromNib()
    self.bindViewModel()
  }

  @objc open func bindViewModel() {
  }
}
```

因为 Kickstarter 在整个项目中都是通过 xib 来构建 UI 的，所以 UI 在初始化时，`awakeFromNib()`会被调用，从而 `bindViewModel()` 也被调用。那么在其他继承自 `UIView` 的 view 中，只需要重写 `bindViewModel()`，就能达到绑定 ViewModel 的目的。

### UIViewController

在 `UIViewController` 中就会稍微复杂一点。Kickstarter 通过 runtime，默认在 `viewDidLoad()` 中调用  `bindViewModel()`。那么在其他继承自 `UIViewController` 的 ViewController 中，只需要重写 `bindViewModel()`，就能达到绑定 ViewModel 的目的。

`UIViewController-Preparation.swift`相关代码如下：

```swift
private func swizzle(_ vc: UIViewController.Type) {

  [
    (#selector(vc.viewDidLoad), #selector(vc.ksr_viewDidLoad)),
    (#selector(vc.viewWillAppear(_:)), #selector(vc.ksr_viewWillAppear(_:))),
    (#selector(vc.traitCollectionDidChange(_:)), #selector(vc.ksr_traitCollectionDidChange(_:))),
    ].forEach { original, swizzled in

      guard let originalMethod = class_getInstanceMethod(vc, original),
        let swizzledMethod = class_getInstanceMethod(vc, swizzled) else { return }

      let didAddViewDidLoadMethod = class_addMethod(vc,
                                                    original,
                                                    method_getImplementation(swizzledMethod),
                                                    method_getTypeEncoding(swizzledMethod))

      if didAddViewDidLoadMethod {
        class_replaceMethod(vc,
                            swizzled,
                            method_getImplementation(originalMethod),
                            method_getTypeEncoding(originalMethod))
      } else {
        method_exchangeImplementations(originalMethod, swizzledMethod)
      }
  }
}

private var hasSwizzled = false

extension UIViewController {
  final public class func doBadSwizzleStuff() {
    guard !hasSwizzled else { return }

    hasSwizzled = true
    swizzle(self)
  }

  @objc internal func ksr_viewDidLoad() {
    self.ksr_viewDidLoad()
    self.bindViewModel()
  }

  /**
   The entry point to bind all view model outputs. Called just before `viewDidLoad`.
   */
  @objc open func bindViewModel() {
  }
}
```

然后在 `AppDelegate.swift`中的 `didFinishLaunchingWithOptions`调用 `doBadSwizzleStuff()`，代码如下：

```swift
func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

    UIViewController.doBadSwizzleStuff()
}
```

通过这两个处理，就能避免编写大量的重复代码。

## ViewModel

我从项目中找了一个代码量比较少的 ViewModel 文件 `HelpWebViewModel.swift`，以这个文件为例。具体代码如下：

```swift
import Library
import Prelude
import ReactiveSwift
import Result

internal protocol HelpWebViewModelInputs {
  /// Call to configure with HelpType.
  func configureWith(helpType: HelpType)

  /// Call when the view loads.
  func viewDidLoad()
}

internal protocol HelpWebViewModelOutputs {
  /// Emits a request that should be loaded into the webview.
  var webViewLoadRequest: Signal<URLRequest, NoError> { get }
}

internal protocol HelpWebViewModelType {
  var inputs: HelpWebViewModelInputs { get }
  var outputs: HelpWebViewModelOutputs { get }
}

internal final class HelpWebViewModel: HelpWebViewModelType, HelpWebViewModelInputs, HelpWebViewModelOutputs {
  internal init() {
    self.webViewLoadRequest = self.helpTypeProperty.signal.skipNil()
      .takeWhen(self.viewDidLoadProperty.signal)
      .map { urlForHelpType($0, baseUrl: AppEnvironment.current.apiService.serverConfig.webBaseUrl) }
      .skipNil()
      .map { AppEnvironment.current.apiService.preparedRequest(forURL: $0) }
  }

  internal var inputs: HelpWebViewModelInputs { return self }
  internal var outputs: HelpWebViewModelOutputs { return self }

  internal let webViewLoadRequest: Signal<URLRequest, NoError>

  fileprivate let helpTypeProperty = MutableProperty<HelpType?>(nil)
  func configureWith(helpType: HelpType) {
    self.helpTypeProperty.value = helpType
  }
  fileprivate let viewDidLoadProperty = MutableProperty(())
  func viewDidLoad() {
    self.viewDidLoadProperty.value = ()
  }
}

private func urlForHelpType(_ helpType: HelpType, baseUrl: URL) -> URL? {
  switch helpType {
  case .cookie:
    return baseUrl.appendingPathComponent("cookies")
  case .contact:
    return nil
  case .helpCenter:
    return baseUrl.appendingPathComponent("help")
  case .howItWorks:
    return baseUrl.appendingPathComponent("about")
  case .privacy:
    return baseUrl.appendingPathComponent("privacy")
  case .terms:
    return baseUrl.appendingPathComponent("terms-of-use")
  case .trust:
    return baseUrl.appendingPathComponent("trust")
  }
}
```

对于 ViewModel，我想说两点：1）使用 ReactiveSwift；2）使用 inputs 和 outputs 区分数据的输入和输出。

### 使用 ReactiveSwift

在文章的开头的 **所需知识** 部分，我就提到过，响应式编程非常适合 MVVM 架构。在 ViewModel 中，我们通常会使用 ReactiveSwift 或者 RxSwift 去定义一些属性，然后在 `UIView` 和 `UIViewController`中的 `bindViewModel()` 方法里面订阅那些属性的变化，然后更新 UI。

至于在开发过程中，我们该选择哪一种呢？我个人更偏向于 ReactiveSwift。大家可以看看这篇对比文章 [How does ReactiveSwift relate to RxSwift?](https://github.com/ReactiveCocoa/ReactiveSwift/blob/master/Documentation/RxComparison.md)。

### 使用 inputs 和 outputs 区分数据的输入和输出

在 ViewModel 中，我们需要接受外部的信息输入，并且告诉外部有哪些信息发生了变化。

Kickstarter-iOS 把信息的输入和输出分别用 `HelpWebViewModelInputs` 和 `HelpWebViewModelOutputs` 分开，这样在使用 ViewModel 的时候就会非常清晰，不会把  inputs 和 outputs 混在一起。例如，我们在Xcode 中编写 `viewModel.outputs.` 时，Xcode 只会提示  `webViewLoadRequest`，而不会把属于 `inputs` 的 `viewDidLoad()`也显示给我们。

这在我们使用 ViewModel 的时候带来了极大的便利，并且让看代码的人一目了然，哪些代码处理输入，哪些代码处理输出，非常清晰。
