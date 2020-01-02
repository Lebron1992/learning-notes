`Result`是一个泛型枚举，用于表示请求成功或者失败。如果是success，携带一个泛型Value；如果是failure，携带一个Error。

```swift
public enum Result<Value> {
    case success(Value)
    case failure(Error)

    // 请求是否成功
    public var isSuccess: Bool {
        switch self {
        case .success:
            return true
        case .failure:
            return false
        }
    }

    // 请求是否成功
    public var isFailure: Bool {
        return !isSuccess
    }

    // 请求成功对应的结果
    public var value: Value? {
        switch self {
        case .success(let value):
            return value
        case .failure:
            return nil
        }
    }

    // 请求失败对应的error
    public var error: Error? {
        switch self {
        case .success:
            return nil
        case .failure(let error):
            return error
        }
    }
}
```

另外还实现了`CustomStringConvertible`和`CustomDebugStringConvertible`，方便调试。

### 其他API

```swift
// 创建一个Result，使用案例：
//
//     func someString() throws -> String { ... }
//
//     let result = Result(value: {
//         return try someString()
//     })
//
//     // result的类型是Result<String>
//
//     // 还可以简写成:
//
//     let result = Result { try someString() }
public init(value: () throws -> Value) {
    do {
        self = try .success(value())
    } catch {
        self = .failure(error)
    }
}

// 把Result的value解压出来，使用案例：
//
//     let possibleString: Result<String> = .success("success")
//     try print(possibleString.unwrap())
//     // Prints "success"
//
//     let noString: Result<String> = .failure(error)
//     try print(noString.unwrap())
//     // Throws error
public func unwrap() throws -> Value {
    switch self {
    case .success(let value):
        return value
    case .failure(let error):
        throw error
    }
}

// result是success的时候执行transform，使用案例：
//
//     let possibleData: Result<Data> = .success(Data())
//     let possibleInt = possibleData.map { $0.count }
//     try print(possibleInt.unwrap())
//     // Prints "0"
//
//     let noData: Result<Data> = .failure(error)
//     let noInt = noData.map { $0.count }
//     try print(noInt.unwrap())
//     // Throws error
public func map<T>(_ transform: (Value) -> T) -> Result<T> {
    switch self {
    case .success(let value):
        return .success(transform(value))
    case .failure(let error):
        return .failure(error)
    }
}

// result是success的时候执行transform，使用案例：
//
//     let possibleData: Result<Data> = .success(Data(...))
//     let possibleObject = possibleData.flatMap {
//         try JSONSerialization.jsonObject(with: $0)
//     }
public func flatMap<T>(_ transform: (Value) throws -> T) -> Result<T> {
    switch self {
    case .success(let value):
        do {
            return try .success(transform(value))
        } catch {
            return .failure(error)
        }
    case .failure(let error):
        return .failure(error)
    }
}

// result是failure的时候执行transform，使用案例：
//
//     let possibleData: Result<Data> = .failure(someError)
//     let withMyError: Result<Data> = possibleData.mapError { MyError.error($0) }
public func mapError<T: Error>(_ transform: (Error) -> T) -> Result {
    switch self {
    case .failure(let error):
        return .failure(transform(error))
    case .success:
        return self
    }
}

// result是failure的时候执行transform，使用案例：
//
//     let possibleData: Result<Data> = .success(Data(...))
//     let possibleObject = possibleData.flatMapError {
//         try someFailableFunction(taking: $0)
//     }
public func flatMapError<T: Error>(_ transform: (Error) throws -> T) -> Result {
    switch self {
    case .failure(let error):
        do {
            return try .failure(transform(error))
        } catch {
            return .failure(error)
        }
    case .success:
        return self
    }
}

// 如果我们想用`请求成功的结`果来做一些操作的时候，可以用这个方法，把那些操作用closure的形式传入即可
@discardableResult
public func withValue(_ closure: (Value) -> Void) -> Result {
    if case let .success(value) = self { closure(value) }

    return self
}

// 如果我们想用`请求失败的结果`做一些操作的时候，可以用这个方法，把那些操作用closure的形式传入即可
@discardableResult
public func withError(_ closure: (Error) -> Void) -> Result {
    if case let .failure(error) = self { closure(error) }

    return self
}

// 如果我们想在`请求成功后`做一些操作的时候，可以用这个方法，把那些操作用closure的形式传入即可
@discardableResult
public func ifSuccess(_ closure: () -> Void) -> Result {
    if isSuccess { closure() }

    return self
}

// 如果我们想在`请求失败后`做一些操作的时候，可以用这个方法，把那些操作用closure的形式传入即可
@discardableResult
public func ifFailure(_ closure: () -> Void) -> Result {
    if isFailure { closure() }

    return self
}
```
