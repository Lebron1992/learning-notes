这里面主要写了如何验证请求是否有效。

### 1. 验证`Request`

```swift
extension Request {

    // 为了代码简洁，把ResponseValidationFailureReason定义为ErrorReason
    fileprivate typealias ErrorReason = AFError.ResponseValidationFailureReason

    // 用于表示验证结果
    public enum ValidationResult {
        case success
        case failure(Error)
    }

    // MIMEType类型
    fileprivate struct MIMEType {
        let type: String
        let subtype: String

        // 是否是通配符
        var isWildcard: Bool { return type == "*" && subtype == "*" }

        init?(_ string: String) {
            let components: [String] = {
                let stripped = string.trimmingCharacters(in: .whitespacesAndNewlines)

                #if swift(>=3.2)
                    let split = stripped[..<(stripped.range(of: ";")?.lowerBound ?? stripped.endIndex)]
                #else
                    let split = stripped.substring(to: stripped.range(of: ";")?.lowerBound ?? stripped.endIndex)
                #endif

                return split.components(separatedBy: "/")
            }()

            if let type = components.first, let subtype = components.last {
                self.type = type
                self.subtype = subtype
            } else {
                return nil
            }
        }

        // 判断与另外一个给定的MIMEType是否匹配
        func matches(_ mime: MIMEType) -> Bool {
            switch (type, subtype) {
            case (mime.type, mime.subtype), (mime.type, "*"), ("*", mime.subtype), ("*", "*"):
                return true
            default:
                return false
            }
        }
    }
    
    // 请求成功的状态码
    fileprivate var acceptableStatusCodes: [Int] { return Array(200..<300) }

    // 接受的ContentType类型
    fileprivate var acceptableContentTypes: [String] {
        if let accept = request?.value(forHTTPHeaderField: "Accept") {
            return accept.components(separatedBy: ",")
        }

        return ["*/*"]
    }

    // 验证响应的状态码是否匹配给定的状态码
    fileprivate func validate<S: Sequence>(
        statusCode acceptableStatusCodes: S,
        response: HTTPURLResponse)
        -> ValidationResult
        where S.Iterator.Element == Int
    {
        if acceptableStatusCodes.contains(response.statusCode) {
            return .success
        } else {
            let reason: ErrorReason = .unacceptableStatusCode(code: response.statusCode)
            return .failure(AFError.responseValidationFailed(reason: reason))
        }
    }

    // 验证响应的ContentType是否匹配给定ContentTypes
    fileprivate func validate<S: Sequence>(
        contentType acceptableContentTypes: S,
        response: HTTPURLResponse,
        data: Data?)
        -> ValidationResult
        where S.Iterator.Element == String
    {
        // 如果有数据，说明请求成功了，直接返回
        guard let data = data, data.count > 0 else { return .success }

        guard
            let responseContentType = response.mimeType,
            let responseMIMEType = MIMEType(responseContentType)
            else { // 响应中没有mimeType
            
                // 如果给定的ContentTypes是通配符，则验证成功
                for contentType in acceptableContentTypes {
                    if let mimeType = MIMEType(contentType), mimeType.isWildcard {
                        return .success
                    }
                }

                let error: AFError = {
                    let reason: ErrorReason = .missingContentType(acceptableContentTypes: Array(acceptableContentTypes))
                    return AFError.responseValidationFailed(reason: reason)
                }()

                return .failure(error)
        }
    
        // 如果响应的contentType匹配给定的ContentTypes，则验证成功
        for contentType in acceptableContentTypes {
            if let acceptableMIMEType = MIMEType(contentType), acceptableMIMEType.matches(responseMIMEType) {
                return .success
            }
        }

        let error: AFError = {
            let reason: ErrorReason = .unacceptableContentType(
                acceptableContentTypes: Array(acceptableContentTypes),
                responseContentType: responseContentType
            )

            return AFError.responseValidationFailed(reason: reason)
        }()

        return .failure(error)
    }
}
```

### 2. 验证`DataRequest`

```swift
extension DataRequest {
    // 验证请求是否有效的closure
    public typealias Validation = (URLRequest?, HTTPURLResponse, Data?) -> ValidationResult

    // 使用给定的Validation验证
    @discardableResult
    public func validate(_ validation: @escaping Validation) -> Self {
        let validationExecution: () -> Void = { [unowned self] in
            if
                let response = self.response,
                self.delegate.error == nil,
                case let .failure(error) = validation(self.request, response, self.delegate.data)
            {
                self.delegate.error = error
            }
        }
        
        // 把验证添加到一个数组中，待请求完成后执行所有验证
        validations.append(validationExecution)

        return self
    }

    // 验证响应状态码是否与给定的状态码匹配
    @discardableResult
    public func validate<S: Sequence>(statusCode acceptableStatusCodes: S) -> Self where S.Iterator.Element == Int {
        return validate { [unowned self] _, response, _ in
            // 直接调用父类的验证方法
            return self.validate(statusCode: acceptableStatusCodes, response: response)
        }
    }

    // 验证响应的contentType是否与给定的contentTypes匹配
    @discardableResult
    public func validate<S: Sequence>(contentType acceptableContentTypes: S) -> Self where S.Iterator.Element == String {
        return validate { [unowned self] _, response, data in
            // 直接调用父类的验证方法
            return self.validate(contentType: acceptableContentTypes, response: response, data: data)
        }
    }

    // 默认的验证方法：
    // 1）验证状态码是否在200...299区间；2）验证contentType是否与请求头中给定的"Accept"相同
    @discardableResult
    public func validate() -> Self {
        return validate(statusCode: self.acceptableStatusCodes).validate(contentType: self.acceptableContentTypes)
    }
}
```

### 2. 验证`DownloadRequest`

这里基本上跟验证`DataRequest`的一样。

```swift
extension DownloadRequest {
    // 验证请求是否有效的closure
    public typealias Validation = (
        _ request: URLRequest?,
        _ response: HTTPURLResponse,
        _ temporaryURL: URL?,
        _ destinationURL: URL?)
        -> ValidationResult

    // 使用给定的Validation验证
    @discardableResult
    public func validate(_ validation: @escaping Validation) -> Self {
        let validationExecution: () -> Void = { [unowned self] in
            let request = self.request
            let temporaryURL = self.downloadDelegate.temporaryURL
            let destinationURL = self.downloadDelegate.destinationURL

            if
                let response = self.response,
                self.delegate.error == nil,
                case let .failure(error) = validation(request, response, temporaryURL, destinationURL)
            {
                self.delegate.error = error
            }
        }
        
        // 把验证添加到一个数组中，待请求完成后执行所有验证
        validations.append(validationExecution)

        return self
    }

    // 验证响应状态码是否与给定的状态码匹配
    @discardableResult
    public func validate<S: Sequence>(statusCode acceptableStatusCodes: S) -> Self where S.Iterator.Element == Int {
        return validate { [unowned self] _, response, _, _ in
            // 直接调用父类的验证方法
            return self.validate(statusCode: acceptableStatusCodes, response: response)
        }
    }

    // 验证响应的contentType是否与给定的contentTypes匹配
    @discardableResult
    public func validate<S: Sequence>(contentType acceptableContentTypes: S) -> Self where S.Iterator.Element == String {
        return validate { [unowned self] _, response, _, _ in
            let fileURL = self.downloadDelegate.fileURL

            guard let validFileURL = fileURL else {
                return .failure(AFError.responseValidationFailed(reason: .dataFileNil))
            }

            do {
                let data = try Data(contentsOf: validFileURL)
                // 直接调用父类的验证方法
                return self.validate(contentType: acceptableContentTypes, response: response, data: data)
            } catch {
                return .failure(AFError.responseValidationFailed(reason: .dataFileReadFailed(at: validFileURL)))
            }
        }
    }

    // 默认的验证方法：
    // 1）验证状态码是否在200...299区间；2）验证contentType是否与请求头中给定的"Accept"相同
    @discardableResult
    public func validate() -> Self {
        return validate(statusCode: self.acceptableStatusCodes).validate(contentType: self.acceptableContentTypes)
    }
}
```
