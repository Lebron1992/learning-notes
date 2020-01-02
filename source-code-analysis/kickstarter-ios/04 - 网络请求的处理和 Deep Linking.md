## 网络请求的处理

从 `Environment` 中，可以了解到 `Service` 是处理应用中所有网络请求的。进入到  `Service`， 这里编写了所有的网络请求方法。再仔细看，你会发现很多请求是通过类似 `request(.facebookConnect(facebookAccessToken: token))` 去调用的。我们就先来看看这个 `request()` 方法的参数 `Route`。

### Route

`Route` 的部分代码如下：

```swift
internal enum Route {
  case activities(categories: [Activity.Category], count: Int?)
  case addImage(fileUrl: URL, toDraft: UpdateDraft)
  case addVideo(fileUrl: URL, toDraft: UpdateDraft)
  case backing(projectId: Int, backerId: Int)
  // ...

  internal var requestProperties:
    (method: Method, path: String, query: [String: Any], file: (name: UploadParam, url: URL)?) {

    switch self {
    case let .activities(categories, count):
      var params: [String: Any] = ["categories": categories.map { $0.rawValue }]
      params["count"] = count
      return (.GET, "/v1/activities", params, nil)

    case let .addImage(file, draft):
      return (.POST, "/v1/projects/\(draft.update.projectId)/updates/draft/images", [:], (.image, file))

    case let .addVideo(file, draft):
      return (.POST, "/v1/projects/\(draft.update.projectId)/updates/draft/video", [:], (.video, file))

    case let .backing(projectId, backerId):
      return (.GET, "/v1/projects/\(projectId)/backers/\(backerId)", [:], nil)

     // ...
    }
  }
}
```

如果你打开源文件，你会发现，`Route`枚举编写了所有用到的请求，并且定义了 `requestProperties` 属性，这样我们就可以通过类似 `.facebookConnect(facebookAccessToken: token)`去获取到想要的请求，然后通过 `requestProperties` 属性，获取到请求参数，接着做进一步的网络请求。

对于类似这种有多种可能情况的处理，用 enum 非常合适，而这也是开发过程中经常会遇到的。

既然各种请求都准备好了，下一步就要进行真正的网络请求了，这些代码就藏在 `Service+RequestHelpers.swift`。

### Service+RequestHelpers

这个文件暴露给外面的接口非常简单，如下：

```swift
extension Service {

  func fetch<A: Swift.Decodable>(query: NonEmptySet<Query>) -> SignalProducer<A, GraphError>

  func applyMutation<A: Swift.Decodable, B: GraphMutation>(mutation: B) -> SignalProducer<A, GraphError>

  func requestPagination<M: Argo.Decodable>(_ paginationUrl: String)
    -> SignalProducer<M, ErrorEnvelope> where M == M.DecodedType

  func request<M: Argo.Decodable>(_ route: Route)
    -> SignalProducer<M, ErrorEnvelope> where M == M.DecodedType

  func request<M: Argo.Decodable>(_ route: Route)
    -> SignalProducer<[M], ErrorEnvelope> where M == M.DecodedType

  func request<M: Argo.Decodable>(_ route: Route)
    -> SignalProducer<M?, ErrorEnvelope> where M == M.DecodedType
}
```

从这些方法的定义我们可以看到，全部使用了泛型，这就意味着一个方法就可以处理某一类型的请求。这六个方法就可以处理整个应用的请求，是不是觉得非常强大😁？

这也是值得我们学习的地方。所以在开发过程中，如果发现自己在重复写类似的代码，那么可以考虑使用泛型能不能解决问题。

## Deep Linking

在开发中，我们通常需要通过 Universal Link、URL Scheme 和 Push Notification 等方式跳转到应用的某一个页面。我们来看一下 Kickstarter-iOS 是怎么处理的。

打开 `Navigation.swift` ，跟网络请求一样，也是用 enum 定义了所有用户去往的目标页面。

那在 Kickstarter-iOS 中，它是怎样通过 deep linking 传入的 url 来最终得到 `Navigation` 其中的一个 case，然后跳转到目标页面呢？

首先，它用一个字典 `allRoutes: [String: (RouteParams) -> Decoded<Navigation>]` 保存了所有的 routes：其中 key 是 url 的模板；value 是一个闭包，这个闭包是根据url 携带的参数解析成 `Navigation`。

然后用一个 `match()` 方法，把传入的 url，最终解析成`Navigation` 这里面最关键的一个方法是 `parsedParams()` ，大家可以去仔细看一下怎么实现的。

```swift
extension Navigation {
  public static func match(_ url: URL) -> Navigation? {
    return allRoutes.reduce(nil) { accum, templateAndRoute in
      let (template, route) = templateAndRoute
      return accum ?? parsedParams(url: url, fromTemplate: template).flatMap(route)?.value
    }
  }
}
```