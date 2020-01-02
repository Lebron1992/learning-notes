这个文件主要处理服务器返回的响应。

### 1. `DefaultDataResponse`

存储数据请求或者上传请求的没有序列化的response的相关数据。

```swift
// 相关的请求
public let request: URLRequest?

/// 服务器返回的响应
public let response: HTTPURLResponse?

// 服务器返回的数据
public let data: Data?

// 遇到的错误
public let error: Error?

// 整个请求相关的时间点
public let timeline: Timeline

// 请求过程中的相关统计信息
var _metrics: AnyObject?

public init(
    request: URLRequest?,
    response: HTTPURLResponse?,
    data: Data?,
    error: Error?,
    timeline: Timeline = Timeline(),
    metrics: AnyObject? = nil)
{
    self.request = request
    self.response = response
    self.data = data
    self.error = error
    self.timeline = timeline
}
```

### 2. `DataResponse<Value>`

#### 1) 属性和初始化方法

```swift
public let request: URLRequest?
public let response: HTTPURLResponse?
public let data: Data?

// 一个带有关联值的泛型枚举，用于表示请求的结果
public let result: Result<Value>

public let timeline: Timeline

// 如果请求成功，返回上面那个result的`case success(Value)`的关联值，否则返回nil
public var value: Value? { return result.value }

// 如果请求失败，返回上面那个result的`case failure(Error)`的关联值，否则返回nil
public var error: Error? { return result.error }

var _metrics: AnyObject?

public init(
    request: URLRequest?,
    response: HTTPURLResponse?,
    data: Data?,
    result: Result<Value>,
    timeline: Timeline = Timeline())
{
    self.request = request
    self.response = response
    self.data = data
    self.result = result
    self.timeline = timeline
}
```

另外还实现了`CustomStringConvertible`和`CustomDebugStringConvertible`，方便调试。

关于泛型，大家可以前往这里了解: [【Swift 3.1】23 - 泛型 (Generics)](http://www.jianshu.com/p/ee8a4a227de4)。

#### 2) 提供了transform函数

作者还另外提供了四个transform函数：`map`、`flatMap`、`mapError`和`FlatMapError`。这些函数的作用类似于系统的集合自带的`map`，如果你理解了系统自带的`map`的意思，那么作者提供四个函数也能很好地理解。

```swift
// DataResponse的result是success的时候执行transform，使用案例：
//
//     let possibleData: DataResponse<Data> = ...
//     let possibleInt = possibleData.map { $0.count }
//
public func map<T>(_ transform: (Value) -> T) -> DataResponse<T> {
    var response = DataResponse<T>(
        request: request,
        response: self.response,
        data: data,
        result: result.map(transform),
        timeline: timeline
    )

    response._metrics = _metrics

    return response
}

// DataResponse的result是success的时候执行transform，使用案例：
//
//     let possibleData: DataResponse<Data> = ...
//     let possibleObject = possibleData.flatMap {
//         try JSONSerialization.jsonObject(with: $0)
//     }
//
public func flatMap<T>(_ transform: (Value) throws -> T) -> DataResponse<T> {
    var response = DataResponse<T>(
        request: request,
        response: self.response,
        data: data,
        result: result.flatMap(transform),
        timeline: timeline
    )

    response._metrics = _metrics

    return response
}

// DataResponse的result是failure的时候执行transform，使用案例：
//
//     let possibleData: DataResponse<Data> = ...
//     let withMyError = possibleData.mapError { MyError.error($0) }
//
public func mapError<E: Error>(_ transform: (Error) -> E) -> DataResponse {
    var response = DataResponse(
        request: request,
        response: self.response,
        data: data,
        result: result.mapError(transform),
        timeline: timeline
    )

    response._metrics = _metrics

    return response
}

// DataResponse的result是failure的时候执行transform，使用案例：
//
//     let possibleData: DataResponse<Data> = ...
//     let possibleObject = possibleData.flatMapError {
//         try someFailableFunction(taking: $0)
//     }
//
public func flatMapError<E: Error>(_ transform: (Error) throws -> E) -> DataResponse {
    var response = DataResponse(
        request: request,
        response: self.response,
        data: data,
        result: result.flatMapError(transform),
        timeline: timeline
    )

    response._metrics = _metrics

    return response
}
```

### 3. `DefaultDownloadResponse` & `DownloadResponse<Value>`

`DefaultDownloadResponse`类似于`DefaultDataResponse`；`DownloadResponse<Value>`类似于`DataResponse<Value>`。大家可以自己看下，如果有不懂的，欢迎留言，我会尽力解答。



### 4. `Response`

这个协议主要规定了如何添加`_metrics`。上面提到的四个类型都实现了这个协议。

```swift
protocol Response {
    /// The task metrics containing the request / response statistics.
    var _metrics: AnyObject? { get set }
    mutating func add(_ metrics: AnyObject?)
}
```
