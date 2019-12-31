# 【Swift 3.1】17 - 可选链 (Optional Chaining)

可选链是在一个可能为`nil`的可选类型上查询和调用属性、方法和下标的一个过程。如果这个可选类型有值，那么属性、方法或者下标调用成功；如果可选类型值为`nil`，调用属性、方法或者下标就会返回`nil`。多个查询可以连在一起，只要任何一个环节为`nil`，那么整个链条失败。

## 可选链作为强制解包的一个方法 (Optional Chaining as an Alternative to Forced Unwrapping)

我们想访问可选类型的属性、方法或者下标，在这个可选类型的后面加上`?`来表示可选链。可选链返回的结果永远是可选类型的值，即使属性、方法或者下标不是可选类型。

首先，创建两个类`Person`和`Residence`:

```swift
class Person {
    var residence: Residence?
}
 
class Residence {
    var numberOfRooms = 1
}
```

创建一个`Person`实例，`residence`属性默认为`nil`：

```swift
let john = Person()
```

如果要访问`Person`实例的`residence`的`numberOfRooms`属性，需要在`residence`后面用`!`解包，这将会触发运行时错误，因为`residence`没有值：

```swift
let roomCount = john.residence!.numberOfRooms
// this triggers a runtime error
```

下面使用可选链来访问`numberOfRooms`的值：

```swift
if let roomCount = john.residence?.numberOfRooms {
    print("John's residence has \(roomCount) room(s).")
} else {
    print("Unable to retrieve the number of rooms.")
}
// Prints "Unable to retrieve the number of rooms."
```

把`Residence`实例赋给`john.residence`，那么`john.residence`就不为`nil`了，接着在使用可选链解包，解包成功：

```swift
john.residence = Residence()

if let roomCount = john.residence?.numberOfRooms {
    print("John's residence has \(roomCount) room(s).")
} else {
    print("Unable to retrieve the number of rooms.")
}
// Prints "John's residence has 1 room(s)."
```

## 为可选链定义更多Class模型 (Defining Model Classes for Optional Chaining)

我们可以使用多层次的可选链调用属性、方法和下标。下面将用四个类来演示：

第一个是`Person`，和上面的一样：

```swift
class Person {
    var residence: Residence?
}
```

第二个类是`Residence`，比之前的更复杂些：

```swift
class Residence {
    var rooms = [Room]()
    var numberOfRooms: Int {
        return rooms.count
    }
    subscript(i: Int) -> Room {
        get {
            return rooms[i]
        }
        set {
            rooms[i] = newValue
        }
    }
    func printNumberOfRooms() {
        print("The number of rooms is \(numberOfRooms)")
    }
    var address: Address?
}
```

第三个是`Room`：

```swift
class Room {
    let name: String
    init(name: String) { self.name = name }
}
```

最后一个是`Address`：

```swift
class Address {
    var buildingName: String?
    var buildingNumber: String?
    var street: String?
    func buildingIdentifier() -> String? {
        if let buildingNumber = buildingNumber, let street = street {
            return "\(buildingNumber) \(street)"
        } else if buildingName != nil {
            return buildingName
        } else {
            return nil
        }
    }
}
```

## 使用可选链访问属性 (Accessing Properties Through Optional Chaining)

创建一个`Person`类，并尝试访问`numberOfRooms`属性：

```swift
let john = Person()
if let roomCount = john.residence?.numberOfRooms {
    print("John's residence has \(roomCount) room(s).")
} else {
    print("Unable to retrieve the number of rooms.")
}
// Prints "Unable to retrieve the number of rooms."
```

因为`john.residence`为`nil`，所以这个可选链还是调用失败。

也可以尝试使用可选链来设置属性值：

```swift
let someAddress = Address()
someAddress.buildingNumber = "29"
someAddress.street = "Acacia Road"
john.residence?.address = someAddress
```

尝试设置`john.residence`的属性`address`也不成功，因为目前`john.residence`的值还是为`nil`。把`someAddress`赋给`address`也是可选链的一部分，但是`=`右边的代码其实没有运行。直接通过上面的代码不容易看出，我们可以定义一个方法来创建`someAddress`来测试下：

```swift
func createAddress() -> Address {
    print("Function was called.")
    
    let someAddress = Address()
    someAddress.buildingNumber = "29"
    someAddress.street = "Acacia Road"
    
    return someAddress
}
john.residence?.address = createAddress()
```

我们可以发现，`createAddress()`并没有调用，因为`Function was called.`没有打印。

## 使用可选链调用方法 (Calling Methods Through Optional Chaining)

我们可以使用可选链来调用方法，并检查那个方法是否调用成功，即使那个方法没有返回值。**注意：** 没有返回值的方法，有一个默认返回类型`Void`。

使用可选链调用没有返回值的方法，方法的返回值将会是`Void?`，而不是`Void`，因为使用可选链访问的结果都是可选类型。

```swift
if john.residence?.printNumberOfRooms() != nil {
    print("It was possible to print the number of rooms.")
} else {
    print("It was not possible to print the number of rooms.")
}
// Prints "It was not possible to print the number of rooms."
```

也可以使用类似上面这种检查方法来检查使用可选链赋值是否成功。使用可选链赋值会返回一个`Void?`：

```swift
if (john.residence?.address = someAddress) != nil {
    print("It was possible to set the address.")
} else {
    print("It was not possible to set the address.")
}
// Prints "It was not possible to set the address."
```

## 使用可选链访问下标 (Accessing Subscripts Through Optional Chaining)

**注意：** 使用可选链访问下标，要把`?`放在中括号前面，而不是后面。

例如：

```swift
if let firstRoomName = john.residence?[0].name {
    print("The first room name is \(firstRoomName).")
} else {
    print("Unable to retrieve the first room name.")
}
// Prints "Unable to retrieve the first room name."
```

`john.residence`为`nil`，所以调用下标失败。

如果我们创建`Residence`实例，并给`rooms`数组赋值，然后再把`Residence`实例赋给`john.residence`：

```swift
let johnsHouse = Residence()
johnsHouse.rooms.append(Room(name: "Living Room"))
johnsHouse.rooms.append(Room(name: "Kitchen"))
john.residence = johnsHouse
 
if let firstRoomName = john.residence?[0].name {
    print("The first room name is \(firstRoomName).")
} else {
    print("Unable to retrieve the first room name.")
}
// Prints "The first room name is Living Room."
```

### 访问可选类型的下标 (Accessing Subscripts of Optioinal Type)

如果一个下标返回一个可选类型的值，例如Swift的`Dictionary`类型的键下标：

```swift
var testScores = ["Dave": [86, 82, 84], "Bev": [79, 94, 81]]
testScores["Dave"]?[0] = 91
testScores["Bev"]?[0] += 1
testScores["Brian"]?[0] = 72
// the "Dave" array is now [91, 82, 84] and the "Bev" array is now [80, 94, 81]
```

## 连接多层次链 (Linking Multiple Levels of Chaining)

如果想要获取的类型不是可选类型，因为可选链，他将会变成可选类型；如果想要获取的类型是可选类型，因为可选链，那么它依然是可选类型。例如，如果想要获取的值是`Init`类型，那么将会返回`Init?`类型，不管可选链有多少层；如果想要获取的值是`Init?`类型，那么仍然返回`Init?`类型，不管可选链有多少层。

例如：

```swift
if let johnsStreet = john.residence?.address?.street {
    print("John's street name is \(johnsStreet).")
} else {
    print("Unable to retrieve the address.")
}
// Prints "Unable to retrieve the address."
```
`john.residence`现在是有值的，但是`john.residence.address`为`nil`，所以`john.residence?.address?.street`为`nil`。

如果设置一个真实的`Address`实例给`john.residence.address`，并且设置一个真实的值给`street`属性，我们可以使用多层次可选链访问`street`属性：

```swift
let johnsAddress = Address()
johnsAddress.buildingName = "The Larches"
johnsAddress.street = "Laurel Street"
john.residence?.address = johnsAddress
 
if let johnsStreet = john.residence?.address?.street {
    print("John's street name is \(johnsStreet).")
} else {
    print("Unable to retrieve the address.")
}
// Prints "John's street name is Laurel Street."
```

## 链接有可选类型返回值的方法 (Chaining on Methods with Optional Return Values)

下面是使用可选链访问有可选类型返回值的方法：

```swift
if let buildingIdentifier = john.residence?.address?.buildingIdentifier() {
    print("John's building identifier is \(buildingIdentifier).")
}
// Prints "John's building identifier is The Larches."
```

进一步在方法返回值上执行可选链：

```swift
if let beginsWithThe =
    john.residence?.address?.buildingIdentifier()?.hasPrefix("The") {
    if beginsWithThe {
        print("John's building identifier begins with \"The\".")
    } else {
        print("John's building identifier does not begin with \"The\".")
    }
}
// Prints "John's building identifier begins with "The"."
```
