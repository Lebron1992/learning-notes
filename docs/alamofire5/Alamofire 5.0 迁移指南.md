# Alamofire 5.0 迁移指南

Alamofire 5.0 是 Alamofire 的最新主要版本，它是一个使用 Swift 编写的用于 iOS、tvOS、macOS 和 watchOS 的 HTTP 网络库。作为一个主要版本，遵循语义版本控制约定，5.0 引入了影响 API 的更改。

提供该指南是为了简化现有的使用了 Alamofire 4.x 的应用程序更新到最新 API 的转换，以及解释新的和更新的功能的设计和结构。由于 Alamofire 5 中更改的广泛性，本指南没有提供所有更改的完整概述。想要了解更多，用户可以去阅读 Alamofire 的 API 和使用文档。

## 升级的好处

- **重写核心**：Alamofire 的核心架构已经被重写，以遵循各种最佳实践。
  - `DispatchQueue` 的使用已更新，以遵循苹果推荐的最佳实践。这意味着，当许多请求同时运行时，Alamofire 的表现将大大提高，而不能像以前的版本那样导致队列耗尽。这将提高整体性能，降低 Alamofire 对应用程序和系统的影响。
  - 内部 APIs 已经明确了责任领域，使得实现某些特性变得更加容易，比如新的 `EventMonitor` 协议和每个请求的 SSL 失败错误等等。
  - 它从一开始就使用了各种净化，特别是线程净化，因此线程和其他运行时问题将比以前的版本少得多。
- **Decodable Responses**：`ResponseDecodable` 和 `DecodableResponseSerializer` 现在提供了内置支持，可以使用任何 `DataDecoder` 类型从网络响应中解析 `Decodable` 类型。
- **Encodable Parameters**：Alamofire 现在支持并倾向于使用 `ParameterEncoder` 协议将 `Encodable` 类型作为参数，允许对请求参数进行完全类型安全的展示。
- **URLEncodedFormEncoder**：除了支持常见的 `Encodable` 参数外，Alamofire 现在还包括 `URLEncodedFormEncoder`，一种用于 URL 表单编码的 `Encoder`。
- **`EventMonitor` 协议**：`EventMonitor` 允许访问 Alamofire 的内部事件，这使得在请求的生命周期中观察特定的操作更加容易。这使得日志记录请求非常容易。
- **异步的 `RequestAdapter`**： `RequestAdatper` 协议现在可以异步操作，从而可以向请求添加异步资源。
- **单个 `Request` `RequestInterceptor`**：现在可以将 `RequestInterceptor` 添加到单个请求中，从而首次实现细粒度控制。
- **`CachedResponseHandler` 和 `RedirectHandler` 协议**：在会话和请求的基础上，轻松访问和控制响应缓存和重定向行为。
- **`HTTPHeaders` 类型**：类型安全访问常见的 HTTP 头文件，扩展了`URLRequest`、`HTTPURLResponse` 和 `URLSessionConfiguration`，允许使用 Alamofire 的新类型设置这些类型的头文件。
- **RetryPolicy**：一种自动支持重试由于网络或其他系统错误而失败的请求的 `RequestRetrier`，具有可自定义的回退、重试限制和其他参数。

## API 变化

Alamofire 5 中的大多数 API 都已更改，因此此列表并不完整。虽然大多数顶层的请求 APIs 保持不变，但几乎所有其他类型都以某种方式发生了变化。有关最新示例，请参阅使用文档。

- `SessionManager` 已重命名为 `Session`，并且它的 APIs 也已完全改变。
- `SessionDelegate` 已重新编写，其公共 API 已完全更改。各种闭包重写已经被删除，大多数现在可以用特定的 Alamofire 特性替换。如果需要控制以前用闭包来实现的功能，可以随时打开一个特性请求。
- `TaskDelegate` 和 各种 `*TaskDelegate` 已经被删除。所有 `URLSession*Delegate` 处理现在都由 `SessionDelegate` 执行。
- `Result` 已被删除。 Alamofire 现在使用 Swift 的 `Result` 类型。
- 全局 `Alamofire` 名称空间（这从来都不是必需的）已经被删除，并替换为 `AF`，并且实际上是对 `Session.default` 的引用。
- `ServerTrustPolicyManager` 已重命名为 `ServerTrustManager`，现在需要每个已评估的请求与提供的主机之一匹配。这可以在初始化实例时通过设置 `allHostsMustBeEvaluted: false` 来禁用。
- `ServerTrustPolicy` 被改为 `ServerTrustEvaluating` 协议和几个遵循这个协议的类型。现在，每个 `ServerTrustPolicy` 案例都有对应的类型：
  - `.performDefaultEvaluation` 被 `DefaultTrustEvaluator` 代替.
  - `.performRevokedEvaluation` 被 `RevocationTrustEvaluator` 代替.
  - `.pinCertificates` 被 `PinnedCertificatesTrustEvaluator` 代替.
  - `.pinPublicKeys` 被 `PublicKeysTrustEvaluator` 代替.
  - `.disableEvalution` 被 `DisabledTrustEvalutor` 代替.
  - `.customEvaluation` 被 `CompositeTrustEvalutor` (用于组合已存在的 `ServerTrustEvaluating` 类型) 或者自定义遵循 `ServerTrustEvaluating` 协议的类型代替.
- `DataResponse` 和 `DownloadResponse` 现在对响应类型和错误类型都是双重通用的。默认情况下，所有 Alamofire APIs 都返回一个 `AF` 前缀的响应类型，并且将 `Error` 类型默认为 `AFError。`
- `Alamofire` 现在为其所有 APIs 返回 `AFError`，在 `AFError` 实例中包装任何底层系统或自定义 APIs。
- `HTTPMethod` 现在是一个结构而不是一个枚举，可以扩展以提供自定义方法。
- `AFError` 现在有几个新的案例，因此对它进行 `switch` 操作必须处理这几个新增的案例。
- Alamofire 提供的 `Notifications` 已将其 keys 重命名。您现在可以订阅：
  - 当请求及其 `URLSessionTasks` 调用 `resume()` 时，将发出 `Request.didResumeNotification` 和 `Request.didResumeTaskNotification` 通知。
  - 当请求及其 `URLSessionTasks` 调用 `suspend()` 时，将发出 `Request.didSuspendNotification` 和 `Request.didSuspendTaskNotification` 通知。
  - 当请求及其 `URLSessionTasks` 调用 `cancel()` 时，将发出 `Request.didCancelNotification` 和 `Request.didCancelTaskNotification` 通知。
  - 当请求及其 `URLSessionTasks` 调用 `finish()` 和 `URLSessionTask` 触发 `didComplete` 代理方法时，将发出 `Request.didFinishNotification` 和 `Request.didCompleteTaskNotification` 通知。
- `MulitpartFormData` 的 API 已经更改，用于创建和上传 `MultipartFormData` 的顶层 `upload` 方法已经更新，以匹配其他请求 APIs，因此不再需要处理 multipart 编码的 `Result`。
- `NetworkReachabilityManager` 已经过重构，以获得更高的可靠性和简单性。不再是设置一个更新闭包，然后启动监听器，这个闭包已经提供给了 `startListening` 方法。
- `Request` 及其各种子类已被重写，并且公共 API 已完全更改。有关当前功能的详细列表，请参阅文档。
- `Timeline` 和 Alamofire 以前的 `URLSessionTaskMetrics` 处理已经被替换为系统自带的 `URLSessionTaskMetrics`，后者现在为 Alamofire 的请求提供所有计时信息。
- `Request` 的 cURL 表示已从 `debugDescription` 中删除，该方法现在对调试输出很有用，并被放在了 `cURLDescription` 方法，该方法提供对 cURL 命令的基于完成处理程序的访问。
- `DefaultDataResponse` 和 `DefaultDownloadResponse` 已被删除。所有 `response` 方法现在都返回 `DataResponse` 或 `DownloadResponse` 类型。
- `DataResponseSerializerProtocol` 和 `DownloadResponseSerializer` 协议的要求已从属性 `serializeResponse` 更改为函数 `serializeResponse` 。此函数可以返回序列化值或引发错误，不再需要 `Result` 返回值。新的 `ResponseSerializer` 协议组合了前两个协议来简化实现。
- `RequestAdapter` 已更新为具有异步需求，允许在请求调整期间访问异步资源。

## 新特性

- Alamofire 现在通过 `af` 名称空间提供 Swift 和 Foundation 类型的扩展。
- 序列化器更新了更多的配置选项，包括允许空响应方法和代码，以及 `DataPreprocessor` 协议，以便为序列化接收的 `Data` 做准备。
- **`RetryPolicy`**：`RequestRetrier`，用于重试由于系统错误（如网络连接）而失败的请求。可配置自定义的解除缓冲设置，并默认为一系列错误，以使您的请求更可靠。
- **`CachedResponseHandler`**：新的协议，它提供了对响应是否被缓存的控制。 `ResponseCacher` 类型是作为协议的易于使用的实现而提供的。
- **`RedirectHandler`**：提供对请求重定向行为控制的新协议。 `Redirector` 类型是作为协议的易于使用的实现提供的。
- **`ParameterEncoder`**：新的协议，它支持将 `Encodable` 的值编码到 `URLRequest`。`JSONParameterEncoder` 和 `URLEncodedFormParameterEncoder` 包含在 Alamofire 中。
- **`URLEncodedFormEncoder`**：生成 `URLEncodedForm` 字符串的 `Encoder`。
