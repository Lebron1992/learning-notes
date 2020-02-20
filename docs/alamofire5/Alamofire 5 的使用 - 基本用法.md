- [Alamofire 5 的使用 - 基本用法](#alamofire-5-的使用---基本用法)
  - [特性](#特性)
  - [组件库](#组件库)
  - [要求的使用环境](#要求的使用环境)
  - [安装方法](#安装方法)
  - [CocoaPods](#cocoapods)
  - [基本用法](#基本用法)
    - [介绍](#介绍)
      - [另外：`AF` 命名空间和引用](#另外af-命名空间和引用)
    - [发起请求](#发起请求)
      - [HTTP Methods](#http-methods)
      - [请求参数和参数编码器](#请求参数和参数编码器)
        - [`URLEncodedFormParameterEncoder`](#urlencodedformparameterencoder)
          - [使用 URL 编码参数的 GET 请求](#使用-url-编码参数的-get-请求)
          - [使用 URL 编码参数的 POST 请求](#使用-url-编码参数的-post-请求)
          - [配置已编码参数的排序](#配置已编码参数的排序)
          - [配置 `Array` 参数的编码](#配置-array-参数的编码)
          - [配置 `Bool` 参数的编码](#配置-bool-参数的编码)
          - [配置 `Data` 参数的编码](#配置-data-参数的编码)
          - [配置 `Date` 参数的编码](#配置-date-参数的编码)
          - [配置 Coding Keys 的编码](#配置-coding-keys-的编码)
          - [配置空格的编码](#配置空格的编码)
        - [`JSONParameterEncoder`](#jsonparameterencoder)
          - [JSON 编码参数的 POST 请求](#json-编码参数的-post-请求)
          - [自定义 `JSONEncoder`](#自定义-jsonencoder)
          - [手动对 `URLRequest` 进行参数编码](#手动对-urlrequest-进行参数编码)
      - [HTTP Headers](#http-headers)
      - [响应验证](#响应验证)
        - [自动验证](#自动验证)
        - [手动验证](#手动验证)
      - [响应处理](#响应处理)
        - [响应 Handler](#响应-handler)
        - [响应 Data Handler](#响应-data-handler)
        - [响应 String Handler](#响应-string-handler)
        - [响应 JSON Handler](#响应-json-handler)
        - [响应 `Decodable` Handler](#响应-decodable-handler)
        - [链式响应 handlers](#链式响应-handlers)
        - [响应 Handler 队列](#响应-handler-队列)
      - [响应缓存](#响应缓存)
      - [身份验证](#身份验证)
        - [HTTP Basic 身份验证](#http-basic-身份验证)
        - [使用 `URLCredential` 进行验证](#使用-urlcredential-进行验证)
        - [手动验证](#手动验证-1)
      - [下载数据到文件中](#下载数据到文件中)
        - [下载文件的存放位置](#下载文件的存放位置)
        - [下载进度](#下载进度)
        - [取消和恢复下载](#取消和恢复下载)
      - [上传数据到服务器](#上传数据到服务器)
        - [上传 Data](#上传-data)
        - [上传文件](#上传文件)
        - [上传多表单数据](#上传多表单数据)
        - [上传进度](#上传进度)
      - [统计指标](#统计指标)
        - [`URLSessionTaskMetrics`](#urlsessiontaskmetrics)
      - [cURL 的命令输出](#curl-的命令输出)

# Alamofire 5 的使用 - 基本用法

我分两篇文章介绍如何使用 Alamofire 5。文章的内容主要是翻译 [Alamofire](https://github.com/Alamofire/Alamofire) 的 readme。[第二篇文章 >>]()

## 特性

- 可链接的请求/响应函数
- URL / JSON 参数编码
- 上传文件 / Data / 流 / 多表单数据
- 使用请求或者恢复数据下载文件
- 使用 URLCredential 进行身份验证
- HTTP 响应验证
- 带有进度的上传和下载闭包
- cURL 命令的输出
- 动态调整和重试请求
- TLS 证书和公钥固定
- 网络可达性
- 全面的单元和集成测试覆盖率

## 组件库

为了让 Alamofire 专注于核心网络实现，Alamofire 软件基金会创建了额外的组件库，为 Alamofire 生态系统带来额外的功能。

- [AlamofireImage](https://github.com/Alamofire/AlamofireImage)：一个图片库，包括图像响应序列化器、`UIImage` 和 `UIImageView` 的扩展、自定义图像滤镜、自动清除内存缓存和基于优先级的图像下载系统。
- [AlamofireNetworkActivityIndicator](https://github.com/Alamofire/AlamofireNetworkActivityIndicator): 使用 Alamofire 控制 iOS 上网络活动指示器的可见性。它包含可配置的延迟计时器，有助于减少闪烁，并且可以支持不由 Alamofire 管理的 URLSession 实例。

## 要求的使用环境

- iOS 10.0+ / macOS 10.12+ / tvOS 10.0+ / watchOS 3.0+
- Xcode 10.2+
- Swift 5+

## 安装方法

### CocoaPods

```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '10.0'
use_frameworks!

target '项目名称' do
    pod 'Alamofire', '~> 5.0'
end
```

iOS 版本和 `Alamofire` 版本可以自己根据实际情况自行更改。CocoaPods 是比较常用的第三方库管理工具，其他方法就不详细说了。其他集成方法可以查看[原文档](https://github.com/Alamofire/Alamofire#installation)。

# 基本用法

## 介绍

Alamofire 为 HTTP 网络请求提供了一个优雅且可组合的接口。它没有实现自己的 HTTP 网络功能。取而代之的是，它建立在由 Foundation 框架提供的 [URL 加载系统](https://developer.apple.com/documentation/foundation/url_loading_system/)之上。系统的核心是 [`URLSession`](https://developer.apple.com/documentation/foundation/urlsession) 和 [`URLSessionTask`](https://developer.apple.com/documentation/foundation/urlsessiontask) 子类。Alamofire 将这些 APIs 和许多其他 APIs 封装在一个更易于使用的接口中，并提供使用 HTTP 网络进行现代应用程序开发所必需的各种功能。但是，了解 Alamofire 的许多核心行为来自何处很重要，因此熟悉 URL 加载系统非常重要。归根结底，Alamofire 的网络特性受到该系统功能的限制，应该始终记住并遵守其行为和最佳实践。

此外，Alamofire（以及 URL 加载系统）中的联网是异步完成的。异步编程可能会让不熟悉这个概念的程序员感到沮丧，但是有[很好的理由](https://developer.apple.com/library/ios/qa/qa1693/_index.html)这样做。

### 另外：`AF` 命名空间和引用

以前的 Alamofire 文档使用了类似 `Alamofire.request()` 的示例。这个 API 虽然看起来需要 `Alamofire` 前缀，但实际上在没有它的情况下也可以。`request` 方法和其他函数在任何带有 `import Alamofire` 的文件中都是全局可用的。从 Alamofire 5 开始，此功能已被删除，被更改为 `AF` ，它是对 `Session.default` 的引用。这允许 Alamofire 提供同样的便利功能，同时不必每次使用 Alamofire 时都污染全局命名空间，也不必全局复制 `Session` API。类似地，由 Alamofire 扩展的类型将使用 `af` 属性扩展来将 Alamofire 添加的功能与其他扩展分开。

## 发起请求

Alamofire 为发出 HTTP 请求提供了多种方便的方法。最简单的是，只需提供一个可以转换为 URL 的 `String` ：

```swift
AF.request("https://httpbin.org/get").response { response in
    debugPrint(response)
}
```

> 所有示例都需要在源文件中的某个位置 `import Alamofire`。

这实际上是 Alamofire `Session` 类型上用于发出请求的两个顶层 APIs 的一种形式。它的完整定义如下：

```swift
open func request<Parameters: Encodable>(
    _ convertible: URLConvertible,
    method: HTTPMethod = .get,
    parameters: Parameters? = nil,
    encoder: ParameterEncoder = URLEncodedFormParameterEncoder.default,
    headers: HTTPHeaders? = nil,
    interceptor: RequestInterceptor? = nil
) -> DataRequest
```

此方法创建一个 `DataRequest`，同时允许组合来自各个组件（如 `method` 和 `headers` ）的请求，同时还允许每个传入 `RequestInterceptors` 和 `Encodable` 参数。

> 还有其他方法允许您使用 `Parameters` 字典和 `ParameterEncoding` 类型来发出请求。不再推荐此 API，最终将被弃用并从 Alamofire 中删除。

这个 API 的第二个版本要简单得多：

```swift
open func request(
    _ urlRequest: URLRequestConvertible,
    interceptor: RequestInterceptor? = nil
) -> DataRequest
```

此方法为遵循 `Alamofire` 的 `URLRequestConvertible` 协议的任何类型创建 `DataRequest` 。所有不同于前一版本的参数都封装在该值中，这会产生非常强大的抽象。这将在我们的[高级用法]()中讨论。

### HTTP Methods

`HTTPMethod` 类型列出了 [RFC 7231 §4.3](https://tools.ietf.org/html/rfc7231#section-4.3) 中定义的 HTTP 方法：

```swift
public struct HTTPMethod: RawRepresentable, Equatable, Hashable {
    public static let connect = HTTPMethod(rawValue: "CONNECT")
    public static let delete = HTTPMethod(rawValue: "DELETE")
    public static let get = HTTPMethod(rawValue: "GET")
    public static let head = HTTPMethod(rawValue: "HEAD")
    public static let options = HTTPMethod(rawValue: "OPTIONS")
    public static let patch = HTTPMethod(rawValue: "PATCH")
    public static let post = HTTPMethod(rawValue: "POST")
    public static let put = HTTPMethod(rawValue: "PUT")
    public static let trace = HTTPMethod(rawValue: "TRACE")

    public let rawValue: String

    public init(rawValue: String) {
        self.rawValue = rawValue
    }
}
```

这些值可以作为 `method` 参数传递给 `AF.request` API：

```swift
AF.request("https://httpbin.org/get")
AF.request("https://httpbin.org/post", method: .post)
AF.request("https://httpbin.org/put", method: .put)
AF.request("https://httpbin.org/delete", method: .delete)
```

重要的是要记住，不同的 HTTP 方法可能有不同的语义，需要不同的参数编码，这取决于服务器的期望。例如，URLSession 或 Alamofire 不支持在 `GET` 请求中传递 `body` 数据，并将返回错误。

Alamofire 还提供了对 `URLRequest` 的扩展，以桥接将字符串返回到 `HTTPMethod` 值的 `httpMethod` 属性：

```swift
public extension URLRequest {
    /// Returns the `httpMethod` as Alamofire's `HTTPMethod` type.
    var method: HTTPMethod? {
        get { return httpMethod.flatMap(HTTPMethod.init) }
        set { httpMethod = newValue?.rawValue }
    }
}
```

如果需要使用 Alamofire 的 `HTTPMethod` 类型不支持的 HTTP 方法，可以扩展该类型以添加自定义值：

```swift
extension HTTPMethod {
    static let custom = HTTPMethod(rawValue: "CUSTOM")
}
```

### 请求参数和参数编码器

Alamofire 支持将任何 `Encodable` 类型作为请求的参数。然后，这些参数通过遵循 `ParameterEncoder` 协议的类型传递，并添加到 `URLRequest` 中，然后通过网络发送。Alamofire 包含两种遵循 `ParameterEncoder` `的类型：JSONParameterEncoder` 和 `URLEncodedFormParameterEncoder` 。这些类型涵盖了现代服务使用的最常见的编码。

```swift
struct Login: Encodable {
    let email: String
    let password: String
}

let login = Login(email: "test@test.test", password: "testPassword")

AF.request("https://httpbin.org/post",
           method: .post,
           parameters: login,
           encoder: JSONParameterEncoder.default).response { response in
    debugPrint(response)
}
```

#### `URLEncodedFormParameterEncoder`

`URLEncodedFormParameterEncoder` 将值编码为 URL 编码字符串，以将其设置为或附加到任何现有 URL 查询字符串，或设置为请求的 HTTP body。通过设置编码的目的地，可以控制编码字符串的设置位置。`URLEncodedFormParameterEncoder.Destination` 枚举有三种情况：

- `.methodDependent` - 对于 `.get`、`.head`、`.delete` 请求，它会将已编码查询字符串应用到现有的查询字符串中；对于其他类型的请求，会将其设置为 HTTP body。
- `.queryString` - 将编码字符串设置或追加到请求的 URL 中。
- `.httpBody` - 将编码字符串设置为 `URLRequest` 的 HTTP body。

如果尚未设置 `Content-Type`，那么会把具有 HTTP body 的已编码请求的 HTTP header 设置为 `application/x-www-form-urlencoded; charset=utf-8`。

在内部，`URLEncodedFormParameterEncoder` 使用 `URLEncodedFormEncoder` 把 `Encodable` 类型编码为 URL 编码形式的 `String`。此编码器可用于自定义各种类型的编码，包括使用 `ArrayEncoding` 的 `Array`、使用 `BoolEncoding` 的`Bool`、使用 `DataEncoding` 的 `Data`、使用 `DateEncoding` 的 `Date`、使用 `KeyEncoding` 的 keys 以及使用 `SpaceEncoding` 的空格。

##### 使用 URL 编码参数的 GET 请求

```swift
let parameters = ["foo": "bar"]

// 下面三种方法都是等价的
AF.request("https://httpbin.org/get", parameters: parameters) // encoding defaults to `URLEncoding.default`
AF.request("https://httpbin.org/get", parameters: parameters, encoder: URLEncodedFormParameterEncoder.default)
AF.request("https://httpbin.org/get", parameters: parameters, encoder: URLEncodedFormParameterEncoder(destination: .methodDependent))

// https://httpbin.org/get?foo=bar
```

##### 使用 URL 编码参数的 POST 请求

```swift
let parameters: [String: [String]] = [
    "foo": ["bar"],
    "baz": ["a", "b"],
    "qux": ["x", "y", "z"]
]

// 下面三种方法都是等价的
AF.request("https://httpbin.org/post", method: .post, parameters: parameters)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: URLEncodedFormParameterEncoder.default)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: URLEncodedFormParameterEncoder(destination: .httpBody))

// HTTP body: "qux[]=x&qux[]=y&qux[]=z&baz[]=a&baz[]=b&foo[]=bar"
```

#### 配置已编码参数的排序

从 Swift 4.2 开始，Swift 的 `Dictionary` 类型使用的随机算法在运行时产生一个随机的内部顺序，并且在应用程序的每次启动都是不同的。这可能会导致已编码参数更改顺序，这可能会影响缓存和其他行为。默认情况下，`URLEncodedFormEncoder` 将对其编码的键值对进行排序。虽然这会为所有 `Encodable` 类型生成常量输出，但它可能与该类型实现的实际编码顺序不匹配。您可以将 `alphabetizeKeyValuePairs` 设置为 `false` 以返回到实现的顺序，因此这将变成随机 `Dictionary` 顺序。

您可以创建自己的 `URLEncodedFormParameterEncoder`，在初始化时，可以在 `URLEncodedFormEncoder` 参数中设置 `alphabetizeKeyValuePairs` 的值：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(alphabetizeKeyValuePairs: false))
```

##### 配置 `Array` 参数的编码

由于没有关于如何对集合类型进行编码的规范，默认情况下，Alamofire 遵循以下约定：将 `[]` 附加到数组值的键（`foo[]=1&foo[]=2`），并附加由中括号包围的嵌套字典值的键（`foo[bar]=baz`）。

`URLEncodedFormEncoder.ArrayEncoding` 枚举提供了以下对数组参数进行编码的方法：

- `.brackets` - 为每个值在键后附加一组空的中括号。这是默认情况。
- `.noBrackets` - 不附加括号。key 按原样编码。

默认情况下，Alamofire 使用 `.brackets` 编码，其中 `foo = [1, 2]` 编码为 `foo[]=1&foo[]=2`。

使用 `.noBrackets` 编码，`foo = [1, 2]` 编码为 `foo=1&foo=2`。

您可以创建自己的 `URLEncodedFormParameterEncoder`，在初始化时，可以在 `URLEncodedFormEncoder` 参数中设置 `arrayEncoding` 的值：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(arrayEncoding: .noBrackets))
```

##### 配置 `Bool` 参数的编码

`URLEncodedFormEncoder.BoolEncoding` 枚举提供了以下用于编码 `Bool` 参数的方法：

- `.numeric` - 把 `true` 编码为 `1`， `false` 编码为 `0`。这是默认情况。
- `.literal` - 把 `true` 和 `false` 编码为字符串文本。

默认情况下，Alamofire 使用 `.numeric`。

您可以创建自己的 `URLEncodedFormParameterEncoder`，在初始化时，可以在 `URLEncodedFormEncoder` 参数中设置 `boolEncoding` 的值：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(boolEncoding: .numeric))
```

##### 配置 `Data` 参数的编码

`DataEncoding` 包括以下用于编码 `Data` 参数的方法：

- `.deferredToData` - 使用 `Data` 的自带 `Encodable` 支持。
- `.base64` - 将 `Data` 编码为 base64 编码的字符串。这是默认情况。
- `.custom((Data) -> throws -> String)` - 使用给定的闭包对 `Data` 进行编码。

您可以创建自己的 `URLEncodedFormParameterEncoder`，在初始化时，可以在 `URLEncodedFormEncoder` 参数中设置 `dataEncoding` 的值：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(dataEncoding: .base64))
```

##### 配置 `Date` 参数的编码

鉴于将 `Date` 编码为 `String` 的方法非常多，`DateEncoding` 包括以下用于编码 `Date` 参数的方法：

- `.deferredToDate` - 使用 `Date` 的自带 `Encodable` 支持。这是默认情况。
- `.secondsSince1970` - 将 `Date` 编码为 1970 年 1 月 1 日 UTC 零点的秒数。
- `.millisecondsSince1970` - 将 `Date` 编码为 1970 年 1 月 1 日 UTC 零点的毫秒数。
- `.iso8601` - 根据 ISO 8601 和 RFC3339 标准对 `Date` 进行编码。
- `.formatted(DateFormatter)` - 使用给定的 `DateFormatter` 对 `Date` 进行编码。
- `.custom((Date) throws -> String)` - 使用给定的闭包对 `Date` 进行编码。

您可以创建自己的 `URLEncodedFormParameterEncoder`，在初始化时，可以在 `URLEncodedFormEncoder` 参数中设置 `dateEncoding` 的值：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(dateEncoding: .iso8601))
```

##### 配置 Coding Keys 的编码

由于 key 参数样式的多样性，`KeyEncoding` 提供了以下方法来从 `lowerCamelCase` 中自定义 key 编码：

- `.useDefaultKeys` - 使用每种类型指定的 key。这是默认情况。
- `.convertToSnakeCase` - 将 key 转换为 snake case：`oneTwoThree` 变成 `one_two_three`。
- `.convertToKebabCase` - 将 key 转换为 kebab case：`oneTwoThree` 变成 `one-two-three`。
- `.capitalized` - 将第一个字母大写，例如 `oneTwoThree` 变为 `OneTwoThree`。
- `.uppercased` - 所有字母大写：`oneTwoThree` 变成 `ONETWOTHREE`。
- `.lowercased` - 所有字母小写：`oneTwoThree` 变成 `onetwothree`。
- `.custom((String) -> String)` - 使用给定的闭包对 key 进行编码。

您可以创建自己的 `URLEncodedFormParameterEncoder`，在初始化时，可以在 `URLEncodedFormEncoder` 参数中设置 `keyEncoding` 的值：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(keyEncoding: .convertToSnakeCase))
```

##### 配置空格的编码

旧的表单编码器使用 `+` 来对空格进行编码，而一些服务器仍然希望使用这种编码，而不是现代的百分比编码，因此 Alamofire 包含以下对空格进行编码的方法：

`.percentEscaped` - 通过应用标准百分比转义对空格字符进行编码。`" "` 编码为`"%20"`。这是默认情况。
`.plusReplaced` - 将空格字符替换为 `+` ，`" "` 编码为`"+"`。

您可以创建自己的 `URLEncodedFormParameterEncoder`，在初始化时，可以在 `URLEncodedFormEncoder` 参数中设置 `spaceEncoding` 的值：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(spaceEncoding: .plusReplaced))
```

#### `JSONParameterEncoder`

`JSONParameterEncoder` 使用 Swift 的 `JSONEncoder` 对 `Encodable` 值进行编码，并将结果设置为 `URLRequest` 的 `httpBody`。如果 `Content-Type` 尚未设置，则将其设置为 `application/json`。

##### JSON 编码参数的 POST 请求

```swift
let parameters: [String: [String]] = [
    "foo": ["bar"],
    "baz": ["a", "b"],
    "qux": ["x", "y", "z"]
]

AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: JSONParameterEncoder.default)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: JSONParameterEncoder.prettyPrinted)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: JSONParameterEncoder.sortedKeys)

// HTTP body: {"baz":["a","b"],"foo":["bar"],"qux":["x","y","z"]}
```

##### 自定义 `JSONEncoder`

您可以自定义 `JSONParameterEncoder` 的行为，方法是将自定义的 `JSONEncoder` 实例传递给它：

```swift
let encoder = JSONEncoder()
encoder.dateEncoding = .iso8601
encoder.keyEncodingStrategy = .convertToSnakeCase
let parameterEncoder = JSONParameterEncoder(encoder: encoder)
```

##### 手动对 `URLRequest` 进行参数编码

`ParameterEncoder` APIs 也可以在 Alamofire 之外使用，方法是直接在`URLRequest` 中编码参数。

```swift
let url = URL(string: "https://httpbin.org/get")!
var urlRequest = URLRequest(url: url)

let parameters = ["foo": "bar"]
let encodedURLRequest = try URLEncodedFormParameterEncoder.default.encode(parameters, into: urlRequest)
```

### HTTP Headers

Alamofire 包含自己的 `HTTPHeaders` 类型，这是一种顺序保持且不区分大小写的 HTTP header name/value 对的表示。`HTTPHeader` 类型封装单个 name/value 对，并为常用的 headers 提供各种静态值。

向 `Request` 添加自定义 `HTTPHeaders` 就像向 `request` 方法传递值一样简单：

```swift
let headers: HTTPHeaders = [
    "Authorization": "Basic VXNlcm5hbWU6UGFzc3dvcmQ=",
    "Accept": "application/json"
]

AF.request("https://httpbin.org/headers", headers: headers).responseJSON { response in
    debugPrint(response)
}
```

`HTTPHeaders` 也可以由 `HTTPHeader` 数组构造：

```swift
let headers: HTTPHeaders = [
    .authorization(username: "Username", password: "Password"),
    .accept("application/json")
]

AF.request("https://httpbin.org/headers", headers: headers).responseJSON { response in
    debugPrint(response)
}
```

> 对于不会变的 HTTP headers，建议在 `URLSessionConfiguration` 上设置它们，以便让它们自动应用于底层 `URLSession` 创建的任何 `URLSessionTask`。

默认的 Alamofire `Session` 为每个 `Request` 提供一组默认的 headers。其中包括：

- `Accept-Encoding`，默认为 `br;q=1.0, gzip;q=0.8, deflate;q=0.6`，根据 [`RFC 7230 §4.2.3`](https://tools.ietf.org/html/rfc7230#section-4.2.3)。
- `Accept-Language`，默认为系统中最多 6 种首选语言，格式为 `en;q=1.0`，根据 [`RFC 7231 §5.3.5`](https://tools.ietf.org/html/rfc7231#section-5.3.5)。
- `User-Agent`，其中包含有关当前应用程序的版本信息。例如：`iOS Example/1.0 (com.alamofire.iOS-Example; build:1; iOS 13.0.0) Alamofire/5.0.0`，根据 [`RFC 7231 §5.5.3`](https://tools.ietf.org/html/rfc7231#section-5.5.3)。

如果需要自定义这些 headers，则应创建自定义 `URLSessionConfiguration`，更新 `defaultHTTPHeaders` 属性，并将配置应用于新 `Session` 实例。使用`URLSessionConfiguration.af.default` 来自定义配置，会保留 Alamofire 的默认 headers。

### 响应验证

默认情况下，无论响应的内容如何，Alamofire 都会将任何已完成的请求视为成功。如果响应具有不可接受的状态代码或 MIME 类型，则在响应处理程序之前调用 `validate()` 将导致生成错误。

#### 自动验证

`validate()` API 自动验证状态代码是否在 `200..<300` 范围内，以及响应的 `Content-Type` header 是否与请求的 `Accept` 匹配（如果有提供）。

```swift
AF.request("https://httpbin.org/get").validate().responseJSON { response in
    debugPrint(response)
}
```

#### 手动验证

```swift
AF.request("https://httpbin.org/get")
    .validate(statusCode: 200..<300)
    .validate(contentType: ["application/json"])
    .responseData { response in
        switch response.result {
        case .success:
            print("Validation Successful")
        case let .failure(error):
            print(error)
        }
    }
```

### 响应处理

Alamofire 的 `DataRequest` 和 `DownloadRequest` 都有相应的响应类型：`DataResponse<Success, Failure: Error>` 和 `DownloadResponse<Success, Failure: Error>`。这两个类型都由两个泛型组成：序列化类型和错误类型。默认情况下，所有响应值都将生成 `AFError` 错误类型（`DataResponse<Success, AFError>`）。Alamofire 在其公共 API 中使用了更简单的 `AFDataResponse<Success>` 和 `AFDownloadResponse<Success>`，它们总是有 `AFError` 错误类型。`UploadRequest` 是 `DataRequest` 的一个子类，使用相同的 `DataResponse` 类型。

处理在 Alamofire 中发出的 `DataRequest` 或 `UploadRequest` 的 `DataResponse` 涉及到链接 response handler，例如 `responseJSON` 链接到 `DataRequest`:

```swift
AF.request("https://httpbin.org/get").responseJSON { response in
    debugPrint(response)
}
```

在上面的示例中，`responseJSON` handler 被添加到 `DataRequest` 中，以便在 `DataRequest` 完成后执行。传递给 handler 闭包的参数是从响应属性来的 `JSONResponseSerializer` 生成的 `AFDataResponse<Any>` 值。

此闭包并不阻塞执行以等待服务器的响应，而是作为回调添加，以便在收到响应后处理该响应。请求的结果仅在响应闭包的范围内可用。任何依赖于从服务器接收到的响应或数据的执行都必须在响应闭包中完成。

> Alamofire 的网络请求是异步完成的。异步编程可能会让不熟悉这个概念的程序员感到沮丧，但是有[很好的理由](https://developer.apple.com/library/ios/qa/qa1693/_index.html)这样做。

默认情况下，Alamofire 包含六个不同的数据响应 handlers，包括：

```swift

// Response Handler - 未序列化的 Response
func response(
    queue: DispatchQueue = .main,
    completionHandler: @escaping (AFDataResponse<Data?>) -> Void
) -> Self

// Response Serializer Handler - Serialize using the passed Serializer
func response<Serializer: DataResponseSerializerProtocol>(
    queue: DispatchQueue = .main,
    responseSerializer: Serializer,
    completionHandler: @escaping (AFDataResponse<Serializer.SerializedObject>) -> Void
) -> Self

// Response Data Handler - Serialized into Data
func responseData(
    queue: DispatchQueue = .main,
    completionHandler: @escaping (AFDataResponse<Data>) -> Void
) -> Self

// Response String Handler - Serialized into String
func responseString(
    queue: DispatchQueue = .main,
    encoding: String.Encoding? = nil,
    completionHandler: @escaping (AFDataResponse<String>) -> Void
) -> Self

// Response JSON Handler - Serialized into Any Using JSONSerialization
func responseJSON(
    queue: DispatchQueue = .main,
    options: JSONSerialization.ReadingOptions = .allowFragments,
    completionHandler: @escaping (AFDataResponse<Any>) -> Void
) -> Self

// Response Decodable Handler - Serialized into Decodable Type
func responseDecodable<T: Decodable>(
    of type: T.Type = T.self,
    queue: DispatchQueue = .main,
    decoder: DataDecoder = JSONDecoder(),
    completionHandler: @escaping (AFDataResponse<T>) -> Void
) -> Self
```

没有一个响应 handlers 对从服务器返回的 `HTTPURLResponse` 执行任何验证。

> 例如，`400..<500` 和 `500..<600` 范围内的响应状态代码不会自动触发错误。Alamofire 使用[响应验证]()链接来实现这一点。

#### 响应 Handler

响应 handler 不计算任何响应数据。它只是直接从 `URLSessionDelegate` 转发所有信息。它相当于使用 `cURL` 执行请求。

```swift
AF.request("https://httpbin.org/get").response { response in
    debugPrint("Response: \(response)")
}
```

我们强烈建议您利用 `Response` 和 `Result` 类型来利用其他响应序列化器。

#### 响应 Data Handler

`responseData` handler 使用 `DataResponseSerializer` 提取并验证服务器返回的数据。如果没有发生错误并且返回数据，则响应结果将为 `.success`， `value` 将为从服务器返回的 `Data`。

```swift
AF.request("https://httpbin.org/get").responseData { response in
    debugPrint("Response: \(response)")
}
```

#### 响应 String Handler

`responseString` handler 使用 `StringResponseSerializer` 将服务器返回的数据转换为具有指定编码的 `String`。如果没有发生错误，并且服务器数据成功序列化为 `String`，则响应结果将为 `.success`，并且值的类型为 `String`。

```swift
AF.request("https://httpbin.org/get").responseString { response in
    debugPrint("Response: \(response)")
}
```

> 如果未指定编码，Alamofire 将使用服务器 `HTTPURLResponse` 中指定的文本编码。如果服务器响应无法确定文本编码，则默认为 `.isoLatin1`。

#### 响应 JSON Handler

`responseJSON` handler 使用 `JSONResponseSerializer` 使用指定的 `JSONSerialization.ReadingOptions` 将服务器返回的数据转换为 `Any` 类型。如果没有出现错误，并且服务器数据成功序列化为 JSON 对象，则响应 `AFResult` 将为 `.success`，值将为 `Any` 类型。

```swift
AF.request("https://httpbin.org/get").responseJSON { response in
    debugPrint("Response: \(response)")
}
```

#### 响应 `Decodable` Handler

`responseDecodable` handler 使用 `DecodableResponseSerializer` 和 指定的 `DataDecoder`（`Decoder` 的协议抽象，可以从 `Data` 解码）将服务器返回的数据转换为传递进来的 `Decodable` 类型。如果没有发生错误，并且服务器数据已成功解码为 `Decodable` 类型，则响应 `Result` 将为 `.success`，并且 `value` 将为传递进来的类型。

```swift
struct HTTPBinResponse: Decodable {
    let url: String
}

AF.request("https://httpbin.org/get").responseDecodable(of: HTTPBinResponse.self) { response in
    debugPrint("Response: \(response)")
}
```

#### 链式响应 handlers

响应 handlers 还可以连接起来：

```swift
Alamofire.request("https://httpbin.org/get")
    .responseString { response in
        print("Response String: \(response.value)")
    }
    .responseJSON { response in
        print("Response JSON: \(response.value)")
    }
```

需要注意的是，对同一请求使用多个响应 handlers 需要多次序列化服务器数据，每个响应 handlers 处理一次。作为最佳实践，通常应避免对同一请求使用多个响应 handlers，特别是在生产环境中。它们只能用于调试或在没有更好选择的情况下使用。

#### 响应 Handler 队列

默认情况下，传递给响应 handler 的闭包在 `.main` 队列上执行，但可以传递一个指定的 DispatchQueue 来执行闭包。实际的序列化工作（将 `Data` 转换为其他类型）总是在后台队列上执行。

```swift
let utilityQueue = DispatchQueue.global(qos: .utility)

AF.request("https://httpbin.org/get").responseJSON(queue: utilityQueue) { response in
    print("Executed on utility queue.")
    debugPrint(response)
}
```

### 响应缓存

响应缓存使用系统自带的 `URLCache` 处理。它提供了内存和磁盘上的复合缓存，并允许您管理用于缓存的内存和磁盘的大小。

> 默认情况下，Alamofire 利用 `URLCache.shared` 实例。要自定义使用的`URLCache` 实例，请查看[高级用法]()。

### 身份验证

身份验证使用系统自带的 `URLCredential` 和 `URLAuthenticationChallenge` 处理。

> 这些身份验证 APIs 用于提示授权的服务器，而不是一般用于需要身份验证或等效的 header APIs。

**支持的身份验证方案**：

- [HTTP Basic](https://en.wikipedia.org/wiki/Basic_access_authentication)
- [HTTP Digest](https://en.wikipedia.org/wiki/Digest_access_authentication)
- [Kerberos](https://en.wikipedia.org/wiki/Kerberos_%28protocol%29)
- [NTLM](https://en.wikipedia.org/wiki/NT_LAN_Manager)

#### HTTP Basic 身份验证

`Request` 的 `authenticate` 方法将在使用 `URLAuthenticationChallenge` 进行质询时自动提供 `URLCredential`（如果适用）：

```swift
let user = "user"
let password = "password"

AF.request("https://httpbin.org/basic-auth/\(user)/\(password)")
    .authenticate(username: user, password: password)
    .responseJSON { response in
        debugPrint(response)
    }
```

#### 使用 `URLCredential` 进行验证

```swift
let user = "user"
let password = "password"

let credential = URLCredential(user: user, password: password, persistence: .forSession)

AF.request("https://httpbin.org/basic-auth/\(user)/\(password)")
    .authenticate(with: credential)
    .responseJSON { response in
        debugPrint(response)
    }
```

> 需要注意的是，当使用 `URLCredential` 进行身份验证时，如果服务器发出质询，底层 `URLSession` 实际上将发出两个请求。第一个请求将不包括“可能”触发服务器质询的 credential。然后，Alamofire 接收质询，追加 credential，并由底层 `URLSession` 重试请求。

#### 手动验证

如果您正在与始终需要 `Authenticate` 或类似 header 而不提示的 API 通信，则可以手动添加：

```swift
let user = "user"
let password = "password"

let headers: HTTPHeaders = [.authorization(username: user, password: password)]

AF.request("https://httpbin.org/basic-auth/user/password", headers: headers)
    .responseJSON { response in
        debugPrint(response)
    }
```

但是，必须是所有请求的一部分的 headers，通常作为自定义 `URLSessionConfiguration` 的一部分或通过使用 `RequestAdapter` 来更好地处理。具体请查看[高级用法]()。

### 下载数据到文件中

除了将数据提取到内存中之外，Alamofire 还提供了 `Session.download`、`DownloadRequest` 和 `DownloadResponse<Success，Failure:Error>` 以方便下载数据到磁盘。虽然下载到内存中对小负载（如大多数 JSON API 响应）非常有用，但获取更大的资源（如图像和视频）应下载到磁盘，以避免应用程序出现内存问题。

```swift
AF.download("https://httpbin.org/image/png").responseData { response in
    if let data = response.value {
        let image = UIImage(data: data)
    }
}
```

> `DownloadRequest` 具有与 `DataRequest` 相同的大多数响应 handlers。但是，由于它将数据下载到磁盘，因此序列化响应涉及从磁盘读取，还可能涉及将大量数据读入内存。在设计下载处理时，记住这些事实是很重要的。

#### 下载文件的存放位置

所有下载的数据最初都存储在系统临时目录中。它最终会在将来的某个时候被系统删除，所以如果它需要更长的寿命，将文件移到其他地方是很重要的。

您可以提供 `Destination` 闭包，将文件从临时目录移动到最终的存放位置。在临时文件实际移动到 `destinationURL` 之前，将执行闭包中指定的 `Options`。当前支持的两个 `Options` 是：

- `.createIntermediateDirectories` - 如果指定，则为目标 URL 创建中间目录。
- `.removePreviousFile` - 如果指定，则从目标 URL 中删除以前的文件。

```swift
let destination: DownloadRequest.Destination = { _, _ in
    let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
    let fileURL = documentsURL.appendingPathComponent("image.png")

    return (fileURL, [.removePreviousFile, .createIntermediateDirectories])
}

AF.download("https://httpbin.org/image/png", to: destination).response { response in
    debugPrint(response)

    if response.error == nil, let imagePath = response.fileURL?.path {
        let image = UIImage(contentsOfFile: imagePath)
    }
}
```

您还可以使用建议的 destination API：

```swift
let destination = DownloadRequest.suggestedDownloadDestination(for: .documentDirectory)

AF.download("https://httpbin.org/image/png", to: destination)
```

#### 下载进度

很多时候向用户报告下载进度是有帮助的。任何 `DownloadRequest` 都可以使用 `downloadProgress` 报告下载进度。

```swift
AF.download("https://httpbin.org/image/png")
    .downloadProgress { progress in
        print("Download Progress: \(progress.fractionCompleted)")
    }
    .responseData { response in
        if let data = response.value {
            let image = UIImage(data: data)
        }
    }
```

`URLSession` 的进度报告 APIs（也是 Alamofire 的）只有在服务器正确返回可用于计算进度的 `Content-Length` header 时才能工作。如果没有这个 header，进度将保持在 `0.0`，直到下载完成，此时进度将跳到 `1.0`。

`downloadProgress` API 还可以接收一个 `queue` 参数，该参数定义应该对哪个 `DispatchQueue` 调用下载进度闭包。

```swift
let utilityQueue = DispatchQueue.global(qos: .utility)

AF.download("https://httpbin.org/image/png")
    .downloadProgress(queue: utilityQueue) { progress in
        print("Download Progress: \(progress.fractionCompleted)")
    }
    .responseData { response in
        if let data = response.value {
            let image = UIImage(data: data)
        }
    }
```

#### 取消和恢复下载

除了所有请求类都有 `cancel()` API 外，`DownloadRequest` 还可以生成恢复数据，这些数据可以用于以后恢复下载。此 API 有两种形式：1）`cancel(producingResumeData: Bool)`，它允许控制是否生成恢复数据，但仅在 `DownloadResponse` 可用；2）`cancel(byProducingResumeData: (_ resumeData: Data?) -> Void)`，它执行相同的操作，但恢复数据在 completion handler 中可用。

如果 `DownloadRequest` 被取消或中断，则底层的 `URLSessionDownloadTask` 可能会生成恢复数据。如果发生这种情况，可以重新使用恢复数据来重新启动停止的 `DownloadRequest`。

> 重要提示：在所有 Apple 平台的某些版本（iOS 10 - 10.2、macOS 10.12 - 10.12.2、tvOS 10 - 10.1、watchOS 3 - 3.1.1）上，`resumeData` 在后台 `URLSessionConfiguration` 上被破坏。`resumeData` 生成逻辑中存在一个潜在的错误，即数据写入错误，并且总是无法恢复下载。有关此错误和可能的解决方法的详细信息，请参阅此 [Stack Overflow 的帖子](https://stackoverflow.com/a/39347461/1342462)。

```swift
var resumeData: Data!

let download = AF.download("https://httpbin.org/image/png").responseData { response in
    if let data = response.value {
        let image = UIImage(data: data)
    }
}

// download.cancel(producingResumeData: true) // Makes resumeData available in response only.
download.cancel { data in
    resumeData = data
}

AF.download(resumingWith: resumeData).responseData { response in
    if let data = response.value {
        let image = UIImage(data: data)
    }
}
```

### 上传数据到服务器

当使用 JSON 或 URL 编码的参数向服务器发送相对少量的数据时，`request()` 通常就足够了。如果需要从内存、文件 URL 或 InputStream 中的 `Data` 发送大量数据，那么 `upload()` 就是您想要使用的。

#### 上传 Data

```swift
let data = Data("data".utf8)

AF.upload(data, to: "https://httpbin.org/post").responseJSON { response in
    debugPrint(response)
}
```

#### 上传文件

```swift
let fileURL = Bundle.main.url(forResource: "video", withExtension: "mov")

AF.upload(fileURL, to: "https://httpbin.org/post").responseJSON { response in
    debugPrint(response)
}
```

#### 上传多表单数据

```swift
AF.upload(multipartFormData: { multipartFormData in
    multipartFormData.append(Data("one".utf8), withName: "one")
    multipartFormData.append(Data("two".utf8), withName: "two")
}, to: "https://httpbin.org/post")
    .responseJSON { response in
        debugPrint(response)
    }
```

#### 上传进度

当用户等待上传完成时，有时向用户显示上传的进度会很方便。任何 UploadRequest 都可以使用 `uploadProgress` 和 `downloadProgress` 报告响应数据下载的上传进度和下载进度。

```swift
let fileURL = Bundle.main.url(forResource: "video", withExtension: "mov")

AF.upload(fileURL, to: "https://httpbin.org/post")
    .uploadProgress { progress in
        print("Upload Progress: \(progress.fractionCompleted)")
    }
    .downloadProgress { progress in
        print("Download Progress: \(progress.fractionCompleted)")
    }
    .responseJSON { response in
        debugPrint(response)
    }
```

### 统计指标

#### `URLSessionTaskMetrics`

Alamofire 为每个 `Request` 收集了 `URLSessionTaskMetrics`。`URLSessionTaskMetrics` 封装了一些关于底层网络连接、请求和响应时间的奇妙统计信息。

```swift
AF.request("https://httpbin.org/get").responseJSON { response in
    print(response.metrics)
}
```

### cURL 的命令输出

调试平台问题可能令人沮丧。幸运的是，Alamofire 的 `Request` 类型可以生成等效的 cURL 命令，以便调试。由于 Alamofire `Request` 创建的异步性，这个 API 有同步和异步两个版本。要尽快获取 cURL 命令，可以将 `cURLDescription` 链接到请求上：

```swift
AF.request("https://httpbin.org/get")
    .cURLDescription { description in
        print(description)
    }
    .responseJSON { response in
        debugPrint(response.metrics)
    }
```

这将会生成：

```
$ curl -v \
-X GET \
-H "Accept-Language: en;q=1.0" \
-H "Accept-Encoding: br;q=1.0, gzip;q=0.9, deflate;q=0.8" \
-H "User-Agent: Demo/1.0 (com.demo.Demo; build:1; iOS 13.0.0) Alamofire/1.0" \
"https://httpbin.org/get"
```
