> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

适配器模式可以让不兼容的类型在一起正常的工作，它包含以下四个组成部分：

![](http://upload-images.jianshu.io/upload_images/2057254-4c8674f217a50efd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **使用adapter的对象**：这个对象拥有遵循新协议的adapter
- **新的协议**：为了能让不兼容的类型一起正常工作而新建的协议
- **适配器**：遵循新的协议，并且拥有旧的对象，调用就对象的方法
- **旧对象**：在新协议建立之前就已经存在的对象，并且不能直接修改它让他去遵循新的协议

看到现在，可能比较难懂。我们举个例子：最新的iPhone都把3.5mm的耳机接口给干掉了，如果只有3.5mm接口的耳机，我们又想用耳机听歌，那么必须要有一个3.5mm转lightning的转换器才能使用3.5mm接口的耳机。在这个例子中，转接头就是**适配器**；3.5mm接口的耳机就是**旧对象**；3.5mm转lighting的技术实现就是**新的协议**；最新的iPhone就是**使用adapter的对象**。

### 什么时候使用

有些方法、类或者模块不能直接修改，特别是当他们来自第三方库的时候，当我们又需要让他们遵循我们的设计的规则时，我们可以使用适配器模式。

### 简单Demo 

国内的很多应用支持微信、QQ、微博等进行登录，他们的登录api肯定会不大一样。现在我们想用一个统一的api来处理这些第三方的登录，就可以用到适配器模式。

这里以微博登录为例。

#### 第三方库的api

我们假设下面的api来自第三方库，不能直接被修改：

```swift
public struct WeiboUser {
    public let username: String
    public let email: String
    public let token: String
}

public final class WeiboAuthenticator {
    public func login(email: String,
                      password: String,
                      completion: (WeiboUser?, Error?) -> Void) {

        let token = "a_token_value"
        let username = "a_user_name"
        let user = WeiboUser(username: username,
                             email: email,
                             token: token)
        completion(user, nil)
    }
}
```

#### 统一第三方登录的api

为了统一第三方登录的api，我们需要一个新的协议`AuthenticationServiceable`:

```swift
struct User {
    let email: String
    let password: String
}

struct Token {
    let value: String
}

protocol AuthenticationServiceable {
    func login(email: String,
               password: String,
               success: (User, Token) -> Void,
               failure: (Error?) -> Void)
}
```

接下来我们改创建适配器了：

```swift
final class WeiboAuthenticatorAdapter: AuthenticationServiceable {

    private lazy var authenticator = WeiboAuthenticator()

    func login(email: String,
               password: String,
               success: (User, Token) -> Void,
               failure: (Error?) -> Void) {

        authenticator.login(email: email, password: password) { (weiboUser, error) in
            guard let weiboUser = weiboUser else {
                failure(error)
                return
            }

            let user = User(username: weiboUser.username,
                            email: weiboUser.email)
            let token = Token(value: weiboUser.token)
            success(user, token)
        }
    }
}
```

我们让`WeiboAuthenticatorAdapter`适配器拥有了一个旧的`WeiboAuthenticator`实例，然后实现`AuthenticationServiceable`协议。

我们把`WeiboAuthenticator`包装到`WeiboAuthenticatorAdapter`适配器了，如果微博的登录api改变，我们只需要在适配器更新代码即可。

#### 使用

```swift
let authService = WeiboAuthenticatorAdapter()
authService.login(email: "test@example.com",
                  password: "password",
                  success: { (user, token) in
                    print("登录成功： 1) 邮件：\(user.email), 2）token: \(token.value)")
},
                  failure: { (error) in
                    print("登录失败：\(error?.localizedDescription ?? "")")
})
```

### 总结

新的协议在适配器模式中是必须的，保证adapter在我们的应用中能正常使用。如果第三方的代码改变，我们更新adapter中的代码即可。
