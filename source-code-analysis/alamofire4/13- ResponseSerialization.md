主要通过扩展`Request`、`DataRequest`和`DownloadRequest`将服务器返回的响应序列化成我们想要的数据。

### 1. 辅助类型

#### 1) `DataResponseSerializerProtocol`和`DataResponseSerializer<Value>`

```swift
// DataResponse序列化器协议，所有的DataResponse序列化器必须遵循这个协议
public protocol DataResponseSerializerProtocol {
    // 序列化后的对象类型
    associatedtype SerializedObject

    // 这个闭包告诉我们如何序列化
    var serializeResponse: (URLRequest?, HTTPURLResponse?, Data?, Error?) -> Result<SerializedObject> { get }
}

// 通用的DataResponse序列化器
public struct DataResponseSerializer<Value>: DataResponseSerializerProtocol {
    // 遵循DataResponseSerializerProtocol协议
    public typealias SerializedObject = Value

    // 遵循DataResponseSerializerProtocol协议
    public var serializeResponse: (URLRequest?, HTTPURLResponse?, Data?, Error?) -> Result<Value>

    // 初始化方法
    public init(serializeResponse: @escaping (URLRequest?, HTTPURLResponse?, Data?, Error?) -> Result<Value>) {
        self.serializeResponse = serializeResponse
    }
}
```

#### 2) `DownloadResponseSerializerProtocol`和`DownloadResponseSerializer<Value>`

与上面的两个类似，大家对照远吗自己看下。

### 2. `Timeline`属性

通过扩展为`Request`添加了timeline属性。

```swift
extension Request {
    var timeline: Timeline {
        let requestStartTime = self.startTime ?? CFAbsoluteTimeGetCurrent()
        let requestCompletedTime = self.endTime ?? CFAbsoluteTimeGetCurrent()
        let initialResponseTime = self.delegate.initialResponseTime ?? requestCompletedTime

        return Timeline(
            requestStartTime: requestStartTime,
            initialResponseTime: initialResponseTime,
            requestCompletedTime: requestCompletedTime,
            serializationCompletedTime: CFAbsoluteTimeGetCurrent()
        )
    }
}
```

### 3. 为`DataRequest`添加默认的序列化方法

这两个方法都没有将响应结果序列化成一个具体的类型。

```swift
// 这个方法没有把响应结果序列化成具体的类型
//
// queue：指定completionHandler在哪个队列执行，默认在主队列
// 把completionHandler添加到delegate的队列，请求完成后执行队列
// 在completionHandler中返回默认的数据响应DefaultDataResponse（没有经过序列化的）
@discardableResult
public func response(queue: DispatchQueue? = nil, completionHandler: @escaping (DefaultDataResponse) -> Void) -> Self {
    delegate.queue.addOperation {
        (queue ?? DispatchQueue.main).async {
            var dataResponse = DefaultDataResponse(
                request: self.request,
                response: self.response,
                data: self.delegate.data,
                error: self.delegate.error,
                timeline: self.timeline
            )

            dataResponse.add(self.delegate.metrics)

            completionHandler(dataResponse)
        }
    }

    return self
}

// 这个方法需要一个序列化器参数，会把响应结果序列化成序列化器需要的具体类型
//
// queue：指定completionHandler在哪个队列执行，默认在主队列
// 把completionHandler添加到delegate的队列，请求完成后执行队列
// 在completionHandler中返回默认的数据响应DefaultDataResponse（经过序列化的）
@discardableResult
public func response<T: DataResponseSerializerProtocol>(
    queue: DispatchQueue? = nil,
    responseSerializer: T,
    completionHandler: @escaping (DataResponse<T.SerializedObject>) -> Void)
    -> Self
{
    delegate.queue.addOperation {
        let result = responseSerializer.serializeResponse(
            self.request,
            self.response,
            self.delegate.data,
            self.delegate.error
        )

        var dataResponse = DataResponse<T.SerializedObject>(
            request: self.request,
            response: self.response,
            data: self.delegate.data,
            result: result,
            timeline: self.timeline
        )

        dataResponse.add(self.delegate.metrics)

        (queue ?? DispatchQueue.main).async { completionHandler(dataResponse) }
    }

    return self
}
```

### 4. 为`DownloadRequest`添加默认的序列化方法

这两个方法跟上面的两个类似，不做过多说明。

```swift
@discardableResult
public func response(
    queue: DispatchQueue? = nil,
    completionHandler: @escaping (DefaultDownloadResponse) -> Void)
    -> Self
{
    // ...
}

@discardableResult
public func response<T: DownloadResponseSerializerProtocol>(
    queue: DispatchQueue? = nil,
    responseSerializer: T,
    completionHandler: @escaping (DownloadResponse<T.SerializedObject>) -> Void)
    -> Self
{
    // ...
}
```

### 5. 把响应序列化成`Data`

```swift
extension Request {
    // 返回序列化后的结果
    public static func serializeResponseData(response: HTTPURLResponse?, data: Data?, error: Error?) -> Result<Data> {
        guard error == nil else { return .failure(error!) }
    
        // 如果返回的状态码是204(没有数据返回)和205(重置数据)，返回空数据
        if let response = response, emptyDataStatusCodes.contains(response.statusCode) { return .success(Data()) }

        guard let validData = data else {
            return .failure(AFError.responseSerializationFailed(reason: .inputDataNil))
        }

        return .success(validData)
    }
}

extension DataRequest {
    // 创建一个数据响应序列化器，把响应序列化成Data类型
    public static func dataResponseSerializer() -> DataResponseSerializer<Data> {
        // 这里其实是初始化DataResponseSerializer
        // 本来是DataResponseSerializer(serializeResponse: closure)，
        // 因为closure是语句中的最后一个closure，所以可以把小括号和serializeResponse省略
        return DataResponseSerializer { _, response, data, error in
            return Request.serializeResponseData(response: response, data: data, error: error)
        }
    }

    // 使用上面创建的数据响应序列化器，把响应序列化成Data
    @discardableResult
    public func responseData(
        queue: DispatchQueue? = nil,
        completionHandler: @escaping (DataResponse<Data>) -> Void)
        -> Self
    {
        return response(
            queue: queue,
            responseSerializer: DataRequest.dataResponseSerializer(),
            completionHandler: completionHandler
        )
    }
}

extension DownloadRequest {
    // 创建一个下载响应序列化器，把响应序列化成Data类型
    public static func dataResponseSerializer() -> DownloadResponseSerializer<Data> {
        return DownloadResponseSerializer { _, response, fileURL, error in
            guard error == nil else { return .failure(error!) }

            guard let fileURL = fileURL else {
                return .failure(AFError.responseSerializationFailed(reason: .inputFileNil))
            }

            do {
                let data = try Data(contentsOf: fileURL)
                return Request.serializeResponseData(response: response, data: data, error: error)
            } catch {
                return .failure(AFError.responseSerializationFailed(reason: .inputFileReadFailed(at: fileURL)))
            }
        }
    }

    // 使用上面创建的下载响应序列化器，把响应序列化成Data
    @discardableResult
    public func responseData(
        queue: DispatchQueue? = nil,
        completionHandler: @escaping (DownloadResponse<Data>) -> Void)
        -> Self
    {
        return response(
            queue: queue,
            responseSerializer: DownloadRequest.dataResponseSerializer(),
            completionHandler: completionHandler
        )
    }
}
```

### 5. 把响应序列化成`String` & `JSON` & `Property List`

把响应序列化成这三种类型的思路，跟序列化成`Data`的思路完全一致，大家可以自己看下，如果有不懂的，欢迎留言，我会尽力解答。
