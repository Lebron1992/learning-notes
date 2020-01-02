`SessionManager`的作用是用于创建各种请求。

### 1. `MultipartFormDataEncodingResult`辅助类型

`MultipartFormDataEncodingResult`用于表示编码多表单的结果，是一个枚举，并关联了一些相关信息。

```swift
public enum MultipartFormDataEncodingResult {
    case success(request: UploadRequest, streamingFromDisk: Bool, streamFileURL: URL?)
    case failure(Error)
}
```

### 2. 属性

```swift
// 默认的SessionManager
open static let `default`: SessionManager = {
    let configuration = URLSessionConfiguration.default
    configuration.httpAdditionalHeaders = SessionManager.defaultHTTPHeaders
    
    return SessionManager(configuration: configuration)
}()

// 默认请求头
open static let defaultHTTPHeaders: HTTPHeaders = {
}()

// 多表单编码时使用的内存临界值, `10_000_000`下划线使得读者更容易读
open static let multipartFormDataEncodingMemoryThreshold: UInt64 = 10_000_000

// 底层的URLSession
open let session: URLSession

// 底层URLSession的代理，session的所有代理都由它来处理
open let delegate: SessionDelegate

// 是否马上开启请求，默认是true
open var startRequestsImmediately: Bool = true

// 请求适配器
open var adapter: RequestAdapter?

// 请求重试器，由代理提供（如果想要请求失败的时候重试，我们需要定义一个请求重试器）
open var retrier: RequestRetrier? {
    get { return delegate.retrier }
    set { delegate.retrier = newValue }
}

// 默认是nil。如果设置了这个handler，SessionDelegate的 `sessionDidFinishEventsForBackgroundURLSession`会自动执行这个handler；
// 如果想要在这个handler执行前去处理自己的event，
// 我们要重写 `sessionDidFinishEventsForBackgroundURLSession`，然后手动调用这个handler
open var backgroundCompletionHandler: (() -> Void)?

// 执行队列
let queue = DispatchQueue(label: "org.alamofire.session-manager." + UUID().uuidString)
```

### 3. 初始化

```swift
// 传入`configuration`和`delegate`，创建SessionManager
public init(
    configuration: URLSessionConfiguration = URLSessionConfiguration.default,
    delegate: SessionDelegate = SessionDelegate(),
    serverTrustPolicyManager: ServerTrustPolicyManager? = nil)
{
    self.delegate = delegate
    self.session = URLSession(configuration: configuration, delegate: delegate, delegateQueue: nil)

    commonInit(serverTrustPolicyManager: serverTrustPolicyManager)
}

// 传入`session`和`delegate`，创建SessionManager（注意：session的delegate和传入的delegate必须是同一个）
public init?(
    session: URLSession,
    delegate: SessionDelegate,
    serverTrustPolicyManager: ServerTrustPolicyManager? = nil)
{
    guard delegate === session.delegate else { return nil }

    self.delegate = delegate
    self.session = session

    commonInit(serverTrustPolicyManager: serverTrustPolicyManager)
}

// 上面两个初始化器相同的一些初始化
private func commonInit(serverTrustPolicyManager: ServerTrustPolicyManager?) {
    session.serverTrustPolicyManager = serverTrustPolicyManager

    delegate.sessionManager = self

	// 把`backgroundCompletionHandler`传给`delegate.sessionDidFinishEventsForBackgroundURLSession`
	// `[weak self] session`这里的`session`在closure里面没有用到，为了代码简洁，其实应该用`_`代替的
    delegate.sessionDidFinishEventsForBackgroundURLSession = { [weak self] session in
        guard let strongSelf = self else { return }
        DispatchQueue.main.async { strongSelf.backgroundCompletionHandler?() }
    }
}

// 被回收了之后，使session失效
deinit {
    session.invalidateAndCancel()
}
```

### 4. 数据请求

```swift
// 提供`URLConvertible`，创建数据请求
@discardableResult
open func request(
    _ url: URLConvertible,
    method: HTTPMethod = .get,
    parameters: Parameters? = nil,
    encoding: ParameterEncoding = URLEncoding.default,
    headers: HTTPHeaders? = nil)
    -> DataRequest
{
    var originalRequest: URLRequest?

    do {
        originalRequest = try URLRequest(url: url, method: method, headers: headers)
        // 使用指定的`encoding`，把参数编码到请求上
        let encodedURLRequest = try encoding.encode(originalRequest!, with: parameters)
        return request(encodedURLRequest)
    } catch {
        return request(originalRequest, failedWith: error)
    }
}

// 提供`URLRequestConvertible`，创建数据请求
@discardableResult
open func request(_ urlRequest: URLRequestConvertible) -> DataRequest {
    var originalRequest: URLRequest?

    do {
        originalRequest = try urlRequest.asURLRequest()
        let originalTask = DataRequest.Requestable(urlRequest: originalRequest!)
		
		// 这里其实是用传进来的urlRequest，创建了一个dataTask
        let task = try originalTask.task(session: session, adapter: adapter, queue: queue)
        // 创建DataRequest
        let request = DataRequest(session: session, requestTask: .data(originalTask, task))
		
		// 因为delegate可能要处理多个请求，作者使用Swift的下标特性把请求记录在delegate的requests字典
        delegate[task] = request

        if startRequestsImmediately { request.resume() }

        return request
    } catch {
        return request(originalRequest, failedWith: error)
    }
}

// 提供`URLRequest`和`error`，创建数据请求
// 解决url或者参数处理过程可能会跑出错误的情况
private func request(_ urlRequest: URLRequest?, failedWith error: Error) -> DataRequest {
	// 首先声明一个关联值为空的data类型的RequestTask
    var requestTask: Request.RequestTask = .data(nil, nil)

	// 如果urlRequest不为空，创建一个data类型的RequestTask
    if let urlRequest = urlRequest {
        let originalTask = DataRequest.Requestable(urlRequest: urlRequest)
        requestTask = .data(originalTask, nil)
    }

    let underlyingError = error.underlyingAdaptError ?? error
    let request = DataRequest(session: session, requestTask: requestTask, error: underlyingError)

	// 如果自定了以重试器，并且error是适配过程中出现的error，那么允许重试
    if let retrier = retrier, error is AdaptError {
        allowRetrier(retrier, toRetry: request, with: underlyingError)
    } else {
        if startRequestsImmediately { request.resume() }
    }

    return request
}
```

关于Swift的下标特性，可以访问[【Swift 3.1】12 - 下标 (Subscripts)](http://www.jianshu.com/p/5503cb6e7c00)。

### 5. 下载请求 & 上传请求 & Stream请求

这三个请求与数据请求的代码大同小异，大家可以自己深入看看。如果有不懂的，欢迎留言，我看到了会尽力解答。

### 6. 重试请求

```swift
// 重试请求是否成功
func retry(_ request: Request) -> Bool {
	// 如果请求没有任务，则重试失败
    guard let originalTask = request.originalTask else { return false }

    do {
        let task = try originalTask.task(session: session, adapter: adapter, queue: queue)

        request.delegate.task = task // resets all task delegate data

        request.retryCount += 1
        request.startTime = CFAbsoluteTimeGetCurrent()
        request.endTime = nil

        task.resume()

        return true
    } catch {
        request.delegate.error = error.underlyingAdaptError ?? error
        return false
    }
}

// 实现重试
private func allowRetrier(_ retrier: RequestRetrier, toRetry request: Request, with error: Error) {
	// 需要重试的请求可能很多，使用异步队列
    DispatchQueue.utility.async { [weak self] in
        guard let strongSelf = self else { return }

        retrier.should(strongSelf, retry: request, with: error) { shouldRetry, timeDelay in
            guard let strongSelf = self else { return }

            guard shouldRetry else {
                if strongSelf.startRequestsImmediately { request.resume() }
                return
            }

            DispatchQueue.utility.after(timeDelay) {
                guard let strongSelf = self else { return }
				
				// 是否重试成功
                let retrySucceeded = strongSelf.retry(request)
				
				// 如果重试成功，更新delegate的requests字典
                if retrySucceeded, let task = request.task {
                    strongSelf.delegate[task] = request
                } else {
                    if strongSelf.startRequestsImmediately { request.resume() }
                }
            }
        }
    }
}
```
