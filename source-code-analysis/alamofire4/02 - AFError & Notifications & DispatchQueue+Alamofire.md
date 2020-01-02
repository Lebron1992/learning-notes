### 一、AFError枚举

作者首先把整个项目中遇到的error都集中在`AFError`枚举。从源码中我们可以看到，项目中遇到的error有五大类，就是`AFError`的五个case，每个case都有一个关联值（[什么是关联值 >>](http://www.jianshu.com/p/4b7ab493cae7)）。我们利用Swift枚举的关联值特性，在抛出错误的时候，关联一个任何类型的值。例如如果是这个错误`case invalidURL(url: URLConvertible)`，这里关联了一个url，这样就可以在遇到error的时候，告诉我们哪个url是不合法的。而其他四个case关联的都是`AFError`里面内置的enum类型（Swift允许[类型嵌套](http://www.jianshu.com/p/dd10a918b0c8)），可以告诉我们更具体的错误原因。

```swift
public enum AFError: Error {

    // 参数编码失败的原因
    public enum ParameterEncodingFailureReason {
    
    }

    // 多表单数据编码失败的原因
    public enum MultipartEncodingFailureReason {
    
    }

    // 响应验证失败的原因
    public enum ResponseValidationFailureReason {

    }

    // 响应序列化失败的原因
    public enum ResponseSerializationFailureReason {

    }

    // 五大类错误
    case invalidURL(url: URLConvertible)
    case parameterEncodingFailed(reason: ParameterEncodingFailureReason)
    case multipartEncodingFailed(reason: MultipartEncodingFailureReason)
    case responseValidationFailed(reason: ResponseValidationFailureReason)
    case responseSerializationFailed(reason: ResponseSerializationFailureReason)
}
```

### 二、AdapterError

这个`AdapterError`是专门为`RequestAdapter`服务的。

这里先简单介绍下`RequestAdapter`：`RequestAdapter`是个协议，我们可以自定义一个请求适配器，并且遵循这个协议。通过我们自定义的适配器，我们在发送请求的时候可以默认添加一些请求相关的数据，具体请查看：[08 - Request](https://github.com/Lebron1992/learning-notes/blob/master/source-code-analysis/alamofire4/08%20-%20Request.md)。

`AdapterError`就是在请求适配过程中可能出现的错误的结构类型。在`Error`的extension中定义的`underlyingAdaptError`就是适配过程中可能出现的错误。

### 三、Error Booleans

这里定义了一些方便使用的只读属性，方便我们在开发的时候作出判断。其实在我们的开发中，也会经常把一些判断写成只读属性或者是一个返回Bool的方法。这是一个非常好的习惯，提高代码的可读性。在Swift中，苹果已经习惯性地把Bool类型的属性以`is`开头命名，我个人也觉得用`is`开头命名，可读性会更高。

```swift
extension AFError {
    public var isInvalidURLError: Bool {
        if case .invalidURL = self { return true }
        return false
    }
    
    // 更多 Error Booleans ...
}
```

### 四、Convenience Properties

这部分内容是一些方便使用的与error相关的属性，没有很特别的。

### 五、Notifications

Notifications这个文件通过扩展`Notification.Name`定义了项目中用到的通知名称。我们可以学习用这个形式来定义`rawValue`：`org.alamofire.notification.name.task.didResume`。另外还定义了一个Key。

### 六、DispatchQueue+Alamofire

`DispatchQueue+Alamofire`里面，通过扩展`DispatchQueue`定义了不同优先级的全局队列：
* `userInteractive`：与用户交互的任务队列，通常跟UI的刷新有关，例如动画之类的
* `userInitiated`：用户发起的并且需要立即得到结果的任务队列
* `utility`：需要花点时间的任务队列
* `background`：后台任务队列，用户不需要关心的，通常时间会比较长

另外还定义了一个方法，经过一个指定的时间后，执行一个closure，这种写法看起来更通熟易懂：

```swift
func after(_ delay: TimeInterval, execute closure: @escaping () -> Void) {
    asyncAfter(deadline: .now() + delay, execute: closure)
}
```
