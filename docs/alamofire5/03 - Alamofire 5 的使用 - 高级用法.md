- [Alamofire 5 的使用 - 高级用法](#alamofire-5-的使用---高级用法)
  - [`Session`](#session)
    - [创建自定义的 `Session` 实例](#创建自定义的-session-实例)
      - [使用 `URLSessionConfiguration` 创建 `Session`](#使用-urlsessionconfiguration-创建-session)
    - [`SessionDelegate`](#sessiondelegate)
    - [`startRequestsImmediately`](#startrequestsimmediately)
    - [`Session` 的 `DispatchQueue`](#session-的-dispatchqueue)
    - [添加 `RequestInterceptor`](#添加-requestinterceptor)
    - [添加 `ServerTrustManager`](#添加-servertrustmanager)
    - [添加 `RedirectHandler`](#添加-redirecthandler)
    - [添加 `CachedResponseHandler`](#添加-cachedresponsehandler)
    - [添加 `EventMonitor`](#添加-eventmonitor)
    - [从 `URLSession` 创建实例](#从-urlsession-创建实例)
  - [请求](#请求)
    - [请求管道](#请求管道)
    - [`Request`](#request)
      - [状态](#状态)
      - [进度](#进度)
      - [处理回调](#处理回调)
      - [自定义缓存](#自定义缓存)
      - [Credentials](#credentials)
      - [`Request` 的 `URLRequest`](#request-的-urlrequest)
      - [`URLSessionTask`](#urlsessiontask)
      - [响应](#响应)
      - [`URLSessionTaskMetrics`](#urlsessiontaskmetrics)
    - [`DataRequest`](#datarequest)
      - [其他状态](#其他状态)
      - [验证](#验证)
    - [`UploadRequest`](#uploadrequest)
      - [其他状态](#其他状态-1)
    - [`DownloadRequest`](#downloadrequest)
      - [其他状态](#其他状态-2)
      - [取消](#取消)
      - [验证](#验证-1)
  - [使用 `RequestInterceptor` 调整和重试请求](#使用-requestinterceptor-调整和重试请求)
    - [`RequestAdapter`](#requestadapter)
    - [`RequestRetrier`](#requestretrier)
  - [安全](#安全)
    - [使用 `ServerTrustManager` 和 `ServerTrustEvaluating` 评估服务器信任](#使用-servertrustmanager-和-servertrustevaluating-评估服务器信任)
      - [`ServerTrustEvaluting`](#servertrustevaluting)
      - [`ServerTrustManager`](#servertrustmanager)
        - [子类化 `ServerTrustPolicyManager`](#子类化-servertrustpolicymanager)
    - [应用传输安全 (App Transport Security)](#应用传输安全-app-transport-security)
      - [在本地网络中使用自签名证书](#在本地网络中使用自签名证书)
  - [自定义缓存和重定向处理](#自定义缓存和重定向处理)
    - [`CachedResponseHandler`](#cachedresponsehandler)
    - [`RedirectHandler`](#redirecthandler)
  - [使用 `EventMonitor`](#使用-eventmonitor)
    - [Logging](#logging)
  - [创建请求](#创建请求)
    - [`URLConvertible`](#urlconvertible)
    - [`URLRequestConvertible`](#urlrequestconvertible)
    - [路由请求](#路由请求)
  - [响应处理](#响应处理)
    - [处理没有序列化的响应](#处理没有序列化的响应)
    - [`ResponseSerializer`](#responseserializer)
      - [`DataResponseSerializer`](#dataresponseserializer)
      - [`StringResponseSerializer`](#stringresponseserializer)
      - [`JSONResponseSerializer`](#jsonresponseserializer)
      - [`DecodableResponseSerializer`](#decodableresponseserializer)
    - [自定义响应 Handlers](#自定义响应-handlers)
      - [响应转换](#响应转换)
      - [创建自定义响应序列化器](#创建自定义响应序列化器)
  - [网络可达性](#网络可达性)

# Alamofire 5 的使用 - 高级用法

这篇文章介绍的是 Alamofire 框架的高级用法，如果之前没有看过基本用法的，可以先去看看 [Alamofire 5 的使用 - 基本用法](https://github.com/Lebron1992/learning-notes/blob/master/docs/alamofire5/Alamofire%205%20%E7%9A%84%E4%BD%BF%E7%94%A8%20-%20%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95.md)

Alamofire 是建立在 `URLSession` 和 URL 加载系统之上的。为了充分利用这个框架，建议您熟悉底层网络的概念和功能。

**建议阅读**：

- [URL Loading System Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html)
- [`URLSession` Class Reference](https://developer.apple.com/reference/foundation/urlsession)
- [`URLCache` Class Reference](https://developer.apple.com/reference/foundation/urlcache)
- [`URLAuthenticationChallenge` Class Reference](https://developer.apple.com/reference/foundation/urlauthenticationchallenge)

## `Session`

Alamofire 的 `Session` 在职责上大致等同于它维护的 `URLSession` 实例：它提供 API 来生成各种 `Request` 子类，这些子类封装了不同的 `URLSessionTask` 子类，以及封装应用于实例生成的所有 `Request` 的各种配置。

`Session` 提供了一个 `default` 单例实例，并且 `AF` 实际上就是 `Session.default`。因此，以下两个语句是等效的：

```swift
AF.request("https://httpbin.org/get")
```

```swift
let session = Session.default
session.request("https://httpbin.org/get")
```

### 创建自定义的 `Session` 实例

大多数应用程序将需要以各种方式自定义其 `Session` 实例的行为。实现这一点的最简单方法是使用以下便利初始化器，并将结果存储在整个应用程序中使用的单个实例中。

```swift
public convenience init(
    configuration: URLSessionConfiguration = URLSessionConfiguration.af.default,
    delegate: SessionDelegate = SessionDelegate(),
    rootQueue: DispatchQueue = DispatchQueue(label: "org.alamofire.session.rootQueue"),
    startRequestsImmediately: Bool = true,
    requestQueue: DispatchQueue? = nil,
    serializationQueue: DispatchQueue? = nil,
    interceptor: RequestInterceptor? = nil,
    serverTrustManager: ServerTrustManager? = nil,
    redirectHandler: RedirectHandler? = nil,
    cachedResponseHandler: CachedResponseHandler? = nil,
    eventMonitors: [EventMonitor] = []
)
```

此初始化器允许自定义 `Session` 的所有基本行为。

#### 使用 `URLSessionConfiguration` 创建 `Session`

要自定义底层 `URLSession` 的行为，可以提供自定义的 `URLSessionConfiguration` 实例。建议从 `URLSessionConfiguration.af.default` 实例开始，因为它添加了 Alamofire 提供的默认 `Accept-Encoding`、`Accept-Language`和`User-Agent` headers，但是可以使用任何 `URLSessionConfiguration` 。

```swift
let configuration = URLSessionConfiguration.af.default
configuration.allowsCellularAccess = false

let session = Session(configuration: configuration)
```

> `URLSessionConfiguration` 不是设置 `Authorization` 或 `Content-Type` headers 的建议位置。相反，可以使用提供的 `headers` APIs、`ParameterEncoder` 或 `RequestAdapter` 将它们添加到 `Request` 中。

> 正如苹果在其[文档](https://developer.apple.com/documentation/foundation/urlsessionconfiguration)中所述，在实例被添加到 `URLSession`（或者，在 Alamofire 的情况下，用于初始化 `Session`）之后对 `URLSessionConfiguration` 属性进行修改没有效果。

### `SessionDelegate`

`SessionDelegate` 实例封装了对各种 `URLSessionDelegate` 和相关协议回调的所有处理。`SessionDelegate` 还充当 Alamofire 生成的每个 `Request` 的 `SessionStateDelegate`，允许 `Request` 从创建它们的 `Session` 实例间接导入状态。`SessionDelegate` 可以使用特定的 `FileManager` 实例进行自定义，该实例将用于任何磁盘访问，例如访问要通过 `UploadRequest` 上传的文件或通过 `DownloadRequest` 下载的文件。

```swift
let delelgate = SessionDelegate(fileManager: .default)
```

### `startRequestsImmediately`

默认情况下，`Session` 将在添加至少一个响应 handler 后立即对 `Request` 调用 `resume()`。将 `startRequestsImmediately` 设置为 `false` 需要手动调用所有请求的 `resume()` 方法。

```swift
let session = Session(startRequestsImmediately: false)
```

### `Session` 的 `DispatchQueue`

默认情况下，`Session` 实例对所有异步工作使用单个 `DispatchQueue`。这包括 `URLSession` 的 `delegate` `OperationQueue` 的 `underlyingQueue`，用于所有 `URLRequest` 创建、所有响应序列化工作以及所有内部 `Session` 和 `Request` 状态的改变。如果性能分析显示瓶颈在于 `URLRequest` 的创建或响应序列化，则可以为 `Session` 的每个工作区域提供单独的 `DispatchQueue`。

```swift
let rootQueue = DispatchQueue(label: "com.app.session.rootQueue")
let requestQueue = DispatchQueue(label: "com.app.session.requestQueue")
let serializationQueue = DispatchQueue(label: "com.app.session.serializationQueue")

let session = Session(
    rootQueue: rootQueue,
    requestQueue: requestQueue,
    serializationQueue: serializationQueue
 )
```

提供的任何自定义 `rootQueue` 都**必须**是串行队列，但 `requestQueue` 和 `serializationQueue` 可以是串行或并行队列。通常建议使用串行队列，除非性能分析显示工作被延迟，在这种情况下，使队列并行可能有助于提高整体性能。

### 添加 `RequestInterceptor`

Alamofire 的 `RequestInterceptor` 协议（`RequestAdapter & RequestRetrier`）提供了重要而强大的请求自适应和重试功能。它可以在 `Session` 和 `Request` 层级使用。有关 `RequestInterceptor` 和 Alamofire 包含的各种实现（如 `RetryPolicy`）的更多详细信息，请参见[下文](https://github.com/Lebron1992/learning-notes/blob/master/docs/alamofire5/03%20-%20Alamofire%205%20%E7%9A%84%E4%BD%BF%E7%94%A8%20-%20%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.md#%E4%BD%BF%E7%94%A8-requestinterceptor-%E8%B0%83%E6%95%B4%E5%92%8C%E9%87%8D%E8%AF%95%E8%AF%B7%E6%B1%82)。

```swift
let policy = RetryPolicy()
let session = Session(interceptor: policy)
```

### 添加 `ServerTrustManager`

Alamofire 的 `ServerTrustManager` 类封装了域名和遵循 `ServerTrustEvaluating`协议的类型实例之间的映射，这提供了定制 `Session` 处理 TLS 安全性的能力。这包括使用证书和公钥固定以及证书吊销检查。有关更多信息，请参阅有关 `ServerTrustManager` 和 `ServerTrustEvaluating` 的部分。初始化 `ServerTrustManger` 非常简单，只需提供域名与要执行的计算类型之间的映射即可：

```swift
let manager = ServerTrustManager(evaluators: ["httpbin.org": PinnedCertificatesTrustEvaluator()])
let session = Session(serverTrustManager: manager)
```

有关评估服务器信任的详细信息，请参阅[下面](https://github.com/Lebron1992/learning-notes/blob/master/docs/alamofire5/03%20-%20Alamofire%205%20%E7%9A%84%E4%BD%BF%E7%94%A8%20-%20%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.md#%E4%BD%BF%E7%94%A8-servertrustmanager-%E5%92%8C-servertrustevaluating-%E8%AF%84%E4%BC%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%BF%A1%E4%BB%BB)的详细文档。

### 添加 `RedirectHandler`

Alamofire 的 `RedirectHandler` 协议定制了 HTTP 重定向响应的处理。它可以在 `Session` 和 `Request` 层级使用。Alamofire 包含了遵循 `RedirectHandler` 协议的 `Redirector` 类型，并提供对重定向的简单控制。有关重定向处理程序的详细信息，请参阅[下面](https://github.com/Lebron1992/learning-notes/blob/master/docs/alamofire5/03%20-%20Alamofire%205%20%E7%9A%84%E4%BD%BF%E7%94%A8%20-%20%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.md#redirecthandler)的详细文档。

```swift
let redirector = Redirector(behavior: .follow)
let session = Session(redirectHandler: redirector)
```

### 添加 `CachedResponseHandler`

Alamofire 的 `CachedResponseHandler` 协议定制了响应的缓存，可以在 `Session` 和 `Request` 层级使用。Alamofire 包含 `ResponseCacher` 类型，它遵循 `CachedResponseHandler` 协议并提供对响应缓存的简单控制。有关详细信息，请参阅[下面](https://github.com/Lebron1992/learning-notes/blob/master/docs/alamofire5/03%20-%20Alamofire%205%20%E7%9A%84%E4%BD%BF%E7%94%A8%20-%20%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.md#cachedresponsehandler)的详细文档。

```swift
let cacher = ResponseCacher(behavior: .cache)
let session = Session(cachedResponseHandler: cacher)
```

### 添加 `EventMonitor`

Alamofire 的 `EventMonitor` 协议提供了对 Alamofire 内部事件的强大洞察力。它可以用来提供日志和其他基于事件的特性。`Session` 在初始化时接受遵循 `EventMonitor` 协议的实例的数组。

```swift
let monitor = ClosureEventMonitor()
monitor.requestDidCompleteTaskWithError = { (request, task, error) in
    debugPrint(request)
}
let session = Session(eventMonitors: [monitor])
```

### 从 `URLSession` 创建实例

除了前面提到的便利初始化器之外，还可以直接从 `URLSession` 初始化 `Session`。但是，在使用这个初始化器时需要记住几个要求，因此建议使用便利初始化器。其中包括：

- Alamofire 不支持为在后台使用而配置的 `URLSession`。初始化 `Session` 时，这将导致运行时错误。
- 必须创建 `SessionDelegate` 实例并将其作为 `URLSession` 的 `delegate`，以及传递给 `Session` 的初始化器。
- 必须将自定义 `OperationQueue` 作为 `URLSession` 的 `delegateQueue`。此队列必须是串行队列，它必须具有备用 `DispatchQueue`，并且必须将该 `DispatchQueue` 作为其 `rootQueue` 传递给 `Session`。

```swift
let rootQueue = DispatchQueue(label: "org.alamofire.customQueue")
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 1
queue.underlyingQueue = rootQueue
let delegate = SessionDelegate()
let configuration = URLSessionConfiguration.af.default
let urlSession = URLSession(configuration: configuration,
                            delegate: delegate,
                            delegateQueue: queue)
let session = Session(session: urlSession, delegate: delegate, rootQueue: rootQueue)
```

## 请求

Alamofire 执行的每个请求都由特定的类、`DataRequest`、`UploadRequest` 和 `DownloadRequest` 封装。这些类中的每一个都封装了每种类型请求所特有的功能，但是 `DataRequest` 和 `DownloadRequest` 继承自一个公共的父类 `Request`（`UploadRequest` 继承自 `DataRequest`）。`Request` 实例从不直接创建，而是通过各种 `request` 方法之一从会话 `Session` 中自动生成。

### 请求管道

一旦使用 `Request` 子类的初始参数或 `URLRequestConvertible` 创建了它，它就会通过组成 Alamofire 请求管道的一系列步骤进行传递。对于成功的请求，这些请求包括：

1. 初始参数（如 HTTP 方法、headers 和参数）被封装到内部 `URLRequestConvertible` 值中。如果直接传递 `URLRequestConvertible` 值，则使用该值时将保持不变。
2. 对 `URLRequestConvertible` 值调用 `asURLRequest()`，创建第一个 `URLRequest` 值。此值将传递给 `Request` 并存储在 `requests` 中。
3. 如果有任何 `Session` 或 `Request` `RequestAdapter` 或 `RequestInterceptor`，则使用先前创建的 `URLRequest` 调用它们。然后将调整后的 `URLRequest` 传递给 `Request` 并存储在 `requests` 中。
4. `Session` 调用 `Request` 创建的 `URLSessionTask`，以基于 `URLRequest` 执行网络请求。
5. 完成 `URLSessionTask` 并收集 `URLSessionTaskMetrics` 后，`Request` 将执行其 `Validator`。
6. 请求执行已附加的任何响应 handlers，如 `responseDecodable`。

在这些步骤中的任何一个，都可以通过创建或接收的 `Error` 值来表示失败，然后将错误值传递给关联的 `Request`。例如，除了步骤 1 和 4 之外，上面的所有其他步骤都可以创建一个`Error`，然后传递给响应 handlers 或可供重试。下面是一些可以或不能在整个请求管道中失败的示例。

- 参数封装不能失败。
- 调用 `asURLRequest()` 时，任何 `URLRequestConvertible` 值都可能创建错误。这允许初始验证各种 `URLRequest` 属性或参数编码失败。
- `RequestAdapter` 在自适应过程中可能会失败，可能是由于缺少授权 token。
- `URLSessionTask` 创建不能失败。
- `URLSessionTask` 可能由于各种原因带有错误地完成，包括网络可用性和取消。这些 `Error` 值将传递回给 `Request`。
- 响应 handlers 可以产生任何错误，通常是由于无效响应或其他分析错误。

一旦将错误传递给 `Request`，`Request` 将尝试运行与 `Session` 或 `Request` 关联的任何 `RequestRetrier`。如果任何 `RequestRetrier` 选择重试该 `Request`，则将再次运行完整的管道。`RequestRetrier`也会产生 `Error`，但这些错误不会触发重试。

### `Request`

尽管 `Request` 不封装任何特定类型的请求，但它包含 Alamofire 执行的所有请求所共有的状态和功能。这包括：

#### 状态

所有 `Request` 类型都包含状态的概念，表示 `Request` 生命周期中的主要事件。

```swift
public enum State {
    case initialized
    case resumed
    case suspended
    case cancelled
    case finished
}
```

请求在创建后以 `.initialized` 状态启动。通过调用适当的生命周期方法，可以挂起、恢复和取消 `Request`。

- `resume()` 恢复或启动请求的网络流量。如果 `startRequestsImmediately` 为 `true`，则在将响应 handlers 添加到 `Request` 后自动调用此函数。
- `suspend()` 挂起或暂停请求及其网络流量。此状态下的 `Request` 可以继续，但只有 `DownloadRequest` 才能继续传输数据。其他 `Request` 将重新开始。
- `cancel()` 取消请求。一旦进入此状态，就无法恢复或挂起 `Request`。调用 `cancel()` 时，将使用 `AFError.explicitlyCancelled` 实例设置请求的 `error` 属性。如果一个 `Request` 被恢复并且在以后没有被取消，那么它将在所有响应验证器和响应序列化器运行之后到达 `.finished` 状态。但是，如果在请求达到 `.finished` 状态后将其他响应序列化器添加到该请求，则它将转换回 `.resumed` 状态并再次执行网络请求。

#### 进度

为了跟踪请求的进度，`Request` 提供了 `uploadProgress` 和 `downloadProgress` 属性以及基于闭包的 `uploadProgress` 和 `downloadProgress` 方法。与所有基于闭包的 `Request` APIs 一样，进度 APIs 可以与其他方法链接到 `Request` 之外。与其他基于闭包的 APIs 一样，它们应该在添加任何响应 handlers（如 `responseDecodable`）之前添加到请求中。

```swift
AF.request(...)
    .uploadProgress { progress in
        print(progress)
    }
    .downloadProgress { progress in
        print(progress)
    }
    .responseDecodable(of: SomeType.self) { response in
        debugPrint(response)
    }
```

重要的是，并不是所有的 `Request` 子类都能够准确地报告它们的进度，或者可能有其他依赖项来报告它们的进度。

- 对于上传进度，可以通过以下方式确定进度：
  - 通过作为上传 body 提供给 `UploadRequest` 的 `Data` 对象的长度。
  - 通过作为 `UploadRequest` 的上传 body 提供的磁盘上文件的长度。
  - 通过根据请求的 `Content-Length` header 的值（如果已手动设置）。
- 对于下载进度，只有一个要求：
  - 服务器响应必须包含`Content-Length` header。不幸的是，`URLSession` 对进度报告可能还有其他未记录的要求，这妨碍了准确的进度报告。

#### 处理回调

Alamofire 的 `RedirectHandler` 协议提供了对 `Request` 的重定向处理的控制和定制。除了每个 `Session` `RedirectHandler` 之外，每个 `Request` 都可以被赋予属于自己的 `RedirectHandler`，并且这个 handler 将重写 `Session` 提供的任何 `RedirectHandler`。

```swift
let redirector = Redirector(behavior: .follow)
AF.request(...)
    .redirect(using: redirector)
    .responseDecodable(of: SomeType.self) { response in
        debugPrint(response)
    }
```

> 注意：一个 `Request` 只能设置一个 `RedirectHandler`。尝试设置多个将导致运行时异常。

#### 自定义缓存

Alamofire 的 `CachedResponseHandler` 协议提供了对响应缓存的控制和定制。除了每个 `Session` 的 `CachedResponseHandlers` 之外，每个 `Request` 都可以被赋予属于自己的 `CachedResponseHandler`，并且这个 handler 将重写 `Session` 提供的任何 `CachedResponseHandler`。

```swift
let cacher = Cacher(behavior: .cache)
AF.request(...)
    .cacheResponse(using: cacher)
    .responseDecodable(of: SomeType.self) { response in
        debugPrint(response)
    }
```

> 注意：一个 `Request` 只能设置一个 `CachedResponseHandler`。尝试设置多个将导致运行时异常。

#### Credentials

为了利用 `URLSession` 提供的自动凭证处理，Alamofire 提供了每个 `Request` API，允许向请求自动添加 `URLCredential` 实例。这包括使用用户名和密码进行 HTTP 身份验证的便利 API，以及任何 `URLCredential` 实例。

添加凭据以自动答复任何 HTTP 身份验证质询很简单：

```swift
AF.request(...)
    .authenticate(username: "user@example.domain", password: "password")
    .responseDecodable(of: SomeType.self) { response in
        debugPrint(response)
    }
```

> 注意：此机制仅支持 HTTP 身份验证提示。如果一个请求需要一个用于所有请求的 `Authentication` header，那么应该直接提供它，或者作为请求的一部分，或者通过一个 `RequestInterceptor`。

此外，添加 `URLCredential` 也同样简单：

```swift
let credential = URLCredential(...)
AF.request(...)
    .authenticate(using: credential)
    .responseDecodable(of: SomeType.self) { response in
        debugPrint(response)
    }
```

#### `Request` 的 `URLRequest`

由 `Request` 发出的每个网络请求最终封装在由传递给 `Session` 请求方法之一的各种参数创建的 `URLRequest` 值中。`Request` 将在其 `requests` 数组属性中保留这些 `URLRequest` 的副本。这些值既包括从传递的参数创建的初始 `URLRequest`，也包括由 `RequestInterceptors` 创建的任何 `URLRequest`。但是，该数组不包括代表 `Request` 发出的 `URLSessionTask` 执行的 `URLRequest`。要检查这些值，`tasks` 属性允许访问 `Request` 执行的所有 `URLSessionTask`。

#### `URLSessionTask`

在许多方面，各种 `Request` 子类充当 `URLSessionTask` 的包装器，提供与特定类型任务交互的特定 API。这些任务通过 `tasks` 数组属性在 `Request` 实例上可见。这包括为 `Request` 创建的初始任务，以及作为重试过程的一部分创建的任何后续任务，每次重试一个任务。

#### 响应

请求完成后，每个 `Request` 可能都有一个可用的 `HTTPURLResponse` 值。此值仅在请求未被取消且没有发出网络请求失败时可用。此外，如果重试请求，则只有最后一个响应可用。可以从 tasks 属性中的 `URLSessionTasks` 获得中间的响应。

#### `URLSessionTaskMetrics`

Alamofire 为 `Request` 执行的每个 `URLSessionTask` 收集 `URLSessionTaskMetrics` 值。这些值存储在 `metrics` 属性，每个值对应于同一索引中的 `tasks` 中的 `URLSessionTask`。

`URLSessionTaskMetrics` 也可从 Alamofire 的各种响应类型中访问，如 `DataResponse`。例如：

```swift
AF.request(...)
    .responseDecodable(of: SomeType.self) { response in {
        print(response.metrics)
    }
```

### `DataRequest`

`DataRequest` 是 `Request` 的一个子类，它封装了 `URLSessionDataTask`，将服务器响应下载到存储在内存中的 `Data` 中。因此，必须认识到，超大下载量可能会对系统性能产生不利影响。对于这些类型的下载，建议使用 `DownloadRequest` 将数据保存到磁盘。

#### 其他状态

除了 `Request` 提供的属性之外，`DataRequest` 还有一些属性。其中包括 `data`（这是服务器响应的累积 `Data`）和 `convertible`（这是创建 `DataRequest` 时使用的 `URLRequestConvertible`），其中包含创建实例的原始参数。

#### 验证

默认情况下，`DataRequest` 不验证响应。相反，必须向其中添加对 `validate()` 的调用，以验证各种属性是否有效。

```swift
public typealias Validation = (URLRequest?, HTTPURLResponse, Data?) -> Result<Void, Error>
```

默认情况下，添加 `validate()` 确保响应状态代码在 `200..<300` 范围内，并且响应的 `Content-Type` 与请求的 `Accept` 匹配。通过传递 `Validation` 闭包可以进一步定制验证：

```swift
AF.request(...)
    .validate { request, response, data in
        ...
    }
```

### `UploadRequest`

`UploadRequest` 是 `DataRequest` 的一个子类，它封装 `URLSessionUploadTask`、将 `Data`、磁盘上的文件或 `InputStream` 上传到远程服务器。

#### 其他状态

除了 `DataRequest` 提供的属性外，`UploadRequest` 还有一些属性。其中包括一个 `FileManager` 实例，用于在上传文件时自定义对磁盘的访问，以及 `upload`，`upload` 封装了用于描述请求的 `URLRequestConvertible` 值和确定要执行的上传类型的 `Uploadable` 值。

### `DownloadRequest`

`DownloadRequest` 是 `Request` 的一个具体子类，它封装了 `URLSessionDownloadTask`，将响应数据下载到磁盘。

#### 其他状态

`DownloadRequest` 除了由 `Request` 提供的属性外，还有一些属性。其中包括取消 `DownloadRequest` 时生成的数据 `resumeData`（可用于以后继续下载）和 `fileURL`（下载完成后下载文件对应的 URL）。

#### 取消

除了支持 `Request` 提供的 `cancel()` 方法外，`DownloadRequest` 还包括 `cancel(producingResumeData shouldProduceResumeData: Bool)`，如果可能的话，可以选择在取消时设置 `resumeData` 属性，以及 `cancel(byProducingResumeData completionHandler: @escaping (_ data: Data?) -> Void)`，它将生成的恢复数据提供给传递进来的闭包。

```swift
AF.download(...)
    .cancel { resumeData in
        ...
    }
```

#### 验证

`DownloadRequest` 支持的验证版本与 `DataRequest` 和 `UploadRequest` 略有不同，因为它的数据被下载到磁盘上。

```swift
public typealias Validation = (_ request: URLRequest?, _ response: HTTPURLResponse, _ fileURL: URL?)
```

必须使用提供的 `fileURL` 访问下载的 `Data`，而不是直接访问下载的 Data。否则，`DownloadRequest` 的验证器的功能与 `DataRequest` 的相同。

## 使用 `RequestInterceptor` 调整和重试请求

Alamofire 的 `RequestInterceptor` 协议（由 `RequestAdapter` 和 `RequestRetrier` 协议组成）支持强大的每个 `Session` 和每个 `Request` 功能。其中包括身份验证系统，在该系统中，向每个 `Request` 添加一个常用的 headers，并在授权过期时重试 `Request`。此外，Alamofire 还包含一个内置的 `RetryPolicy` 类型，当由于各种常见的网络错误而导致请求失败时，可以轻松重试。

### `RequestAdapter`

Alamofire 的 `RequestAdapter` 协议允许在通过网络发出之前检查和修改 `Session` 执行的每个 `URLRequest`。适配器的一个非常常见的用途，是在特定类型身份验证后面将 `Authorization` header 添加请求。

```swift
func adapt(_ urlRequest: URLRequest, for session: Session, completion: @escaping (Result<URLRequest, Error>) -> Void)
```

它的参数包括：

- `urlRequest`：最初从用于创建请求的参数或 `URLRequestConvertible` 值创建的 `urlRequest`。
- `session`：创建调用适配器的 `Request` 的 `Session`。
- `completion`： 一个必须调用的、用来表示适配器已完成的异步 completion handler。它的异步特性使 `RequestAdapter` 能够在请求通过网络发送之前从网络或磁盘访问异步资源。提供给 `completion` 闭包的 `Result` 可以返回带有修改后的 `URLRequest` 的 `.success` 值，或者返回带有关联错误的 `.failure` 值，然后将使用该值使请求失败。例如，添加 `Authorization` header 需要修改 `URLRequest`，然后调用 `completion`。

```swift
let accessToken: String

func adapt(_ urlRequest: URLRequest, for session: Session, completion: @escaping (Result<URLRequest, Error>) -> Void) {
    var urlRequest = urlRequest
    urlRequest.headers.add(.authorization(bearer: accessToken))

    completion(.success(urlRequest))
}
```

### `RequestRetrier`

Alamofire 的 `RequestRetrier` 协议允许重试在执行时遇到错误的请求。这包括在 Alamofire 的[请求管道](https://github.com/Lebron1992/learning-notes/blob/master/docs/alamofire5/03%20-%20Alamofire%205%20%E7%9A%84%E4%BD%BF%E7%94%A8%20-%20%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.md#%E8%AF%B7%E6%B1%82%E7%AE%A1%E9%81%93)的任何阶段产生的错误。

`RequestRetrier` 协议只有一个方法：

```swift
func retry(_ request: Request, for session: Session, dueTo error: Error, completion: @escaping (RetryResult) -> Void)
```

它的参数包括：

- `request`: 遇到错误的 `Request`。
- `session`: 管理 `Request` 的 `Session`。
- `error`: 触发重试的 `Error`，通常是一个 `AFError`。
- `completion`: 必须调用的异步 completion handler，以表示 `Request` 是否需要重试。调用的时候必须传入 `RetryResult`。

`RetryResult` 类型表示在 `RequestRetrier` 中实现的任何逻辑的结果。定义为：

```swift
/// Outcome of determination whether retry is necessary.
public enum RetryResult {
    /// Retry should be attempted immediately.
    case retry
    /// Retry should be attempted after the associated `TimeInterval`.
    case retryWithDelay(TimeInterval)
    /// Do not retry.
    case doNotRetry
    /// Do not retry due to the associated `Error`.
    case doNotRetryWithError(Error)
}
```

例如，如果请求是等幂的，Alamofire 的 `RetryPolicy` 类型将自动重试由于某种网络错误而失败的请求。

```swift
open func retry(_ request: Request, for session: Session, dueTo error: Error, completion: @escaping (RetryResult) -> Void) {
    if request.retryCount < retryLimit,
       let httpMethod = request.request?.method,
       retryableHTTPMethods.contains(httpMethod),
       shouldRetry(response: request.response, error: error) {
        let timeDelay = pow(Double(exponentialBackoffBase), Double(request.retryCount)) * exponentialBackoffScale
        completion(.retryWithDelay(timeDelay))
    } else {
        completion(.doNotRetry)
    }
}
```

## 安全

在与服务器和 web 服务通信时使用安全的 HTTPS 连接是保护敏感数据的重要步骤。默认情况下，Alamofire 接收与 `URLSession` 相同的自动 TLS 证书和证书链验证。虽然这保证了证书链的有效性，但并不能防止中间人（MITM）攻击或其他潜在的漏洞。为了减轻 MITM 攻击，处理敏感客户数据或财务信息的应用程序应使用 Alamofire 的 `ServerTrustEvaluating` 协议提供的证书或公钥固定。

### 使用 `ServerTrustManager` 和 `ServerTrustEvaluating` 评估服务器信任

#### `ServerTrustEvaluting`

`ServerTrustEvaluting` 协议提供了执行任何类型服务器信任评估的方法。它只有一个方法：

```swift
func evaluate(_ trust: SecTrust, forHost host: String) throws
```

此方法提供从底层 `URLSession` 接收的 [`SecTrust`](https://developer.apple.com/documentation/security/certificate_key_and_trust_services/trust) 值和主机 `String`，并提供执行各种评估的机会。

Alamofire 包括许多不同类型的信任评估器，为评估过程提供可组合的控制：

- `DefaultTrustEvaluator`：使用默认服务器信任评估，同时允许您控制是否验证质询提供的主机。
- `RevocationTrustEvaluator`：检查接收到的证书的状态以确保它没有被吊销。这通常不会在每个请求上执行，因为它需要网络请求开销。
- `PinnedCertificatesTrustEvaluator`: 使用提供的证书验证服务器信任。如果某个固定证书与某个服务器证书匹配，则认为服务器信任有效。此评估器还可以接受自签名证书。
- `PublicKeysTrustEvaluator`: 使用提供的公钥验证服务器信任。如果某个固定公钥与某个服务器证书公钥匹配，则认为服务器信任有效。
- `CompositeTrustEvaluator`: 评估一个 `ServerTrustEvaluating` 值数组，只有在所有数组中值都成功时才成功。此类型可用于组合，例如，`RevocationTrustEvaluator` 和 `PinnedCertificatesTrustEvaluator`。
- `DisabledEvaluator`：此评估器应仅在调试方案中使用，因为它禁用所有求值，而这些求值又将始终认为任何服务器信任都是有效的。此评估器不应在生产环境中使用！

#### `ServerTrustManager`

`ServerTrustManager` 负责存储 `ServerTrustEvaluating` 值到特定主机的内部映射。这允许 Alamofire 使用不同的评估器评估每个主机。

```swift
let evaluators: [String: ServerTrustEvaluating] = [
    // 默认情况下，包含在 app bundle 的证书会自动固定。
    "cert.example.com": PinnedCertificatesTrustEvalutor(),
    // 默认情况下，包含在 app bundle 的来自证书的公钥会被自动使用。
    "keys.example.com": PublicKeysTrustEvalutor(),
]

let manager = ServerTrustManager(evaluators: serverTrustPolicies)
```

此 `ServerTrustManager` 将具有以下行为：

- `cert.example.com` 将始终在启用默认和主机验证的情况下使用证书固定，因此需要满足以下条件才能允许 TLS 握手成功：
  - 证书链*必须*有效。
  - 证书链*必须*包含一个固定证书。
  - 质询主机*必须*与证书链的叶证书中的主机匹配。
- `keys.example.com` 将始终在启用默认和主机验证的情况下使用公钥固定，因此需要满足以下条件才能允许 TLS 握手成功：
  - 证书链*必须*有效。
  - 证书链*必须*包含一个固定的公钥。
  - 质询主机*必须*与证书链中的证书中的主机匹配。
- 对其他主机的请求将产生一个错误，因为服务器信任管理器要求默认评估所有主机。

##### 子类化 `ServerTrustPolicyManager`

如果发现自己需要更灵活的服务器信任策略匹配行为（例如通配符域名），那么子类化 `ServerTrustManager`，并用自己的自定义实现重写 `serverTrustEvaluator(forHost:)` 方法。

```swift
final class CustomServerTrustPolicyManager: ServerTrustPolicyManager {
    override func serverTrustEvaluator(forHost host: String) -> ServerTrustEvaluating? {
        var policy: ServerTrustPolicy?

        // Implement your custom domain matching behavior...

        return policy
    }
}
```

### 应用传输安全 (App Transport Security)

在 iOS 9 中添加了 App Transport Security（ATS），使用带有多个 `ServerTrustEvaluating` 对象的自定义 `ServerTrustManager` 可能不会有任何效果。如果您持续看到 `CFNetwork SSLHandshake failed (-9806)` 错误，则可能遇到了此问题。苹果的 ATS 系统会覆盖整个质询系统，除非您在应用程序的 plist 中配置 ATS 设置以禁用足够多的 ATS 设置，以允许您的应用程序评估服务器信任。如果遇到此问题（自签名证书的概率很高），可以通过将 [`NSAppTransportSecurity`](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity) 设置添加到 `Info.plist` 来解决此问题。您可以使用 `nscurl` 工具的 `--ats-diagnostics` 选项对主机执行一系列测试，以查看可能需要哪些 ATS 重写。

#### 在本地网络中使用自签名证书

如果尝试连接到本地主机上运行的服务器，并且使用自签名证书，则需要将以下内容添加到 `Info.plist` 中。

```xml
<dict>
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsLocalNetworking</key>
        <true/>
    </dict>
</dict>
```

根据[苹果文档](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW35)，将 `NSAllowsLocalNetworking` 设置为 `YES` 允许加载本地资源，而不必为应用程序的其余部分禁用 ATS。

## 自定义缓存和重定向处理

`URLSession` 允许使用 `URLSessionDataDelegate` 和 `URLSessionTaskDelegate` 方法自定义缓存和重定向行为。Alamofire 将这些定制点呈现为 `CachedResponseHandler` 和 `RedirectHandler` 协议。

### `CachedResponseHandler`

`CachedResponseHandler` 协议允许控制将 HTTP 响应缓存到与发出请求的 `Session` 相关联的 `URLCache` 实例中。该协议只有一个方法：

```swift
func dataTask(_ task: URLSessionDataTask,
              willCacheResponse response: CachedURLResponse,
              completion: @escaping (CachedURLResponse?) -> Void)
```

从方法签名中可以看出，此控制仅适用于使用底层 `URLSessionDataTask` 进行网络传输的 `Request`，这些请求包括 `DataRequest` 和 `UploadRequest`（因为 `URLSessionUploadTask` 是 `URLSessionDataTask` 的一个子类）。考虑响应进行缓存的条件非常广泛，因此最好查看 `URLSessionDataDelegate` 方法 [`urlSession(_:dataTask:willCacheResponse:completionHandler:)`](https://developer.apple.com/documentation/foundation/urlsessiondatadelegate/1411612-urlsession) 的文档。一旦考虑将响应用于缓存，就可以进行各种有价值的操作：

- 通过返回 `nil` `CachedURLResponse` 来防止完全缓存响应。
- 修改 `CachedURLResponse` 的 `storagePolicy`，以更改缓存值的存放位置。
- 直接修改底层 `URLResponse`，添加或删除值。
- 修改与响应关联的 `Data`（如果有）。

Alamofire 包含遵循 `CachedResponseHandler` 协议的 `ResponseCacher` 类型，使缓存（或者不缓存）或修改响应变得容易。`ResponseCacher` 接受一个 `Behavior` 值来控制缓存行为。

```swift
public enum Behavior {
    /// Stores the cached response in the cache.
    case cache
    /// Prevents the cached response from being stored in the cache.
    case doNotCache
    /// Modifies the cached response before storing it in the cache.
    case modify((URLSessionDataTask, CachedURLResponse) -> CachedURLResponse?)
}
```

`ResponseCacher` 可以在 `Session` 和 `Request` 的基础上使用，如上所述。

### `RedirectHandler`

`RedirectHandler` 协议允许控制特定 `Request` 的重定向行为。它只有一个方法：

```swift
func task(_ task: URLSessionTask,
          willBeRedirectedTo request: URLRequest,
          for response: HTTPURLResponse,
          completion: @escaping (URLRequest?) -> Void)
```

此方法提供了修改重定向的 `URLRequest` 或传递 `nil` 以完全禁用重定向的机会。Alamofire 提供了遵循 `RedirectHandler` 协议的 `Redirector` 类型，使其易于 follow、not follow 或修改重定向请求。`Redirector` 接受一个 `Behavior` 值来控制重定向行为。

```swift
public enum Behavior {
    /// Follow the redirect as defined in the response.
    case follow
    /// Do not follow the redirect defined in the response.
    case doNotFollow
    /// Modify the redirect request defined in the response.
    case modify((URLSessionTask, URLRequest, HTTPURLResponse) -> URLRequest?)
}
```

## 使用 `EventMonitor`

`EventMonitor` 协议允许观察和检查大量内部 Alamofire 事件。这些事件包括由 Alamofire 实现的所有 `URLSessionDelegate`、`URLSessionTaskDelegate` 和 `URLSessionDownloadDelegate` 方法以及大量内部 `Request` 事件。除了这些事件（默认情况下是不起作用的空方法）之外，`EventMonitor` 协议还需要一个 `DispatchQueue`，在这个 `DispatchQueue` 上调度所有事件以保持性能。此 `DispatchQueue` 默认为 `.main`，但对于任何自定义一致类型，建议使用专用串行队列。

### Logging

也许 `EventMonitor` 协议的最大用途是实现相关事件的日志记录。一个简单的实现可能如下所示：

```swift
final class Logger: EventMonitor {
    let queue = DispatchQueue(label: ...)

    // Event called when any type of Request is resumed.
    func requestDidResume(_ request: Request) {
        print("Resuming: \(request)")
    }

    // Event called whenever a DataRequest has parsed a response.
    func request<Value>(_ request: DataRequest, didParseResponse response: DataResponse<Value, AFError>) {
        debugPrint("Finished: \(response)")
    }
}
```

此 `Logger` 类型可以按上述方法添加到 `Session` 中：

```swift
let logger = Logger()
let session = Session(eventMonitors: [logger])
```

## 创建请求

作为一个框架，Alamofire 有两个主要目标：

1. 使原型和工具的网络请求易于实现
2. 作为 APP 网络请求的通用基础

它通过使用强大的抽象、提供有用的默认值和包含常见任务的实现来实现这些目标。然而，一旦 Alamofire 的使用超出了一些请求，就有必要超越高级的、默认的实现，进入为特定应用程序定制的行为。Alamofire 提供 `URLConvertible` 和 `URLRequestConvertible` 协议来帮助进行这种定制。

### `URLConvertible`

可以使用遵循 `URLConvertible` 协议的类型来构造 URL，然后使用 URL 在内部构造 URL 请求。默认情况下，`String`、`URL` 和 `URLComponents` 遵循了 `URLConvertible` 协议，允许将它们中的任何一个作为 URL 参数传递给 `request`、`upload` 和 `download` 方法：

```swift
let urlString = "https://httpbin.org/get"
AF.request(urlString)

let url = URL(string: urlString)!
AF.request(url)

let urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: true)!
AF.request(urlComponents)
```

鼓励以有意义的方式与 web 应用程序交互的应用程序具有遵循 `URLConvertible` 的自定义类型，这是将特定于域的模型映射到服务器资源的一种方便方法。

### `URLRequestConvertible`

遵循 `URLRequestConvertible` 协议的类型可用于构造 `URLRequest`。默认情况下，`URLRequest` 遵循 `URLRequestConvertible`，允许将其直接传递到 `request`、`upload` 和 `download` 方法中。Alamofire 使用 `URLRevestExchange` 作为请求管道中流动的所有请求的基础。直接使用 `URLRequest` 是在 Alamofire 提供的 `ParamterEncoder` 之外自定义 `URLRequest` 创建的推荐方法。

```swift
let url = URL(string: "https://httpbin.org/post")!
var urlRequest = URLRequest(url: url)
urlRequest.method = .post

let parameters = ["foo": "bar"]

do {
    urlRequest.httpBody = try JSONEncoder().encode(parameters)
} catch {
    // Handle error.
}

urlRequest.headers.add(.contentType("application/json"))

AF.request(urlRequest)
```

鼓励以有意义的方式与 web 应用程序交互的应用程序具有遵循 `URLRequestConvertible` 的自定义类型，以确保所请求端点的一致性。这种方法可以用来消除服务器端的不一致性，提供类型安全的路由，以及管理其他状态。

### 路由请求

随着应用程序规模的增长，在构建网络堆栈时采用通用模式非常重要。该设计的一个重要部分是如何路由您的请求。Alamofire `URLConvertible` 和 `URLRequestConvertible` 协议以及 `Router` 设计模式都可以帮助您。

“router” 是定义“路由”或请求组件的类型。这些组件可以包括 `URLRequest` 的部分、发出请求所需的参数以及每个请求的各种 Alamofire 设置。一个简单的 router 可能看起来像这样：

```swift
enum Router: URLRequestConvertible {
    case get, post

    var baseURL: URL {
        return URL(string: "https://httpbin.org")!
    }

    var method: HTTPMethod {
        switch self {
        case .get: return .get
        case .post: return .post
        }
    }

    var path: String {
        switch self {
        case .get: return "get"
        case .post: return "post"
        }
    }

    func asURLRequest() throws -> URLRequest {
        let url = baseURL.appendingPathComponent(path)
        var request = URLRequest(url: url)
        request.method = method

        return request
    }
}

AF.request(Router.get)
```

更复杂的 router 可以包括请求的参数。使用 Alamofire 的 `ParameterEncoder` 协议和包含的编码器，任何 `Encodable` 类型都可以用作参数：

```swift
enum Router: URLRequestConvertible {
    case get([String: String]), post([String: String])

    var baseURL: URL {
        return URL(string: "https://httpbin.org")!
    }

    var method: HTTPMethod {
        switch self {
        case .get: return .get
        case .post: return .post
        }
    }

    var path: String {
        switch self {
        case .get: return "get"
        case .post: return "post"
        }
    }

    func asURLRequest() throws -> URLRequest {
        let url = baseURL.appendingPathComponent(path)
        var request = URLRequest(url: url)
        request.method = method

        switch self {
        case let .get(parameters):
            request = try URLEncodedFormParameterEncoder().encode(parameters, into: request)
        case let .post(parameters):
            request = try JSONParameterEncoder().encode(parameters, into: request)
        }

        return request
    }
}
```

Router 可以扩展到具有任意数量可配置属性的任意数量的端点，但是一旦达到了一定的复杂程度，就应该考虑将一个大的 router 分成较小的 router 作为 API 的一部分。

## 响应处理

Alamofire 通过各种 `response` 方法和 `ResponseSerializer` 协议提供响应处理。

### 处理没有序列化的响应

`DataRequest` 和 `DownloadRequest` 都提供了一些方法，这些方法允许在不调用任何 `ResponseSerializer` 的情况下进行响应处理。对于无法将大文件加载到内存中的 `DownloadRequest`，这一点最为重要。

```swift
// DataRequest
func response(queue: DispatchQueue = .main, completionHandler: @escaping (AFDataResponse<Data?>) -> Void) -> Self

// DownloadRequest
func response(queue: DispatchQueue = .main, completionHandler: @escaping (AFDownloadResponse<URL?>) -> Void) -> Self
```

与所有响应 handlers 一样，所有序列化工作（在本例中为“无”）都在内部队列上执行，并在传递给方法的 `queue` 上调用 completion handler。这意味着在默认情况下不需要将其分派回主队列。但是，如果要在 completion handler 中执行任何重要的工作，建议将自定义队列传递给响应方法，必要时在 handler 本身中将分派回主队列。

### `ResponseSerializer`

`ResponseSerializer` 协议由 `DataResponseSerializerProtocol` 和 `DownloadResponseSerializerProtocol` `协议组成。ResponseSerializer` 的组合版本如下：

```swift
public protocol ResponseSerializer: DataResponseSerializerProtocol & DownloadResponseSerializerProtocol {
    /// The type of serialized object to be created.
    associatedtype SerializedObject

    /// `DataPreprocessor` used to prepare incoming `Data` for serialization.
    var dataPreprocessor: DataPreprocessor { get }
    /// `HTTPMethod`s for which empty response bodies are considered appropriate.
    var emptyRequestMethods: Set<HTTPMethod> { get }
    /// HTTP response codes for which empty response bodies are considered appropriate.
    var emptyResponseCodes: Set<Int> { get }

    func serialize(request: URLRequest?, response: HTTPURLResponse?, data: Data?, error: Error?) throws -> SerializedObject
    func serializeDownload(request: URLRequest?,
                           response: HTTPURLResponse?,
                           fileURL: URL?,
                           error: Error?) throws -> SerializedObject
}
```

默认情况下，`serializeDownload` 方法是通过从磁盘读取下载的数据并调用 `serialize` 来实现的。因此，使用上面提到的 `DownloadRequest` 的响应 `response(queue:completionHandler:)` 方法实现对大型下载的自定义处理可能更为合适。

`ResponseSerializer` 为 `dataPreprocessor`、`emptyResponseMethods` 和 `emptyResponseCodes` 提供了各种默认实现，这些实现可以在自定义类型中进行定制，如 Alamofire 附带的各种 `ResponseSerializer`。

所有 `ResponseSerializer` 的使用都通过 `DataRequest` 和 `DownloadRequest` 上的方法进行：

```swift
// DataRequest
func response<Serializer: DataResponseSerializerProtocol>(
    queue: DispatchQueue = .main,
    responseSerializer: Serializer,
    completionHandler: @escaping (AFDataResponse<Serializer.SerializedObject>) -> Void) -> Self

// DownloadRequest
func response<Serializer: DownloadResponseSerializerProtocol>(
    queue: DispatchQueue = .main,
    responseSerializer: Serializer,
    completionHandler: @escaping (AFDownloadResponse<Serializer.SerializedObject>) -> Void) -> Self
```

Alamofire 包括几个常见的响应 handlers，包括：

- `responseData(queue:completionHandler)`：使用 `DataResponseSerializer` 验证和预处理响应 `Data`。
- `responseString(queue:encoding:completionHandler:)`：使用提供的 `String.Encoding` 将响应 `Data` 解析为 `String`。
- `responseJSON(queue:options:completionHandler)`：使用提供的 `JSONSerialization.ReadingOptions` 使用 `JSONSerialization` 解析响应 `Data`。不建议使用此方法，仅为与现有的 Alamofire 用法兼容而提供。相反，应该使用 `responseDecodable`。
- `responseDecodable(of:queue:decoder:completionHandler:)`：使用提供的 `DataDecoder` 将响应 `Data` 解析为提供的或推断的 `Decodable` 类型。默认情况下使用 `JSONDecoder`。JSON 和泛型响应解析推荐用此方法。

### `DataResponseSerializer`

在 `DataRequest` 或 `DownloadRequest` 上调用 `responseData(queue:completionHandler:)` 使用 `DataResponseSerializer` 验证 `Data` 是否已正确返回（除非 `emptyResponseMethods` 和 `emptyResponseCodes` 允许，否则不允许空响应），并将该 `Data` 传递到 `dataPreprocessor`。此响应 handler 对于自定义 `Data` 处理非常有用，但通常不是必需的。

### `StringResponseSerializer`

对 `DataRequest` 或 `DownloadRequest` 调用 `responseString(queue:encoding:completionHa` 使用 `StringResponseSerializer` 验证 `Data` 是否已正确返回（除非 `emptyResponseMethods` 和 `emptyResponseCodes` 允许，否则不允许空响应）并将该 `Data` 传递到 `dataPreprocessor`。然后，使用从 `HTTPURLResponse` 解析的 `String.Encoding` 处理 `Data`，并初始化一个 `String`。

### `JSONResponseSerializer`

在 `DataRequest` 或 `DownloadRequest` 上调用 `responseJSON(queue:options:completionHandler)` 使用 `JSONResponseSerializer` 验证 `Data` 是否已正确返回（除非 `emptyResponseMethods` 和 `emptyResponseCodes` 允许，否则不允许空响应），并将该 `Data` 传递给 `dataPreprocessor`。然后，使用提供的选项把预处理的 `Data` 传递给 `JSONSerialization.jsonObject(with:options:)`。不再推荐使用此序列化程序。相反，使用 `DecodableResponseSerializer` 提供了更好的快速体验。

### `DecodableResponseSerializer`

在 `DataRequest` 或 `DownloadRequest` 上调用 `responseDecodable(of:queue:decoder:completionHandler)` 使用 `DecodableResponseSerializer` 来验证 `Data` 是否已正确返回（除非 `emptyResponseMethods` 和 `emptyResponseCodes` 允许，否则不允许空响应），并将该 `Data` 传递给 `dataPreprocessor`。然后，预处理的 `Data` 传递给提供的 `DataDecoder`，并解析为提供的或推断的 `Decodable` 类型。

### 自定义响应 Handlers

除了包含在 Alamofire 中的灵活的 `ResponseSerializer` 之外，还有其他定制响应处理的方法。

#### 响应转换

使用现有的 `ResponseSerializer` 然后转换输出是定制响应 handler 的最简单方法之一。`DataResponse` 和 `DownloadResponse` 都有 `map`、`tryMap`、`mapError` 和 `tryMapError` 方法，这些方法可以转换响应，同时保留与响应相关联的元数据。例如，可以使用 `map` 从可解码响应中提取属性，同时还保留以前的任何解析错误。

```swift
AF.request(...).responseDecodable(of: SomeType.self) { response in
    let propertyResponse = response.map { $0.someProperty }

    debugPrint(propertyResponse)
}
```

引发错误的转换也可以与 `tryMap` 一起使用，可能用于执行验证：

```swift
AF.request(..).responseDecodable(of: SomeType.self) { response in
    let propertyResponse = response.tryMap { try $0.someProperty.validated() }

    debugPrint(propertyResponse)
}
```

#### 创建自定义响应序列化器

当 Alamofire 提供的 `ResponseSerializer` 或响应转换不够灵活，或者定制量很大时，创建 `ResponseSerializer` 是封装该逻辑的好方法。集成自定义 `ResponseSerializer` 通常有两个部分：创建遵循协议的类型和扩展相关请求类型以方便使用。例如，如果服务器返回了一个特殊编码的 `String`（可能是用逗号分隔的值），那么这种格式的 `ResponseSerializer` 可能如下所示：

```swift
struct CommaDelimitedSerializer: ResponseSerializer {
    func serialize(
        request: URLRequest?,
        response: HTTPURLResponse?,
        data: Data?,
        error: Error?
    ) throws -> [String] {
        // Call the existing StringResponseSerializer to get many behaviors automatically.
        let string = try StringResponseSerializer().serialize(
            request: request,
            response: response,
            data: data,
            error: error
        )

        return Array(string.split(separator: ","))
    }
}
```

请注意，`serialize` 方法的返回类型要满足 `SerializedObject` `associatedtype` 要求。在更复杂的序列化器中，此返回类型本身可以是泛型的，从而允许泛型类型的序列化，如 `DecodableResponseSerializer` 所示。

为了使 `CommaDelimitedSerializer` 更有用，可以添加其他行为，比如允许通过将空 HTTP 方法和响应代码传递给底层的 `StringResponseSerializer` 来自定义它们。

## 网络可达性

`NetworkReachabilityManager` 监听移动网络和 `WiFi` 网络接口的主机和地址的可达性变化。

```swift
let manager = NetworkReachabilityManager(host: "www.apple.com")

manager?.startListening { status in
    print("Network Status Changed: \(status)")
}
```

> 一定要记住保存 `manager`，否则不会报告状态更改。另外，不要将 scheme 包含在 `host` 字符串中，否则可达性将无法正常工作。

当使用网络可达性来确定下一步要做什么时，需要记住一些重要的事情。

- **不要**使用可达性来确定是否应发送网络请求。
  - 你应该**总是**把请求发出去。
- 恢复可访问性后，使用事件重试失败的网络请求。
  - 尽管网络请求可能仍然失败，但现在是重试请求的好时机。
- 网络可达性状态可用于确定网络请求失败的原因。
  - 如果网络请求失败，则更有用的方法是告诉用户网络请求由于离线而失败，而不是更技术性的错误，例如“请求超时”

或者，使用 `RequestRetrier`（如内置的 `RetryPolicy`）可能会更简单、更可靠，而不是使用可访问性更新来重试因网络故障而失败的请求。默认情况下，`RetryPolicy` 将在各种错误条件下重试等幂请求，包括离线的网络连接。
