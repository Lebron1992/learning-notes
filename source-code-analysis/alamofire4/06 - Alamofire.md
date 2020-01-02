Alamofire这个文件包含了所有请求的入口。

### 1. `URLConvertible`和`URLRequestConvertible`协议

我们知道URL可以有三个来源：1)`String`; 2) `URL`; 3) `URLComponents`。所以作者定义了一个`URLConvertible`协议，让着三个来源遵循这个协议，这样就可以直接使用`URLConvertible`类型的变量转换成URL。

```swift
public protocol URLConvertible {
    func asURL() throws -> URL
}

extension String: URLConvertible {
    public func asURL() throws -> URL {
        guard let url = URL(string: self) else { throw AFError.invalidURL(url: self) }
        return url
    }
}

extension URL: URLConvertible {
    public func asURL() throws -> URL { return self }
}

extension URLComponents: URLConvertible {
    public func asURL() throws -> URL {
        guard let url = url else { throw AFError.invalidURL(url: self) }
        return url
    }
}
```

类似地，用同样的思想作者也定义了`URLRequestConvertible`。

### 2. 初始化

```swift
extension URLRequest {
    // 请求的初始化方法, HTTPMethod是一个枚举；HTTPHeaders实际是[String: String]
    public init(url: URLConvertible, method: HTTPMethod, headers: HTTPHeaders? = nil) throws {
        let url = try url.asURL()

        self.init(url: url)

        httpMethod = method.rawValue

        if let headers = headers {
            for (headerField, headerValue) in headers {
                setValue(headerValue, forHTTPHeaderField: headerField)
            }
        }
    }

    // 使用自定义的请求适配器适配请求
    func adapt(using adapter: RequestAdapter?) throws -> URLRequest {
        guard let adapter = adapter else { return self }
        return try adapter.adapt(self)
    }
}
```

`RequestAdapter`是一个协议，我们可以自定义一个类型并遵循这个协议，可以针对这个请求做一些额外的配置，在[【Alamofire源码解析】08 - Request](http://www.jianshu.com/p/e1d1331128ae)有具体的例子讲解。

### 3. 数据请求

```swift
// 传入URLConvertible和其他信息
@discardableResult
public func request(
    _ url: URLConvertible,
    method: HTTPMethod = .get,
    parameters: Parameters? = nil,
    encoding: ParameterEncoding = URLEncoding.default,
    headers: HTTPHeaders? = nil)
    -> DataRequest
{
    return SessionManager.default.request(
        url,
        method: method,
        parameters: parameters,
        encoding: encoding,
        headers: headers
    )
}

// 传入URLRequestConvertible
@discardableResult
public func request(_ urlRequest: URLRequestConvertible) -> DataRequest {
    return SessionManager.default.request(urlRequest)
}
```

### 4. 下载请求 & 上传请求 & Stream请求

这三个类型的请求跟数据请求的代码大同小异，比较好理解。这里就不展开去讲了。
