在开发过程中，我们经常要考虑当前的网络状态。Alamofire作为一个网络框架，也提供了一个关于网络状态的工具类。

### 1. `NetworkReachabilityStatus` 和 `ConnectionType`

`NetworkReachabilityManager`类里面内置了这两个枚举类型：

```swift
// 网络是否可用
public enum NetworkReachabilityStatus {
    case unknown // 未知状态
    case notReachable // 不可用
    case reachable(ConnectionType) // 可用（关联了网络连接类型）
}

// 连接类型
public enum ConnectionType {
    case ethernetOrWiFi // 网线或WiFi
    case wwan // 移动网络
}
```

### 2. `NetworkReachabilityManager`类

```swift
// 定义了一个叫Listener的closure类型，网络状态会通过Listener传达出去
public typealias Listener = (NetworkReachabilityStatus) -> Void

// 网络是否可用
public var isReachable: Bool { return isReachableOnWWAN || isReachableOnEthernetOrWiFi }

// 移动网络是否可用
public var isReachableOnWWAN: Bool { return networkReachabilityStatus == .reachable(.wwan) }

// 网线或WiFi是否可用
public var isReachableOnEthernetOrWiFi: Bool { return networkReachabilityStatus == .reachable(.ethernetOrWiFi) }

// 当前网络状态
public var networkReachabilityStatus: NetworkReachabilityStatus {
    guard let flags = self.flags else { return .unknown }
    return networkReachabilityStatusForFlags(flags)
}

// 上面那个Listener的执行队列，是一个主队列
public var listenerQueue: DispatchQueue = DispatchQueue.main

// 网络状态会通过这个listener传达出去
public var listener: Listener?

// 当前的reachability flags。这是一个OptionSet类型
// 这些标记可以告诉我们：某个网络节点或地址是否可达；
// 通信之前是否要先连接；建立连接是否需要用户干预
private var flags: SCNetworkReachabilityFlags? {
    var flags = SCNetworkReachabilityFlags()

    if SCNetworkReachabilityGetFlags(reachability, &flags) {
        return flags
    }

    return nil
}

private let reachability: SCNetworkReachability
private var previousFlags: SCNetworkReachabilityFlags

// MARK: - 初始化方法

// 用一个特定的host创建NetworkReachabilityManager
public convenience init?(host: String) {
    guard let reachability = SCNetworkReachabilityCreateWithName(nil, host) else { return nil }
    self.init(reachability: reachability)
}

// 创建NetworkReachabilityManager，监听0.0.0.0（简单地说，它可以表示所有的IP地址），包括IPv4和IPv6
public convenience init?() {
    var address = sockaddr_in() // socket地址
    address.sin_len = UInt8(MemoryLayout<sockaddr_in>.size)
    address.sin_family = sa_family_t(AF_INET)

    guard let reachability = withUnsafePointer(to: &address, { pointer in
        return pointer.withMemoryRebound(to: sockaddr.self, capacity: MemoryLayout<sockaddr>.size) {
            return SCNetworkReachabilityCreateWithAddress(nil, $0)
        }
    }) else { return nil }

    self.init(reachability: reachability)
}

private init(reachability: SCNetworkReachability) {
    self.reachability = reachability
    self.previousFlags = SCNetworkReachabilityFlags()
}

deinit {
    stopListening()
}

// MARK: - Listening

//开始监听（@discardableResult表示返回值可以忽略，Xcode就不会有警告）
@discardableResult
public func startListening() -> Bool {
    var context = SCNetworkReachabilityContext(version: 0, info: nil, retain: nil, release: nil, copyDescription: nil)
    context.info = Unmanaged.passUnretained(self).toOpaque()

    let callbackEnabled = SCNetworkReachabilitySetCallback(
        reachability,
        { (_, flags, info) in
            let reachability = Unmanaged<NetworkReachabilityManager>.fromOpaque(info!).takeUnretainedValue()
            reachability.notifyListener(flags)
    },
        &context
    )

    let queueEnabled = SCNetworkReachabilitySetDispatchQueue(reachability, listenerQueue)

    listenerQueue.async {
        self.previousFlags = SCNetworkReachabilityFlags()
        self.notifyListener(self.flags ?? SCNetworkReachabilityFlags())
    }

    return callbackEnabled && queueEnabled
}

//停止监听
public func stopListening() {
    SCNetworkReachabilitySetCallback(reachability, nil, nil)
    SCNetworkReachabilitySetDispatchQueue(reachability, nil)
}

// MARK: - 通知listener，其实就是执行listener闭包

func notifyListener(_ flags: SCNetworkReachabilityFlags) {
    guard previousFlags != flags else { return }
    previousFlags = flags

    listener?(networkReachabilityStatusForFlags(flags))
}

// MARK: - 根据标记返回当前的网络状态

func networkReachabilityStatusForFlags(_ flags: SCNetworkReachabilityFlags) -> NetworkReachabilityStatus {
    guard isNetworkReachable(with: flags) else { return .notReachable }

    var networkStatus: NetworkReachabilityStatus = .reachable(.ethernetOrWiFi)

    #if os(iOS)
        if flags.contains(.isWWAN) { networkStatus = .reachable(.wwan) }
    #endif

    return networkStatus
}

func isNetworkReachable(with flags: SCNetworkReachabilityFlags) -> Bool {
    let isReachable = flags.contains(.reachable)
    let needsConnection = flags.contains(.connectionRequired)
    let canConnectAutomatically = flags.contains(.connectionOnDemand) || flags.contains(.connectionOnTraffic)
    let canConnectWithoutUserInteraction = canConnectAutomatically && !flags.contains(.interventionRequired)

    return isReachable && (!needsConnection || canConnectWithoutUserInteraction)
}
```

### 3. 总结

这个类里面涉及到了比较多底层网络的API，由于开发中我们几乎很少涉及，这里就不详细说了。
