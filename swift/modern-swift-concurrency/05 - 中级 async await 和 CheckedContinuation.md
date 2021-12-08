# 05 - 中级 async await 和 CheckedContinuation

## 介绍 continuation

回调和委托模式构成了苹果平台上异步编程的基石。对于回调，传入一个在工作完成时执行的闭包。对于委托模式，可以创建一个委托对象，然后在工作进行或完成时对其调用某些方法。

为了鼓励新并发模型的采用，苹果设计了一个最小但功能强大的API，在桥接现有代码时非常方便。它围绕着一个叫做 **continuation** 的概念。

continuation 是跟踪程序在给定点的状态的对象。Swift 并发模型为每个异步工作单元分配一个continuation，而不是为其创建整个线程。这允许并发模型根据硬件的功能更有效地扩展工作。它只创建与可用 CPU 内核数量相同的线程，并在 continuations 之间切换，而不是在线程之间切换，从而提高效率。

我们已经了解 await 调用的工作原理：当前代码暂停执行，并将线程和系统资源交给中央处理程序，由中央处理程序决定下一步的操作。当等待的函数完成时，只要没有更高优先级的任务挂起，原始代码就会恢复。但这是怎么做到的呢？

当原始代码挂起时，它会创建一个表示挂起点处的状态的 continuation。当恢复执行或抛出错误时，并发系统将从 continuation 中重新创建状态。

这一切都发生在使用 `async` 函数的背后。我们还可以自己创建 continuation，可以使用它来扩展使用回调或委托的现有代码。这些 API 也可以从使用 `await` 中获益。

手动创建 continuations 可以将现有代码逐渐迁移到新的并发模型。

## 手动创建 continuations

continuation API 有两种：

- **CheckedContinuation**：恢复挂起的执行或抛出错误的机制。它提供正确使用的运行时检查，并记录任何误用。
- **UnsafeContinuation**：CheckedContinuation 的替代方案，但不进行安全检查。当性能至关重要且不需要额外的安全性时，使用此选项。

> **注意**：这两个 API 本质上是相同的，因此在本章中将只使用 CheckedContinuation。对于本章中提到的名称中包含 “checked” 的任何函数，可以假定也存在 “unsafe” 的函数。

我们通常不会自己初始化一个 continuation。相反，使用两个方便的带有闭包的通用函数。闭包提供了一个随时可用的 continuation 作为输入参数：

- `withCheckedContinuation(_:)`：包装闭包并返回一个 checked continuation。
- `withCheckedThrowingContinuation(_:)`：包装抛出闭包。在需要错误处理时使用此选项。

我们**只能**恢复 continuation 一次。强制执行此规则是 checked 和 unsafe continuations 之间的区别。可以使用以下方法之一恢复 continuation：

- `resume()`：不传入值恢复挂起的任务。
- `resume(returning:)`：恢复挂起的任务并返回给定值。
- `resume(throwing:)`：继续挂起的任务，抛出提供的错误。
- `resume(with:)`：使用包含值或错误的 `Result` 进行恢复。

上面的方法是唯一可以调用 continuation 的方法。

## 包装委托模式

我们将以 `CLLocationManagerDelegate` 为例，演示如何使用 continuation 来包装委托模式。

在本例中，目的是获取用户的当前位置。在处理 `CLLocationManagerDelegate` 之前，首先创建一个 continuation：

```swift
class ViewModel: ObservableObject {

  private var locationDelegate: LocationDelegate?

  func shareLocation() {
    let location: CLLocation = try await 
    withCheckedThrowingContinuation { [weak self] continuation in

    }
  }
}
```

`withCheckedThrowingContinuation(_:)` 接受抛出闭包，挂起当前任务，然后执行闭包。我们应该在闭包中调用异步代码，然后在完成后恢复 continuation 参数。在这种情况下，将返回一个 error 或 location。

我们可以像传递任何其他变量一样传递 continuation，将其存储在模型中或传递给其他函数。无论它在哪里结束，调用它的 `resume(...)` 方法之一将始终在原始调用点恢复执行。

如果现在调用 `shareLocation()` 方法，在控制台将会看到类似下面的错误：

```
SWIFT TASK CONTINUATION MISUSE: shareLocation() leaked its continuation!
```

运行时检测到从未使用 continuation，并且在闭包结束时释放了变量。在 `try await withCheckedThrowingContinuation(...)` 中的代码永远不会从暂停点成功恢复。

就像之前提到的，必须从每个代码路径准确调用 `resume(...)` 方法一次。

下面定义 `LocationDelegate` 来使用 continuation：

```swift
typealias LocationContinuation = CheckedContinuation<CLLocation, Error>

class LocationDelegate: NSObject {
  private var continuation: LocationContinuation?
  private let manager = CLLocationManager()

  init(continuation: LocationContinuation) {
    self.continuation = continuation
    super.init()
    manager.delegate = self
    manager.requestWhenInUseAuthorization()
  }
}

extension LocationDelegate: CLLocationManagerDelegate {

  func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
    switch manager.authorizationStatus {
    case .notDetermined:
      break
    case .authorizedAlways, .authorizedWhenInUse:
      manager.startUpdatingLocation()
    default:
      continuation?.resume(throwing: "应用未授权位置权限")
      continuation = nil
    }
  }

  func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    guard let location = locations.first else {
      return
    }
    continuation?.resume(returning: location)
    continuation = nil
  }

  func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
    continuation?.resume(throwing: error)
    continuation = nil
  }
}
```

在上面代码中，每一种可能获取到位置或者出错的路径中都调用了 `continuation.resume(...)`，然后把 `continuation` 设置为 `nil`，保证后续不能再使用。

使用 `LocationDelegate`：

```swift
let location: CLLocation = try await 
withCheckedThrowingContinuation { [weak self] continuation in
  self.locationDelegate = LocationDelegate(continuation: continuation)
}
```

执行此代码将会：

- `manager` 在 `LocationDelegate` 初始化时调用 `locationManagerDidChangeAuthorization`。
- 用户授予权限后，`manager` 将获取设备位置。
- `manager` 调用 `didUpdateLocations`。
- `LocationDelegate` 调用 `continuation.resume(...)` 返回第一个可用位置来恢复挂起任务。
- 原始调用点 `let location: CLLocation = try await withCheckedThrowingContinuation ...` 恢复执行，允许我们使用返回的位置。

## 包装回调

```swift
let status: ATTrackingManager.AuthorizationStatus = await
withCheckedContinuation({ continuation in
  ATTrackingManager.requestTrackingAuthorization { status in
    continuation.resume(returning: status)
  }
})
```

包装回调更简单，直接在回调使用 `continuation.resmue(...)` 返回结果即可。
