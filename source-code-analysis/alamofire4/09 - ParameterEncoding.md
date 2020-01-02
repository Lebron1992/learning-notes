这个文件里面写了三种参数编码的方式：`URLEncoding`、`JSONEncoding`和`PropertyListEncoding`。

### 1. 辅助类型

```swift
// 用枚举列举了请求方法，并且用大写的形式指定了rawValue
public enum HTTPMethod: String {
    case options = "OPTIONS"
    case get     = "GET"
    case head    = "HEAD"
    case post    = "POST"
    case put     = "PUT"
    case patch   = "PATCH"
    case delete  = "DELETE"
    case trace   = "TRACE"
    case connect = "CONNECT"
}

// Dictionary类型的参数
public typealias Parameters = [String: Any]

```

### 2. `ParameterEncoding`协议

规定了如何把参数编码到请求当中。

```swift
public protocol ParameterEncoding {
    func encode(_ urlRequest: URLRequestConvertible, with parameters: Parameters?) throws -> URLRequest
}
```

### 3. `URLEncoding`

把参数直接编码到URL中。

#### 1). `Destination`

列举了参数编码到请求中的三种方式。

```swift
// methodDependent: 由请求方法自己决定，`GET`, `HEAD` 和 `DELETE`直接拼接到请求url中，其他方法放到HTTP body
// queryString: 直接拼接到请求url中
// httpBody: 放到HTTP body
public enum Destination {
    case methodDependent, queryString, httpBody
}
```

#### 2). 属性和初始化

```swift
// 编码方式是methodDependent的URLEncoding实例
public static var `default`: URLEncoding { return URLEncoding() }

// 编码方式是methodDependent的URLEncoding实例，其实跟default是相同的
public static var methodDependent: URLEncoding { return URLEncoding() }

// 编码方式是queryString的URLEncoding实例
public static var queryString: URLEncoding { return URLEncoding(destination: .queryString) }

// 编码方式是httpBody的URLEncoding实例
public static var httpBody: URLEncoding { return URLEncoding(destination: .httpBody) }

public let destination: Destination

public init(destination: Destination = .methodDependent) {
    self.destination = destination
}
```

#### 3). 编码

```swift
// 实现ParameterEncoding协议
public func encode(_ urlRequest: URLRequestConvertible, with parameters: Parameters?) throws -> URLRequest {
    var urlRequest = try urlRequest.asURLRequest()
    
    // 如果没有传入参数，直接返回
    guard let parameters = parameters else { return urlRequest }
    
    // 把参数编码到URL中
    if let method = HTTPMethod(rawValue: urlRequest.httpMethod ?? "GET"), encodesParametersInURL(with: method) {
        guard let url = urlRequest.url else {
            throw AFError.parameterEncodingFailed(reason: .missingURL)
        }
        
        if var urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: false), !parameters.isEmpty {
            let percentEncodedQuery = (urlComponents.percentEncodedQuery.map { $0 + "&" } ?? "") + query(parameters)
            urlComponents.percentEncodedQuery = percentEncodedQuery
            urlRequest.url = urlComponents.url
        }
    } else { // 把参数编码到HTTP body
        if urlRequest.value(forHTTPHeaderField: "Content-Type") == nil {
            urlRequest.setValue("application/x-www-form-urlencoded; charset=utf-8", forHTTPHeaderField: "Content-Type")
        }
        
        urlRequest.httpBody = query(parameters).data(using: .utf8, allowLossyConversion: false)
    }
    
    return urlRequest
}

// 根据传入的key和value，返回处理好的查询字符串组件
public func queryComponents(fromKey key: String, value: Any) -> [(String, String)] {
    var components: [(String, String)] = []
    
    if let dictionary = value as? [String: Any] { // value是字典
        for (nestedKey, value) in dictionary {
            // 用递归再次处理key和value
            components += queryComponents(fromKey: "\(key)[\(nestedKey)]", value: value)
        }
    } else if let array = value as? [Any] { // value是数组
        for value in array {
            // 用递归再次处理key和value
            components += queryComponents(fromKey: "\(key)[]", value: value)
        }
    } else if let value = value as? NSNumber { // value是NSNumber
        if value.isBool {
            // NSNumber其实是Bool类型，用1或者0代替
            components.append((escape(key), escape((value.boolValue ? "1" : "0"))))
        } else {
            components.append((escape(key), escape("\(value)")))
        }
    } else if let bool = value as? Bool { // value是Bool类型，用1或者0代替
        components.append((escape(key), escape((bool ? "1" : "0"))))
    } else {
        components.append((escape(key), escape("\(value)")))
    }
    
    return components
}

// 对字符串进行百分号编码。
// 下面这些分隔符保留原字符串：
// - General Delimiters: ":", "#", "[", "]", "@"
// - Sub-Delimiters: "!", "$", "&", "'", "(", ")", "*", "+", ",", ";", "="


public func escape(_ string: String) -> String {
    let generalDelimitersToEncode = ":#[]@"
    let subDelimitersToEncode = "!$&'()*+,;="
    
    var allowedCharacterSet = CharacterSet.urlQueryAllowed
    allowedCharacterSet.remove(charactersIn: "\(generalDelimitersToEncode)\(subDelimitersToEncode)")
    
    var escaped = ""
    
    //==========================================================================================================
    //
    //  在iOS 8.1和8.2中，如果一次性对大量的中文字符进行百分号编码会造成crash，所以进行了分批处理
    //  （这个bug是国人发现的，具体可以查看这个issue：https://github.com/Alamofire/Alamofire/issues/206）
    //
    //==========================================================================================================
    
    if #available(iOS 8.3, *) {
        escaped = string.addingPercentEncoding(withAllowedCharacters: allowedCharacterSet) ?? string
    } else {
        let batchSize = 50
        var index = string.startIndex
        
        while index != string.endIndex {
            let startIndex = index
            let endIndex = string.index(index, offsetBy: batchSize, limitedBy: string.endIndex) ?? string.endIndex
            let range = startIndex..<endIndex
            
            let substring = string[range]
            
            escaped += substring.addingPercentEncoding(withAllowedCharacters: allowedCharacterSet) ?? String(substring)
            
            index = endIndex
        }
    }
    
    return escaped
}

// 返回一个进行了百分号编码并且拼接好的字符串
private func query(_ parameters: [String: Any]) -> String {
    var components: [(String, String)] = []
    
    // `<`这个其实是一个其实是一个closure，意思是让keys按照从小到大的顺序重新排列
    for key in parameters.keys.sorted(by: <) {
        let value = parameters[key]!
        components += queryComponents(fromKey: key, value: value)
    }
    return components.map { "\($0)=\($1)" }.joined(separator: "&")
}

// 判断是否要把参数编码到URL里
private func encodesParametersInURL(with method: HTTPMethod) -> Bool {
    switch destination {
    case .queryString:
        return true
    case .httpBody:
        return false
    default:
        break
    }
    
    switch method {
    case .get, .head, .delete:
        return true
    default:
        return false
    }
}
```

### 4. `JSONEncoding`

把参数以SJON类型编码到请求体中。

#### 1). 属性和初始化

```swift
// 默认的JSONEncoding实例
public static var `default`: JSONEncoding { return JSONEncoding() }

// 用更好输出效果的JSONEncoding实例
public static var prettyPrinted: JSONEncoding { return JSONEncoding(options: .prettyPrinted) }

public let options: JSONSerialization.WritingOptions

public init(options: JSONSerialization.WritingOptions = []) {
    self.options = options
}
```

#### 2). 编码

```swift
// 把字典编码到请求体中
public func encode(_ urlRequest: URLRequestConvertible, with parameters: Parameters?) throws -> URLRequest {
    var urlRequest = try urlRequest.asURLRequest()
    
    // 没有参数，直接返回
    guard let parameters = parameters else { return urlRequest }
    
    do {
        // 把参数序列化成JSON
        let data = try JSONSerialization.data(withJSONObject: parameters, options: options)
        
        // 设置Content-Type
        if urlRequest.value(forHTTPHeaderField: "Content-Type") == nil {
            urlRequest.setValue("application/json", forHTTPHeaderField: "Content-Type")
        }
        
        urlRequest.httpBody = data
    } catch {
        throw AFError.parameterEncodingFailed(reason: .jsonEncodingFailed(error: error))
    }
    
    return urlRequest
}

// 其实根上面那个方法几乎一样，只是第二个参数类型不同
public func encode(_ urlRequest: URLRequestConvertible, withJSONObject jsonObject: Any? = nil) throws -> URLRequest {
    var urlRequest = try urlRequest.asURLRequest()
    
    guard let jsonObject = jsonObject else { return urlRequest }
    
    do {
        // 把参数序列化成JSON
        let data = try JSONSerialization.data(withJSONObject: jsonObject, options: options)
        
        // 设置Content-Type
        if urlRequest.value(forHTTPHeaderField: "Content-Type") == nil {
            urlRequest.setValue("application/json", forHTTPHeaderField: "Content-Type")
        }
        
        urlRequest.httpBody = data
    } catch {
        throw AFError.parameterEncodingFailed(reason: .jsonEncodingFailed(error: error))
    }
    
    return urlRequest
}
```

### 5. `PropertyListEncoding`

把参数以PropertyList类型编码到请求体中。

#### 1). 属性和初始化

```swift
// 默认的PropertyListEncoding实例，以xml的形式序列化
public static var `default`: PropertyListEncoding { return PropertyListEncoding() }

// 以xm的形式序列化的PropertyListEncoding实例
public static var xml: PropertyListEncoding { return PropertyListEncoding(format: .xml) }

// 以二进制的形式序列化的PropertyListEncoding实例
public static var binary: PropertyListEncoding { return PropertyListEncoding(format: .binary) }

public let format: PropertyListSerialization.PropertyListFormat
public let options: PropertyListSerialization.WriteOptions

public init(
    format: PropertyListSerialization.PropertyListFormat = .xml,
    options: PropertyListSerialization.WriteOptions = 0)
{
    self.format = format
    self.options = options
}
```

#### 2). 编码

```swift
// 实现ParameterEncoding协议
public func encode(_ urlRequest: URLRequestConvertible, with parameters: Parameters?) throws -> URLRequest {
    var urlRequest = try urlRequest.asURLRequest()
    
    guard let parameters = parameters else { return urlRequest }
    
    do {
        // 把参数序列化成PropertyList
        let data = try PropertyListSerialization.data(
            fromPropertyList: parameters,
            format: format,
            options: options
        )
        
        // 设置Content-Type
        if urlRequest.value(forHTTPHeaderField: "Content-Type") == nil {
            urlRequest.setValue("application/x-plist", forHTTPHeaderField: "Content-Type")
        }
        
        urlRequest.httpBody = data
    } catch {
        throw AFError.parameterEncodingFailed(reason: .propertyListEncodingFailed(error: error))
    }
    
    return urlRequest
}
```
