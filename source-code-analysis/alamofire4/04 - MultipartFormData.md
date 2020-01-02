多表单是什么回事？简单地说，多表单可以让我们一次性上传多个不同类型的文件给服务器。

`MultipartFormData`，这个类可以帮助我们创建多表单数据，然后通过请求体上传到服务器。目前有两种方式可以编码多表单数据：1）**直接在内存中编码**，非常高效，但是数据不能太大；2）**先把数据写到一个文件中**，然后再进行编码，适用于视频等文件。大文件一定要使用第二种方式，否则会造成内存不够用的。

### 一、辅助类型

#### 1. `EncodingCharacters`

`crlf`回车换行。

```swift
struct EncodingCharacters {
    static let crlf = "\r\n"
}
```

#### 2. `BoundaryGenerator`

因为多表单上传的是多种不同的数据，所以我们要在不同类型的文件之间添加一个边界。而这个边界生成器就是用来生成边界的。

```swift
struct BoundaryGenerator {
    // 边界类型，开始/内部/结束
    enum BoundaryType {
        case initial, encapsulated, final
    }

    // 随机边界，`arc4random()`返回`UInt32`，`%08x`表示将UInt32格式化成十六进制，
    // 并取前面8位，如果不够8位，用0代替
    static func randomBoundary() -> String {
        return String(format: "alamofire.boundary.%08x%08x", arc4random(), arc4random())
    }

    // 将String类型的边界，根据边界的类型转化成Data
    static func boundaryData(forBoundaryType boundaryType: BoundaryType, boundary: String) -> Data {
        let boundaryText: String

        switch boundaryType {
        case .initial:
            boundaryText = "--\(boundary)\(EncodingCharacters.crlf)"
        case .encapsulated:
            boundaryText = "\(EncodingCharacters.crlf)--\(boundary)\(EncodingCharacters.crlf)"
        case .final:
            boundaryText = "\(EncodingCharacters.crlf)--\(boundary)--\(EncodingCharacters.crlf)"
        }

        return boundaryText.data(using: String.Encoding.utf8, allowLossyConversion: false)!
    }
}
```

#### 3. `BodyPart`

表示多表单内部的一个表单：

```swift
class BodyPart {
    let headers: HTTPHeaders // [String: String]
    let bodyStream: InputStream // 数据输入流
    let bodyContentLength: UInt64 // 此表单的数据大小
    var hasInitialBoundary = false // 是否包含开始边界
    var hasFinalBoundary = false // 是否包含结束边界

    init(headers: HTTPHeaders, bodyStream: InputStream, bodyContentLength: UInt64) {
        self.headers = headers
        self.bodyStream = bodyStream
        self.bodyContentLength = bodyContentLength
    }
}
```

### 二、MultipartFormData类

#### 1. 属性

```swift
/// 包含边界的`Content-Type`
open lazy var contentType: String = "multipart/form-data; boundary=\(self.boundary)"

/// 整个多表单的数据大小，不包括边界
public var contentLength: UInt64 { return bodyParts.reduce(0) { $0 + $1.bodyContentLength } }

/// 多表单的边界
public let boundary: String

private var bodyParts: [BodyPart] // 整个多表单内部的表单数组
private var bodyPartError: AFError? // 表单拼接过程中可能出现的错误
private let streamBufferSize: Int // 流的缓存大小，默认是1024个字节，也就是1kb
```

这部分用到了一个高阶函数`reduce`，这是一个Swift内置的集合类型的函数，通过传入一个closure来合并这个集合。`bodyParts.reduce(0) { $0 + $1.bodyContentLength }`的意思是求`bodyParts`数组中每个bodyPart的`bodyContentLength`的和。他这个是一种简写的方式，其实完整的写法是：

```swift
public var contentLength: UInt64 {
    return bodyParts.reduce(0, { (sum, bodyPart) -> UInt64 in
        return sum + bodyPart.bodyContentLength
    })
}
```

上面这种方式就容易理解多了吧。不懂的话，我们再看一个简单的例子：

```swift
let numbers = [1, 2, 3, 4]
let numberSum = numbers.reduce(0, { sum, number in
    return sum + number
})
// numberSum == 10
```

上面这个例子是求`numbers`元素的和，本质上跟我们源码是一个意思。当`numbers.reduce`在运行的时候，最开始的求和结果是我们传入的`0`，然后用第一个元素`1`放到closure运行，这时`sum`是`0`，`number`就是第一个元素`1`，所以结果就是`0 + 1`；然后用第二个元素`2`放到closure运行，这时`sum`就是上次的运算结果`1`，`number`就是第二个元素`2`，结果就是`1 + 2`；以此类推，最后结果是`10`。

Swift还内置了很多类似这样的函数，例如`map`、`flatMap`、`filter`等等，大家可以自己去了解下。

#### 2. 初始化方法

我们看到这个`self.bodyParts = []`，`[]`数组初始化的简写方式，代码更简洁，而不需要写成这样`[BodyPart]()`。在Swift开发过程中，在不失去代码可读性的情况下，代码能简洁尽量简洁。关于代码风格问题，我们可以使用Swiftlint来帮助我们，可以看看我写的这篇文章[【iOS开发】Swift代码风格检查库 —— SwiftLint](http://www.jianshu.com/p/f872484fcd50)。

```swift
public init() {
    self.boundary = BoundaryGenerator.randomBoundary()
    self.bodyParts = []
    self.streamBufferSize = 1024
}
```

#### 3. Body Parts

```swift

// MARK: - 这三个方法可以让我们通过传入数据类型，然后通过流的形式把数据拼接到多表单；
// 这三个方法大同小异，只是请求头不同

public func append(_ data: Data, withName name: String) {
    let headers = contentHeaders(withName: name)
    let stream = InputStream(data: data)
    let length = UInt64(data.count)

    append(stream, withLength: length, headers: headers)
}

public func append(_ data: Data, withName name: String, mimeType: String) {
    let headers = contentHeaders(withName: name, mimeType: mimeType)
    let stream = InputStream(data: data)
    let length = UInt64(data.count)

    append(stream, withLength: length, headers: headers)
}

public func append(_ data: Data, withName name: String, fileName: String, mimeType: String) {
    let headers = contentHeaders(withName: name, fileName: fileName, mimeType: mimeType)
    let stream = InputStream(data: data)
    let length = UInt64(data.count)

    append(stream, withLength: length, headers: headers)
}

// MARK: - 这两个方法可以让我们传文件的url，然后通过流的形式把文件拼接到多表单中

public func append(_ fileURL: URL, withName name: String) {
    let fileName = fileURL.lastPathComponent
    let pathExtension = fileURL.pathExtension

    if !fileName.isEmpty && !pathExtension.isEmpty {
        let mime = mimeType(forPathExtension: pathExtension)
        append(fileURL, withName: name, fileName: fileName, mimeType: mime)
    } else {
        setBodyPartError(withReason: .bodyPartFilenameInvalid(in: fileURL))
    }
}

public func append(_ fileURL: URL, withName name: String, fileName: String, mimeType: String) {
    // 其实这个headers，不应该放在这里定义，而是放在这个方法里最后一行代码上面；
    // 因为这个headers只有在最后一行代码那里才被用到，我们应该遵循一个原则：用到的时候才创建。
    let headers = contentHeaders(withName: name, fileName: fileName, mimeType: mimeType)

    //============================================================
    //                 检查 1 - 是否是文件路径？
    //============================================================

    guard fileURL.isFileURL else {
        setBodyPartError(withReason: .bodyPartURLInvalid(url: fileURL))
        return
    }

    //============================================================
    //                检查 2 - 文件路径能否访问?
    //============================================================

    do {
        let isReachable = try fileURL.checkPromisedItemIsReachable()
        guard isReachable else {
            setBodyPartError(withReason: .bodyPartFileNotReachable(at: fileURL))
            return
        }
    } catch {
        setBodyPartError(withReason: .bodyPartFileNotReachableWithError(atURL: fileURL, error: error))
        return
    }

    //============================================================
    //             检查 3 - 文件路径是否是一个文件夹？
    //============================================================

    var isDirectory: ObjCBool = false
    let path = fileURL.path

    guard FileManager.default.fileExists(atPath: path, isDirectory: &isDirectory) && !isDirectory.boolValue else {
        setBodyPartError(withReason: .bodyPartFileIsDirectory(at: fileURL))
        return
    }

    //============================================================
    //               检查 4 - 是否能获得文件的大小?
    //============================================================

    let bodyContentLength: UInt64

    do {
        guard let fileSize = try FileManager.default.attributesOfItem(atPath: path)[.size] as? NSNumber else {
            setBodyPartError(withReason: .bodyPartFileSizeNotAvailable(at: fileURL))
            return
        }

        bodyContentLength = fileSize.uint64Value
    }
    catch {
        setBodyPartError(withReason: .bodyPartFileSizeQueryFailedWithError(forURL: fileURL, error: error))
        return
    }

    //============================================================
    //          检查 5 - 是否能通过文件路径创建一个流?
    //============================================================

    guard let stream = InputStream(url: fileURL) else {
        setBodyPartError(withReason: .bodyPartInputStreamCreationFailed(for: fileURL))
        return
    }

    append(stream, withLength: bodyContentLength, headers: headers)
}


// MARK: - 这两个方法可以让我们传一个流，然后拼接到多表单中

public func append(
    _ stream: InputStream,
    withLength length: UInt64,
    name: String,
    fileName: String,
    mimeType: String)
{
    let headers = contentHeaders(withName: name, fileName: fileName, mimeType: mimeType)
    append(stream, withLength: length, headers: headers)
}

public func append(_ stream: InputStream, withLength length: UInt64, headers: HTTPHeaders) {
    let bodyPart = BodyPart(headers: headers, bodyStream: stream, bodyContentLength: length)
    bodyParts.append(bodyPart)
}

// MARK: - 数据编码

// 一次性把所有body parts编码到一个Data中，只适用于小文件；
// 对于大文件，应该使用 `writeEncodedData(to:)`
public func encode() throws -> Data {
}

// 把数据写到一个文件中，适用于大文件
public func writeEncodedData(to fileURL: URL) throws {
    if let bodyPartError = bodyPartError {
        throw bodyPartError
    }

    if FileManager.default.fileExists(atPath: fileURL.path) {
        throw AFError.multipartEncodingFailed(reason: .outputStreamFileAlreadyExists(at: fileURL))
    } else if !fileURL.isFileURL {
        throw AFError.multipartEncodingFailed(reason: .outputStreamURLInvalid(url: fileURL))
    }

    guard let outputStream = OutputStream(url: fileURL, append: false) else {
        throw AFError.multipartEncodingFailed(reason: .outputStreamCreationFailed(for: fileURL))
    }

    // 打开了一个流，用完之后必须关闭
    // 很好地利用了Swift的defer特性，把打开和关闭写在一起
    outputStream.open()
    defer { outputStream.close() }

    self.bodyParts.first?.hasInitialBoundary = true
    self.bodyParts.last?.hasFinalBoundary = true

    for bodyPart in self.bodyParts {
        try write(bodyPart, to: outputStream)
    }
}

// MARK: - 私有的Body Part相关方法（不对外暴露的方法，我们应该加上private）

// 编码Body Part
private func encode(_ bodyPart: BodyPart) throws -> Data {
    var encoded = Data()

    let initialData = bodyPart.hasInitialBoundary ? initialBoundaryData() : encapsulatedBoundaryData()
    encoded.append(initialData)

    let headerData = encodeHeaders(for: bodyPart)
    encoded.append(headerData)

    let bodyStreamData = try encodeBodyStream(for: bodyPart)
    encoded.append(bodyStreamData)

    if bodyPart.hasFinalBoundary {
        encoded.append(finalBoundaryData())
    }

    return encoded
}

// 编码Header
private func encodeHeaders(for bodyPart: BodyPart) -> Data {
    var headerText = ""

    for (key, value) in bodyPart.headers {
        headerText += "\(key): \(value)\(EncodingCharacters.crlf)"
    }
    headerText += EncodingCharacters.crlf

    return headerText.data(using: String.Encoding.utf8, allowLossyConversion: false)!
}

// 编码body stream
private func encodeBodyStream(for bodyPart: BodyPart) throws -> Data {
    let inputStream = bodyPart.bodyStream
    inputStream.open()
    defer { inputStream.close() }

    var encoded = Data()

    while inputStream.hasBytesAvailable {
        var buffer = [UInt8](repeating: 0, count: streamBufferSize)
        let bytesRead = inputStream.read(&buffer, maxLength: streamBufferSize)

        if let error = inputStream.streamError {
            throw AFError.multipartEncodingFailed(reason: .inputStreamReadFailed(error: error))
        }

        if bytesRead > 0 {
            encoded.append(buffer, count: bytesRead)
        } else {
            break
        }
    }

    return encoded
}

// MARK: - 把Body Part写入到OutputStream的相关方法

private func write(_ bodyPart: BodyPart, to outputStream: OutputStream) throws {
    try writeInitialBoundaryData(for: bodyPart, to: outputStream)
    try writeHeaderData(for: bodyPart, to: outputStream)
    try writeBodyStream(for: bodyPart, to: outputStream)
    try writeFinalBoundaryData(for: bodyPart, to: outputStream)
}

private func writeInitialBoundaryData(for bodyPart: BodyPart, to outputStream: OutputStream) throws {
    let initialData = bodyPart.hasInitialBoundary ? initialBoundaryData() : encapsulatedBoundaryData()
    return try write(initialData, to: outputStream)
}

private func writeHeaderData(for bodyPart: BodyPart, to outputStream: OutputStream) throws {
    let headerData = encodeHeaders(for: bodyPart)
    return try write(headerData, to: outputStream)
}

private func writeBodyStream(for bodyPart: BodyPart, to outputStream: OutputStream) throws {
    let inputStream = bodyPart.bodyStream

    inputStream.open()
    defer { inputStream.close() }

    while inputStream.hasBytesAvailable {
        var buffer = [UInt8](repeating: 0, count: streamBufferSize)
        let bytesRead = inputStream.read(&buffer, maxLength: streamBufferSize)

        if let streamError = inputStream.streamError {
            throw AFError.multipartEncodingFailed(reason: .inputStreamReadFailed(error: streamError))
        }

        if bytesRead > 0 {
            if buffer.count != bytesRead {
                buffer = Array(buffer[0..<bytesRead])
            }

            try write(&buffer, to: outputStream)
        } else {
            break
        }
    }
}

private func writeFinalBoundaryData(for bodyPart: BodyPart, to outputStream: OutputStream) throws {
    if bodyPart.hasFinalBoundary {
        return try write(finalBoundaryData(), to: outputStream)
    }
}

// MARK: - 把已经缓存的数据写入到OutputStream的相关方法

private func write(_ data: Data, to outputStream: OutputStream) throws {
    var buffer = [UInt8](repeating: 0, count: data.count)
    data.copyBytes(to: &buffer, count: data.count)

    return try write(&buffer, to: outputStream)
}

private func write(_ buffer: inout [UInt8], to outputStream: OutputStream) throws {
    var bytesToWrite = buffer.count

    while bytesToWrite > 0, outputStream.hasSpaceAvailable {
        let bytesWritten = outputStream.write(buffer, maxLength: bytesToWrite)

        if let error = outputStream.streamError {
            throw AFError.multipartEncodingFailed(reason: .outputStreamWriteFailed(error: error))
        }

        bytesToWrite -= bytesWritten

        if bytesToWrite > 0 {
            buffer = Array(buffer[bytesWritten..<buffer.count])
        }
    }
}

// MARK: - 根据文件的扩展名获取Mime Type

private func mimeType(forPathExtension pathExtension: String) -> String {
    if
        let id = UTTypeCreatePreferredIdentifierForTag(kUTTagClassFilenameExtension, pathExtension as CFString, nil)?.takeRetainedValue(),
        let contentType = UTTypeCopyPreferredTagWithClass(id, kUTTagClassMIMEType)?.takeRetainedValue()
    {
        return contentType as String
    }

    return "application/octet-stream"
}

// MARK: - 生成Content Headers

private func contentHeaders(withName name: String, fileName: String? = nil, mimeType: String? = nil) -> [String: String] {
    var disposition = "form-data; name=\"\(name)\""
    if let fileName = fileName { disposition += "; filename=\"\(fileName)\"" }

    var headers = ["Content-Disposition": disposition]
    if let mimeType = mimeType { headers["Content-Type"] = mimeType }

    return headers
}

// MARK: - 把边界编码为数据

private func initialBoundaryData() -> Data {
    return BoundaryGenerator.boundaryData(forBoundaryType: .initial, boundary: boundary)
}

private func encapsulatedBoundaryData() -> Data {
    return BoundaryGenerator.boundaryData(forBoundaryType: .encapsulated, boundary: boundary)
}

private func finalBoundaryData() -> Data {
    return BoundaryGenerator.boundaryData(forBoundaryType: .final, boundary: boundary)
}

// MARK: - 设置bodyPartError

private func setBodyPartError(withReason reason: AFError.MultipartEncodingFailureReason) {
    guard bodyPartError == nil else { return }
    bodyPartError = AFError.multipartEncodingFailed(reason: reason)
}
```

在这一部分中，重点提一下Swift的`defer`特性。对于某些操作，需要放到一个作用域内最后面、并且必须要执行的代码，我们可以用`defer`。例如上面的例子，打开了一个流，用完之后必须关闭，所以用`defer`把关闭代码封装起来，并且跟打开操作放在一起，这样可以很有效的减少漏写的几率。不管后面的代码发生了什么，`defer`里面的代码总会在最后一行代码执行完成后执行。
