# 09 - 全局 Actor

应用程序运行在一个主线程上，所以不能创建第二个或第三个主线程。因此，有一个默认的、共享的 actor 实例，可以在任何地方安全地使用，这是合理的。

应用程序范围内单实例共享状态的一些例子如下：

- 应用程序的数据库层。
- 图片或数据缓存。
- 用户的身份验证状态。

Swift 允许创建自己的全局 actor，就像 `MainActor` 一样，适用于需要从任何地方都可以访问单例的情况。

在 Swift 中，可以使用 `@globalActor` 对 actor 进行注释，从而使其自动遵循 `GlobalActor` 协议：

```swift
@globalActor actor MyActor {
  ...
}
```

`GlobalActor` 只有一个要求：必须具有一个名为 shared 的静态属性，该属性是可全局访问的 actor 实例。

正如使用 `@MainActor` 注释方法以允许其代码更改应用程序的 UI 一样，也可以使用带 `@` 前缀的注释自动执行自己自定义的全局 actor 方法：

```swift
@MyActor func say(_ text: String) {
  ... 自动在 MyActor 上执行 ...
}
```

为了避免由于不同线程同时写入数据而导致的并发问题，只需要注释所有相关的方法，并使它们在我们的全局 actor 上运行。

事实上，可以用一个全局 actor 对一个完整的类进行注释，这将把一个类的所有方法和属性进行注释（只要它们不是 `nonisolated` 的）：

```swift
@MyActor class MyClass {
  ...
}
```
