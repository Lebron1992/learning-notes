这个文件里面主要定义了各种请求类型。

### 1. `RequestAdapter`协议

`RequestAdapter`协议允许`SessionManager`的`Request`在创建的时候被适配。

```swift
public protocol RequestAdapter {
    func adapt(_ urlRequest: URLRequest) throws -> URLRequest
}
```

举个例子，大家就知道`RequestAdapter`用来干嘛的了。例如我们在请求的时候需要把`accessToken`拼接到请求头：

```swift
class AccessTokenAdapter: RequestAdapter {
    private let accessToken: String

    init(accessToken: String) {
        self.accessToken = accessToken
    }

    func adapt(_ urlRequest: URLRequest) throws -> URLRequest {
        var urlRequest = urlRequest

        if let urlString = urlRequest.url?.absoluteString, urlString.hasPrefix("https://httpbin.org") {
            urlRequest.setValue("Bearer " + accessToken, forHTTPHeaderField: "Authorization")
        }

        return urlRequest
    }
}

// 使用
let sessionManager = SessionManager()
sessionManager.adapter = AccessTokenAdapter(accessToken: "1234")

// 在创建请求的时候就会把accessToken加进去
sessionManager.request("https://httpbin.org/get")
```

### 2. 重试

```swift
// 一个closure类型，给`RequestRetrier`判断是否需要重试
public typealias RequestRetryCompletion = (_ shouldRetry: Bool, _ timeDelay: TimeInterval) -> Void

// 请求重试器协议
public protocol RequestRetrier {
    // 决定请求是否需要重试
    func should(_ manager: SessionManager, retry request: Request, with error: Error, completion: @escaping RequestRetryCompletion)
}
```

### 3. `TaskConvertible`协议

这个协议的主要目的是，让遵循这个协议的类型，通过实现协议的方法来创建`URLSessionTask`

```swift
protocol TaskConvertible {
    func task(session: URLSession, adapter: RequestAdapter?, queue: DispatchQueue) throws -> URLSessionTask
}
```

### 4. `Request`类

#### 1）辅助类型

```swift
// 监测上传或者下载进度的closure类型
public typealias ProgressHandler = (Progress) -> Void

// 列举了请求任务的类型，并携带了相关的关联值
enum RequestTask {
    case data(TaskConvertible?, URLSessionTask?)
    case download(TaskConvertible?, URLSessionTask?)
    case upload(TaskConvertible?, URLSessionTask?)
    case stream(TaskConvertible?, URLSessionTask?)
}
```

#### 2）属性

```swift
// TaskDelegate，用于处理URLSessionTask的所有callback
// internal(set): 这个指的是只能在内部赋值
// get和set都加了锁，防止多线程同时get和set
open internal(set) var delegate: TaskDelegate {
    get {
        taskDelegateLock.lock() ; defer { taskDelegateLock.unlock() }
        return taskDelegate
    }
    set {
        taskDelegateLock.lock() ; defer { taskDelegateLock.unlock() }
        taskDelegate = newValue
    }
}

open var task: URLSessionTask? { return delegate.task }
open let session: URLSession
open var request: URLRequest? { return task?.originalRequest }
open var response: HTTPURLResponse? { return task?.response as? HTTPURLResponse }
open internal(set) var retryCount: UInt = 0
let originalTask: TaskConvertible?

// 用于记录请求的时间
var startTime: CFAbsoluteTime?
var endTime: CFAbsoluteTime?

var validations: [() -> Void] = []

private var taskDelegate: TaskDelegate
private var taskDelegateLock = NSLock()
```

#### 3）初始化方法

```swift
init(session: URLSession, requestTask: RequestTask, error: Error? = nil) {
    self.session = session

	// 根据请求任务的类型，创建taskDelegate
    switch requestTask {
    case .data(let originalTask, let task):
        taskDelegate = DataTaskDelegate(task: task)
        self.originalTask = originalTask
    case .download(let originalTask, let task):
        taskDelegate = DownloadTaskDelegate(task: task)
        self.originalTask = originalTask
    case .upload(let originalTask, let task):
        taskDelegate = UploadTaskDelegate(task: task)
        self.originalTask = originalTask
    case .stream(let originalTask, let task):
        taskDelegate = TaskDelegate(task: task)
        self.originalTask = originalTask
    }

    delegate.error = error
    
    把请求结束的时间用队列记录下来，在请求完成后会执行这个队列
    delegate.queue.addOperation { self.endTime = CFAbsoluteTimeGetCurrent() }
}
```

#### 4）验证

下面的前两个方法把`Self`作为返回值，方便我们把同一个类的两个方法链接起来调用，例如：

```swift
let user = "user"
let password = "password"

Alamofire.request("https://httpbin.org/basic-auth/\(user)/\(password)")
    .authenticate(user: user, password: password)
    .responseJSON { response in
        debugPrint(response)
}

// 可以通过这个方法来关联一个HTTP基础证书
@discardableResult
open func authenticate(
    user: String,
    password: String,
    persistence: URLCredential.Persistence = .forSession)
    -> Self
{
    let credential = URLCredential(user: user, password: password, persistence: persistence)
    return authenticate(usingCredential: credential)
}

// 可以通过这个方法来关联一个指定的证书
@discardableResult
open func authenticate(usingCredential credential: URLCredential) -> Self {
    delegate.credential = credential
    return self
}

// 返回一个base64编码的验证头
open static func authorizationHeader(user: String, password: String) -> (key: String, value: String)? {
    guard let data = "\(user):\(password)".data(using: .utf8) else { return nil }

    let credential = data.base64EncodedString(options: [])

    return (key: "Authorization", value: "Basic \(credential)")
}
```

#### 5）状态

```swift
/// 开始请求
open func resume() {
    guard let task = task else { delegate.queue.isSuspended = false ; return }

    if startTime == nil { startTime = CFAbsoluteTimeGetCurrent() }

    task.resume()

    NotificationCenter.default.post(
        name: Notification.Name.Task.DidResume,
        object: self,
        userInfo: [Notification.Key.Task: task]
    )
}

/// 暂停请求
open func suspend() {
    guard let task = task else { return }

    task.suspend()

    NotificationCenter.default.post(
        name: Notification.Name.Task.DidSuspend,
        object: self,
        userInfo: [Notification.Key.Task: task]
    )
}

/// 取消请求
open func cancel() {
    guard let task = task else { return }

    task.cancel()

    NotificationCenter.default.post(
        name: Notification.Name.Task.DidCancel,
        object: self,
        userInfo: [Notification.Key.Task: task]
    )
}
```

另外还实现了`CustomStringConvertible`和`CustomDebugStringConvertible`，方便调试使用。

### 5. `DataRequest`类

继承于`Request`。

#### 1) Requestable
内置了`Requestable`，并实现`TaskConvertible`协议。目的是用`urlRequest`创建一个dataTask。

```swift
struct Requestable: TaskConvertible {
    let urlRequest: URLRequest

    func task(session: URLSession, adapter: RequestAdapter?, queue: DispatchQueue) throws -> URLSessionTask {
        do {
            let urlRequest = try self.urlRequest.adapt(using: adapter)
            return queue.sync { session.dataTask(with: urlRequest) }
        } catch {
            throw AdaptError(error: error)
        }
    }
}
```

#### 2) 属性

```swift
// DataRequest对应的请求
open override var request: URLRequest? {
    if let request = super.request { return request }
    if let requestable = originalTask as? Requestable { return requestable.urlRequest }

    return nil
}

// 从服务器获取数据的进度
open var progress: Progress { return dataDelegate.progress }

// DataTask对应的delegate，负责处理与data task相关的所有回调。在父类的初始化方法已经初始化过，这里直接返回，并且强转为DataTaskDelegate类型
// 在使用强转的时候要非常小心，如果不能确定能强转成功的，请务必使用`if let`，否则如果强转不成功，程序crash
var dataDelegate: DataTaskDelegate { return delegate as! DataTaskDelegate }

// 调用这个方法，设置一个closure，这个closure会在数据从服务器返回过程中周期性的调用；
// closure里面的data只包含最近从服务器获取的数据，不包含之前获取的数据；
// 如果调用了这个方法设置closure，那么下载的数据只能从这里访问，不会在其他地方存储，
// 并且`Response`对象中的data为`nil`
@discardableResult
open func stream(closure: ((Data) -> Void)? = nil) -> Self {
    dataDelegate.dataStream = closure
    return self
}

// 调用这个方法，设置一个closure，这个closure会在数据从服务器返回过程中周期性的调用，用于跟踪下载进度
@discardableResult
open func downloadProgress(queue: DispatchQueue = DispatchQueue.main, closure: @escaping ProgressHandler) -> Self {
    dataDelegate.progressHandler = (closure, queue)
    return self
}
```

### 6. `DownloadRequest`类

继承于`Request`。

#### 1) `DownloadOptions`

是一个OptionSet类型，用于指定一些下载选项:

```swift
public struct DownloadOptions: OptionSet {
    public let rawValue: UInt

    // 创建中间目录
    public static let createIntermediateDirectories = DownloadOptions(rawValue: 1 << 0)

    // 删除之前的文件
    public static let removePreviousFile = DownloadOptions(rawValue: 1 << 1)

    public init(rawValue: UInt) {
        self.rawValue = rawValue
    }
}
```

#### 2) `DownloadFileDestination`

下载完成后，会执行这个closure，程序会把下载完成后把文件临时存放在`temporaryURL`，然后再移动到`destinationURL`。

```swift
public typealias DownloadFileDestination = (
    _ temporaryURL: URL,
    _ response: HTTPURLResponse)
    -> (destinationURL: URL, options: DownloadOptions)
```

#### 3) `Downloadable`

内置了`Downloadable`，并实现`TaskConvertible`协议。目的是用`urlRequest`创建一个downloadTask。创建下载任务的时候有两种情况：1）全新的下载；2）之前下载好了部分数据，接着继续下载。

```swift
enum Downloadable: TaskConvertible {
    case request(URLRequest)
    case resumeData(Data)

    func task(session: URLSession, adapter: RequestAdapter?, queue: DispatchQueue) throws -> URLSessionTask {
        do {
            let task: URLSessionTask

            switch self {
            case let .request(urlRequest):
                let urlRequest = try urlRequest.adapt(using: adapter)
                task = queue.sync { session.downloadTask(with: urlRequest) }
            case let .resumeData(resumeData):
                task = queue.sync { session.downloadTask(withResumeData: resumeData) }
            }

            return task
        } catch {
            throw AdaptError(error: error)
        }
    }
}
```

#### 4) 属性

```swift
// DownloadRequest对应的请求
open override var request: URLRequest? {
    if let request = super.request { return request }

    if let downloadable = originalTask as? Downloadable, case let .request(urlRequest) = downloadable {
        return urlRequest
    }

    return nil
}

// 已经下载好的数据，用于继续下载
open var resumeData: Data? { return downloadDelegate.resumeData }

// 下载进度
open var progress: Progress { return downloadDelegate.progress }

// // DownloadTask对应的delegate，负责处理URLSessionDownloadDelegate的所有回调。在父类的初始化方法已经初始化过，这里直接返回，并且强转为DownloadTaskDelegate类型
var downloadDelegate: DownloadTaskDelegate { return delegate as! DownloadTaskDelegate }
```

#### 5) 方法

```swift

/// 重写父类的cancel方法
open override func cancel() {
  // 取消下载的时候要把resumeData记录起来，方便后续继续下载
    downloadDelegate.downloadTask.cancel { self.downloadDelegate.resumeData = $0 }

    NotificationCenter.default.post(
        name: Notification.Name.Task.DidCancel,
        object: self,
        userInfo: [Notification.Key.Task: task as Any]
    )
}

// 调用这个方法，设置一个closure，这个closure会在数据从服务器返回过程中周期性的调用，用于跟踪下载进度
@discardableResult
open func downloadProgress(queue: DispatchQueue = DispatchQueue.main, closure: @escaping ProgressHandler) -> Self {
    downloadDelegate.progressHandler = (closure, queue)
    return self
}

// 提供一个建议的DownloadFileDestination，最终会把下载好的文件移动到用户的documents目录
open class func suggestedDownloadDestination(
    for directory: FileManager.SearchPathDirectory = .documentDirectory,
    in domain: FileManager.SearchPathDomainMask = .userDomainMask)
    -> DownloadFileDestination
{
    return { temporaryURL, response in
        let directoryURLs = FileManager.default.urls(for: directory, in: domain)

        if !directoryURLs.isEmpty {
            return (directoryURLs[0].appendingPathComponent(response.suggestedFilename!), [])
        }

        return (temporaryURL, [])
    }
}
```

### 7. `UploadRequest`类

继承于`DataRequest`。

#### 1) `Uploadable`

内置了`Uploadable`，并实现`TaskConvertible`协议。目的是用`urlRequest`创建一个uploadTask。创建上传任务的时候有三种情况：1）上传数据；2）上传文件；3）上传流。

```swift
enum Uploadable: TaskConvertible {
    case data(Data, URLRequest)
    case file(URL, URLRequest)
    case stream(InputStream, URLRequest)
    
    func task(session: URLSession, adapter: RequestAdapter?, queue: DispatchQueue) throws -> URLSessionTask {
        do {
            let task: URLSessionTask
            
            switch self {
            case let .data(data, urlRequest):
                let urlRequest = try urlRequest.adapt(using: adapter)
                task = queue.sync { session.uploadTask(with: urlRequest, from: data) }
            case let .file(url, urlRequest):
                let urlRequest = try urlRequest.adapt(using: adapter)
                task = queue.sync { session.uploadTask(with: urlRequest, fromFile: url) }
            case let .stream(_, urlRequest):
                let urlRequest = try urlRequest.adapt(using: adapter)
                task = queue.sync { session.uploadTask(withStreamedRequest: urlRequest) }
            }
            
            return task
        } catch {
            throw AdaptError(error: error)
        }
    }
}
```

#### 2) 属性

```swift
// UploadRequest对应的请求
open override var request: URLRequest? {
    if let request = super.request { return request }
    
    guard let uploadable = originalTask as? Uploadable else { return nil }
    
    switch uploadable {
    case .data(_, let urlRequest), .file(_, let urlRequest), .stream(_, let urlRequest):
        return urlRequest
    }
}

// 上传进度
open var uploadProgress: Progress { return uploadDelegate.uploadProgress }

// // UploadTask对应的delegate，负责处理与上传相关的回调。在父类的初始化方法已经初始化过，这里直接返回，并且强转为UploadTaskDelegate类型
var uploadDelegate: UploadTaskDelegate { return delegate as! UploadTaskDelegate }

// 调用这个方法，设置一个closure，这个closure会在数据上传过程中周期性的调用，用于跟踪上传进度
@discardableResult
open func uploadProgress(queue: DispatchQueue = DispatchQueue.main, closure: @escaping ProgressHandler) -> Self {
    uploadDelegate.uploadProgressHandler = (closure, queue)
    return self
}
```

### 8. `StreamRequest`类

继承于`Request`，只能在iOS、macOS和tvOS使用。同样地，内置了`Streamable`，并实现`TaskConvertible`协议。目的是用host和端口或者`NetService`创建一个streamTask。

```swift
@available(iOS 9.0, macOS 10.11, tvOS 9.0, *)
open class StreamRequest: Request {
    enum Streamable: TaskConvertible {
        case stream(hostName: String, port: Int)
        case netService(NetService)
        
        func task(session: URLSession, adapter: RequestAdapter?, queue: DispatchQueue) throws -> URLSessionTask {
            let task: URLSessionTask
            
            switch self {
            case let .stream(hostName, port):
                task = queue.sync { session.streamTask(withHostName: hostName, port: port) }
            case let .netService(netService):
                task = queue.sync { session.streamTask(with: netService) }
            }
            
            return task
        }
}
```
