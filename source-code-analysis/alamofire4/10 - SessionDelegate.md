`SessionDelegate`负责处理URLSession的所有回调。作者的整体思路：声明了与每个回调一一对应的closure，然后再实现session的所有回调。在回调的实现里面，1) 如果是必须要实现的回调，先判断我们是否有给对应的closure赋值，如果有，执行我们的closure；如果没有，使用默认的实现；2)不是必须要实现的回调，直接调用closure。

### 1. 与回调对应的closures

声明了与每个回调一一对应的closure，如果我们要重写默认的实现，给对应的closure赋值即可

```swift
/// 对应 `urlSession(_:didBecomeInvalidWithError:)` 回调
open var sessionDidBecomeInvalidWithError: ((URLSession, Error?) -> Void)?

/// 对应 `urlSession(_:didReceive:completionHandler:)` 回调
open var sessionDidReceiveChallenge: ((URLSession, URLAuthenticationChallenge) -> (URLSession.AuthChallengeDisposition, URLCredential?))?

/// 对应 `urlSession(_:didReceive:completionHandler:)` 回调
open var sessionDidReceiveChallengeWithCompletion: ((URLSession, URLAuthenticationChallenge, @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) -> Void)?

/// ......
```

### 2. 属性和初始化

```swift
// 请求重试器
var retrier: RequestRetrier?
// 因为SessionManager强引用了SessionDelegate，所以这里要用weak修饰，不然会造成循环引用
weak var sessionManager: SessionManager?

// 如果我们有多个网络请求都使用默认的SessionManager，
// 那么这几个请求的session对应的回调都是由同一个SessionDelegate来处理的，
// 所以需要把请求用字典的形式存储起来
private var requests: [Int: Request] = [:]
private let lock = NSLock()

/// 使用下标的形式存取requests字典，并且加了锁，保证线程安全
open subscript(task: URLSessionTask) -> Request? {
    get {
        lock.lock() ; defer { lock.unlock() }
        return requests[task.taskIdentifier]
    }
    set {
        lock.lock() ; defer { lock.unlock() }
        requests[task.taskIdentifier] = newValue
    }
}

public override init() {
    super.init()
}
```

### 3. 重写`NSObject`方法

返回回调是否有对应的closure。

```swift
open override func responds(to selector: Selector) -> Bool {
    // 代码省略
}
```

### 4. 实现对应的回调

如果不是必须要实现的回调，直接调用closure，没啥好说的，这里直接忽略。只讲解必须要实现的回调。

#### 1) `URLSessionDelegate`

```swift
// 服务器向客户端索取请求证书
open func urlSession(
    _ session: URLSession,
    didReceive challenge: URLAuthenticationChallenge,
    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void)
{
    // 先判断我们是否有提供对应的closure
    guard sessionDidReceiveChallengeWithCompletion == nil else {
        sessionDidReceiveChallengeWithCompletion?(session, challenge, completionHandler)
        return
    }

    // 初始化一个默认的配置，就像客户端没有实现这个回调
    var disposition: URLSession.AuthChallengeDisposition = .performDefaultHandling
    var credential: URLCredential?

    // 如果提供了sessionDidReceiveChallenge closure，执行它，并更新disposition和credential
    if let sessionDidReceiveChallenge = sessionDidReceiveChallenge {
        (disposition, credential) = sessionDidReceiveChallenge(session, challenge)
    } else if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust { // 需要的认证方法是ServerTrust
    
        // 获取保护区域的主机地址
        let host = challenge.protectionSpace.host

        // 判断我们是否提供了服务器信任策略
        if let serverTrustPolicy = session.serverTrustPolicyManager?.serverTrustPolicy(forHost: host),
            let serverTrust = challenge.protectionSpace.serverTrust {
            
            // 评估serverTrust对这个主机地址是否有效
            if serverTrustPolicy.evaluate(serverTrust, forHost: host) {
                // 评估通过，把disposition更新为使用证书；并更新证书
                disposition = .useCredential
                credential = URLCredential(trust: serverTrust)
            } else {
                // 评估不通过，取消认证，整个请求也会被取消
                disposition = .cancelAuthenticationChallenge
            }
        }
    }
    
    // 最后执行completionHandler
    completionHandler(disposition, credential)
}
```

#### 2) `URLSessionTaskDelegate`

```swift
// 服务器向客户端索取请求证书
open func urlSession(
    _ session: URLSession,
    task: URLSessionTask,
    didReceive challenge: URLAuthenticationChallenge,
    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void)
{
    // 先判断我们是否有提供对应的closure
    guard taskDidReceiveChallengeWithCompletion == nil else {
        taskDidReceiveChallengeWithCompletion?(session, task, challenge, completionHandler)
        return
    }

    // 判断是否有提供taskDidReceiveChallenge closure
    if let taskDidReceiveChallenge = taskDidReceiveChallenge {
        // 有提供，执行completionHandler
        let result = taskDidReceiveChallenge(session, task, challenge)
        completionHandler(result.0, result.1)
    } else if let delegate = self[task]?.delegate { // 没有提供closure，判断是否有delegate
        // 有delegate，执行delegate的实现
        delegate.urlSession(
            session,
            task: task,
            didReceive: challenge,
            completionHandler: completionHandler
        )
    } else {
        // 最后，执行对应的session回调
        urlSession(session, didReceive: challenge, completionHandler: completionHandler)
    }
}

// metrics收集好，更新metrics（不支持watchOS）
@available(iOS 10.0, macOS 10.12, tvOS 10.0, *)
@objc(URLSession:task:didFinishCollectingMetrics:)
open func urlSession(_ session: URLSession, task: URLSessionTask, didFinishCollecting metrics: URLSessionTaskMetrics) {
    self[task]?.delegate.metrics = metrics
}

// 数据传输任务完成
open func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
    /// 不需要重试的时候调用
    let completeTask: (URLSession, URLSessionTask, Error?) -> Void = { [weak self] session, task, error in
        // 在closure里面使用了[weak self]，先判断self是否还在
        guard let strongSelf = self else { return }

        // 执行taskDidComplete
        strongSelf.taskDidComplete?(session, task, error)

        // 执行代理的didCompleteWithError
        strongSelf[task]?.delegate.urlSession(session, task: task, didCompleteWithError: error)

        // 发出任务完成通知
        NotificationCenter.default.post(
            name: Notification.Name.Task.DidComplete,
            object: strongSelf,
            userInfo: [Notification.Key.Task: task]
        )

        // 对应的请求完成，设置为nil
        strongSelf[task] = nil
    }

    guard let request = self[task], let sessionManager = sessionManager else {
        completeTask(session, task, error)
        return
    }

    // 在检查错误之前，执行所以验证
    request.validations.forEach { $0() }

    var error: Error? = error

    if request.delegate.error != nil {
        error = request.delegate.error
    }

    if let retrier = retrier, let error = error { // 如果有error，并且有重试器
        // 询问重试器是否要重试
        retrier.should(sessionManager, retry: request, with: error) { [weak self] shouldRetry, timeDelay in
            // 判断是否要重试，不需要则执行执行completeTask
            guard shouldRetry else { completeTask(session, task, error) ; return }

            DispatchQueue.utility.after(timeDelay) { [weak self] in
                guard let strongSelf = self else { return }
                
                // 使用sessionManager重试请求，并返回重试成功与否
                let retrySucceeded = strongSelf.sessionManager?.retry(request) ?? false

                if retrySucceeded, let task = request.task { // 重试请求成功
                    strongSelf[task] = request
                    return
                } else {
                    // 重试请求失败，执行completeTask
                    completeTask(session, task, error)
                }
            }
        }
    } else {
        // 没有错误和重试器，执行completeTask
        completeTask(session, task, error)
    }
}
```

#### 3) `URLSessionDataDelegate` & `URLSessionDownloadDelegate` & `URLSessionStreamDelegate`

这是三个代理协议对应的实现都比较简单，要么直接执行对应的closure；要么先判断是否有对应的closure，如果就执行closure，如果没有就执行对应代理的实现。
