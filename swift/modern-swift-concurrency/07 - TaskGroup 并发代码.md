# 07 - TaskGroup 并发代码

`async let` 绑定是一种强大的机制，可以帮助设计异步流程，特别是在有很多任务时，其中一些任务需要并行运行，而另一些任务则相互依赖并按顺序运行。

虽然可以灵活地决定使用 `async let` 运行多少任务和哪些任务，但这种语法并不能提供真正的动态并发。

假设需要并行运行一千个任务。写异步 `async let` 一千次是不可能的！

`TaskGroup` 就是用来解决这个问题的。它是一个允许在代码中创建动态并发的 API，可以减少数据争用的可能性，并能让我们安全地处理结果。

## 介绍 TaskGroup

有两种 API 用于构造任务组：`TaskGroup` 和 `ThrowingTaskGroup`。这两个 API 几乎相同，区别在于后者允许抛出错误。

可以使用以下泛型函数之一创建一个组，并帮助编译器正确检查代码的类型：

- `withTaskGroup(of:returning:body:)`：使用给定的任务返回类型、结果返回类型以及作为初始化和运行组的闭包创建组。
- `withThrowingTaskGroup(of:returning:body:)`：采用类似的参数，但每个任务以及整个组都可能抛出错误。

对于这些函数，要注意的是：它们仅在组完成其所有任务后返回。

例如：

```swift
//1
let images = try await withThrowingTaskGroup(
  of: Data.self             // 每个任务的返回类型
  returning: [UIImage].self // 任务组的返回类型。可以在闭包中指定返回类型，这个参数就可以省略
) { group in
  for index in 0..<numberOfImages {
    let url = baseURL.appendingPathComponent("image\(index).png")
    // 把任务添加到组
    group.addTask {
      // 执行异步任务
      return try await URLSession.shared
        .data(from: url, delegate: nil)
        .0
    }
  }
  // TaskGroup 遵循 AsyncSequence 协议，当每个任务完成后，把结果放到数组中并返回
  return try await group.reduce(into: [UIImage]()) { result, data in
    if let image = UIImage(data: data) {
      result.append(image)
    }
  }
}
```

`TaskGroup` 遵循 `AsyncSequence` 协议，因此可以异步迭代组以获取任务返回值，就像常规 `Sequence` 一样。

我们可以使用以下 API 来管理任务组：

- `addTask(priority:operation:)`：将任务添加到组中，以给定（可选）优先级并发执行。
- `addTaskUnlessCancelled(priority:operation:)`：与 `addTask(...)` 相同，只是如果组已取消，则它不会执行任何操作。
- `cancelAll()`：取消组。它将取消当前正在运行的所有任务以及将来添加的所有任务。
- `isCancelled`：如果组被取消，则返回 `true`。
- `isEmpty`：如果组已完成其所有任务，或没有要开始的任务，则返回 `true`。
- `waitForAll()`：等待所有任务完成。在完成组的工作后需要执行某些代码时使用它。
