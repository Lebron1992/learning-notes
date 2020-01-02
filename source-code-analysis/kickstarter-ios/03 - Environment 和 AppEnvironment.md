有经验的 iOS 开发者应该都知道，在开发过程中我们需要设计一些对象来存储应用的全局状态，例如当前的登录用户等等。而在 Kickstarter-iOS 中，`Environment` 和 `AppEnvironment` 就是干这事的。这篇文章我们来研究下这两个 struct 。

## Environment

打开这个文件，从注释可以看到，`Environment` 是应用所需要的全局变量和单例的集合。仔细分析里面属性的定义，我们可以发现很多都是属于 protocol 类型的，例如：

```swift
public let apiService: ServiceType
public let cookieStorage: HTTPCookieStorageProtocol
public let device: UIDeviceType
public let ubiquitousStore: KeyValueStoreType
public let userDefaults: KeyValueStoreType
// ...
```

这么做的好处是当有需要的时候，可以随时替换另外一个遵循对应 protocol的对象。这也就是我们所说的面向协议编程。

大家还可以结合自己的项目，去看看其他的属性，看还有什么值得学习的地方。

## AppEnvironment
刚开始看这个项目，看到有 `Environment` 和 `AppEnvironment`，可能会觉得有点困惑，为什么有了 `Environment`，还要搞一个`AppEnvironment`？下面我们来仔细看看。

先看一下 `AppEnvironment` 里面的方法：

```swift
public struct AppEnvironment : AppEnvironmentType {

    internal static let environmentStorageKey: String

    internal static let oauthTokenStorageKey: String

    public static func login(_ envelope: AccessTokenEnvelope)

    public static func updateCurrentUser(_ user: User)

    public static func updateServerConfig(_ config: ServerConfigType)

    public static func updateConfig(_ config: Config)

    public static func updateLanguage(_ language: Language)

    public static func logout()

    public static var current: Environment! { get }

    public static func pushEnvironment(_ env: Environment)

    public static func popEnvironment() -> Environment?

    public static func replaceCurrentEnvironment(_ env: Environment)

	  // 参数太长，省略了
    public static func pushEnvironment(...)

	  // 参数太长，省略了
    public static func replaceCurrentEnvironment(...)

    public static func fromStorage(ubiquitousStore: KeyValueStoreType, userDefaults: KeyValueStoreType) -> Environment

    internal static func saveEnvironment(environment env: Environment = AppEnvironment.current, ubiquitousStore: KeyValueStoreType, userDefaults: KeyValueStoreType)
}
```

从上面的方法我们可以总结出，`AppEnvironment`是用来管理 `Environment`。如果我们不新建一个 `AppEnvironment`，那么这些管理代码就会放到 `Environment`，这会造成在一个 Model 上进行业务逻辑的处理，而这明显是不合理的。

如果你在项目中全局搜索 `pushEnvironment` 和 `popEnvironment`，你会发现，这两个方法都是在测试文件中被调用，说明这两个方法是为测试而生的。

另外 `AppEnvironment` 还提供了 `replaceCurrentEnvironment()` 方法，携带了所有对应  `Environment` 的参数，这可以让我们很容易替换当前 Environment 的某个全局变量。例如在 `AppDelegate.swift` 我们可以看到：

```swift
#if DEBUG
      if KsApi.Secrets.isOSS {
        AppEnvironment.replaceCurrentEnvironment(apiService: MockService())
      }
#endif
```

把 `KsApi.Secrets.isOSS` 设置为 `true` 之后，我们就可以使用 `MockService()`，实在是非常方便。
