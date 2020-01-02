这个文件定义了几个delegate类型。`TaskDelegate`是`DataTaskDelegate`、`DownloadTaskDelegate`的父类；其中`DataTaskDelegate`是`UploadTaskDelegate`的父类。

### 1. `TaskDelegate`

跟`SessionDelegate`类似，`TaskDelegate`用于处理task的所有回调和执行队列的operations。

#### 1) 属性和初始化方法

```swift
// 任务完成后，执行这个队列里的所有operations
open let queue: OperationQueue

// 服务器返回的数据，这里返回nil，然后让子类去重写
public var data: Data? { return nil }

// 整个任务过程中出现的错误
public var error: Error?

// 作者为了在get和set的时候加锁，定义了一个私有的_task
var task: URLSessionTask? {
    set {
        taskLock.lock(); defer { taskLock.unlock() }
        _task = newValue
    }
    get {
        taskLock.lock(); defer { taskLock.unlock() }
        return _task
    }
}

var initialResponseTime: CFAbsoluteTime? // 服务器最开始响应的时间
var credential: URLCredential? // URL证书
var metrics: AnyObject? // URLSessionTaskMetrics

private var _task: URLSessionTask? {
    didSet { reset() }
}

private let taskLock = NSLock()

init(task: URLSessionTask?) {
    _task = task

    // 这里利用了一个closure来初始化queue
    // 我们可以学习这种模式，可以让代码看起来更清晰
    self.queue = {
        let operationQueue = OperationQueue()

        operationQueue.maxConcurrentOperationCount = 1
        // 刚初始化，把这个属性设置为true暂时不执行队列里面的operation
        operationQueue.isSuspended = true
        operationQueue.qualityOfService = .utility

        return operationQueue
    }()
}

func reset() {
    error = nil
    initialResponseTime = nil
}
```

#### 2) 实现`URLSessionTaskDelegate`

```swift
// 定义了与回调对应的closure
var taskWillPerformHTTPRedirection: ((URLSession, URLSessionTask, HTTPURLResponse, URLRequest) -> URLRequest?)?
var taskDidReceiveChallenge: ((URLSession, URLSessionTask, URLAuthenticationChallenge) -> (URLSession.AuthChallengeDisposition, URLCredential?))?
var taskNeedNewBodyStream: ((URLSession, URLSessionTask) -> InputStream?)?
var taskDidCompleteWithError: ((URLSession, URLSessionTask, Error?) -> Void)?

// 告诉代理服务器请求HTTP重定向
// @objc的意思是告诉在OC使用的时候如何显示这个方法
@objc(URLSession:task:willPerformHTTPRedirection:newRequest:completionHandler:)
func urlSession(
    _ session: URLSession,
    task: URLSessionTask,
    willPerformHTTPRedirection response: HTTPURLResponse,
    newRequest request: URLRequest,
    completionHandler: @escaping (URLRequest?) -> Void)
{
    var redirectRequest: URLRequest? = request

    // 如果提供了taskWillPerformHTTPRedirection，则更新redirectRequest
    if let taskWillPerformHTTPRedirection = taskWillPerformHTTPRedirection {
        redirectRequest = taskWillPerformHTTPRedirection(session, task, response, request)
    }

    completionHandler(redirectRequest)
}

// 服务器请求认证
@objc(URLSession:task:didReceiveChallenge:completionHandler:)
func urlSession(
    _ session: URLSession,
    task: URLSessionTask,
    didReceive challenge: URLAuthenticationChallenge,
    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void)
{
    // 初始化一个默认的配置，就像客户端没有实现这个回调
    var disposition: URLSession.AuthChallengeDisposition = .performDefaultHandling
    var credential: URLCredential?

    if let taskDidReceiveChallenge = taskDidReceiveChallenge {
        // 如果taskDidReceiveChallenge，更新disposition和credential
        (disposition, credential) = taskDidReceiveChallenge(session, task, challenge)
    } else if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust { // 认证方法是ServerTrust
        
        // 获取保护区域的主机地址
        let host = challenge.protectionSpace.host

        // 判断我们是否提供了服务器信任策略
        if let serverTrustPolicy = session.serverTrustPolicyManager?.serverTrustPolicy(forHost: host),
            let serverTrust = challenge.protectionSpace.serverTrust
        {
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
    } else {
        if challenge.previousFailureCount > 0 { // 之前认证失败的次数大于0
            // 更新disposition为rejectProtectionSpace，当前这个challenge被拒绝
            disposition = .rejectProtectionSpace
        } else { // 之前认证失败的次数不大于0
            
            // 使用证书进行认证
            credential = self.credential ?? session.configuration.urlCredentialStorage?.defaultCredential(for: challenge.protectionSpace)

            if credential != nil {
                disposition = .useCredential
            }
        }
    }

    // 最后执行completionHandler
    completionHandler(disposition, credential)
}

// 服务器请求新的inputStream
@objc(URLSession:task:needNewBodyStream:)
func urlSession(
    _ session: URLSession,
    task: URLSessionTask,
    needNewBodyStream completionHandler: @escaping (InputStream?) -> Void)
{
    var bodyStream: InputStream?

    if let taskNeedNewBodyStream = taskNeedNewBodyStream {
        bodyStream = taskNeedNewBodyStream(session, task)
    }

    completionHandler(bodyStream)
}

// 任务完成
@objc(URLSession:task:didCompleteWithError:)
func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
    // 如果有taskDidCompleteWithError，则执行
    if let taskDidCompleteWithError = taskDidCompleteWithError {
        taskDidCompleteWithError(session, task, error)
    } else { // 没有taskDidCompleteWithError
        if let error = error {
            
            // 设置error
            if self.error == nil { self.error = error }

            // 如果error不为空的话，userInfo里面携带有resumeData
            if let downloadDelegate = self as? DownloadTaskDelegate,
                let resumeData = (error as NSError).userInfo[NSURLSessionDownloadTaskResumeData] as? Data
            {
                downloadDelegate.resumeData = resumeData
            }
        }
        
        // 执行队列里的所有operations
        queue.isSuspended = false
    }
}
```

### 2. `DataTaskDelegate`

#### 1) 属性和初始化方法

```swift
var dataTask: URLSessionDataTask { return task as! URLSessionDataTask }

// 重写父类的属性，服务器返回的数据
override var data: Data? {
    if dataStream != nil { // 如果有dataStream去处理data，这里的data就返回nil
        return nil
    } else {
        return mutableData // 返回从服务器接收到的数据
    }
}

var progress: Progress // 进度
var progressHandler: (closure: Request.ProgressHandler, queue: DispatchQueue)?

var dataStream: ((_ data: Data) -> Void)? // 处理数据的closure

private var totalBytesReceived: Int64 = 0
private var mutableData: Data // 用于存储从服务器接收到的数据

private var expectedContentLength: Int64? // 服务器文件的总大小

override init(task: URLSessionTask?) {
    mutableData = Data()
    progress = Progress(totalUnitCount: 0)

    super.init(task: task)
}

override func reset() {
    super.reset()

    progress = Progress(totalUnitCount: 0)
    totalBytesReceived = 0
    mutableData = Data()
    expectedContentLength = nil
}
```

### 3. `DownloadTaskDelegate` & `UploadTaskDelegate`

这两个delegate与`DataTaskDelegate`非常类似，大家可以自己看下。