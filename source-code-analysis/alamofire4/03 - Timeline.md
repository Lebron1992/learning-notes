`Timeline`这个类主要负责记录整个请求过程中的时间点。

```swift
public struct Timeline {
    /// 请求开始的时刻
    public let requestStartTime: CFAbsoluteTime

    /// 首次从服务器接收到数据或者发送数据给服务器的时刻
    public let initialResponseTime: CFAbsoluteTime

    /// 请求完成的时刻
    public let requestCompletedTime: CFAbsoluteTime

    /// 序列化完成的时刻
    public let serializationCompletedTime: CFAbsoluteTime

    /// 从请求开始到首次收到服务器响应的时间间隔：initialResponseTime - requestStartTime
    public let latency: TimeInterval

    /// 从请求开始到请求完成的时间间隔：requestCompletedTime - requestStartTime
    public let requestDuration: TimeInterval

    /// 从请求完成到序列化完成的时间间隔：serializationCompletedTime - requestCompletedTime
    public let serializationDuration: TimeInterval

    /// 从请求开始到序列化完成的时间间隔：serializationCompletedTime - requestStartTime
    public let totalDuration: TimeInterval
}
```

刚开始觉得`CFAbsoluteTime`很高深莫测，其实是一个`Double`类型。

```swift
public typealias CFTimeInterval = Double
public typealias CFAbsoluteTime = CFTimeInterval
```

另外还实现了`CustomStringConvertible`和`CustomDebugStringConvertible`协议，方便我们调试使用。

```swift

// MARK: - CustomStringConvertible

extension Timeline: CustomStringConvertible {
    public var description: String {
    }
}

// MARK: - CustomDebugStringConvertible

extension Timeline: CustomDebugStringConvertible {
    public var debugDescription: String {
    }
}
```
