### 集合类型 (Collection Types)

Swift提供了三种集合类型：数组(`Array`)、集合(`Set`)、字典(`Dictionary`)。`Array`是有顺序的值的集合；`Set`是多个唯一的值的无序集合；`Dictionary`是无序的键值对集合。

**注意：**Swift的`Array`、`Set`和`Dictionary`都属于泛型集合。

#### 数组 (Array)

数组只能存储相同类型的值。相同的值可以出现在数组的不同位置中。

##### 数组类型的速记语法 (Array Type Shorthand Syntax)

一个数组的类型是这样写的：`Array<Element>`，`Element`是数组元素值的类型，也可以简写成：`[Element]`。

##### 创建一个空数组 (Creating an Empty Array)

```swift
var someInt = [Int]()
print("someInts is of type [Int] with \(someInts.count) items.")
// Prints "someInts is of type [Int] with 0 items."
```

`someInt`被推断为`[Int]`类型。

如果上下文已经提供了数组的类型，空素组还可以写成`[]`：

```swift
someInt.append(3)
// someInts now contains 1 value of type Int
someInt = []
// someInts is now an empty array, but is still of type [Int]
```

##### 创建一个有默认值的数组 (Creating an Array with a Default Value)

```swift
var threeDoubles = Array(repeating: 0.0, count: 3)
// threeDoubles is of type [Double], and equals [0.0, 0.0, 0.0]
```

##### 通过合并两个数组来创建数组 (Creating an Array by Adding Two Arrays Together)

```swift
let anotherThreeDoubles = Array(repeating: 2.5, count: 3)
// anotherThreeDoubles is of type [Double], and equals [2.5, 2.5, 2.5]

var sixDoubles = threeDoubles + anotherThreeDoubles
// sixDoubles is inferred as [Double], and equals [0.0, 0.0, 0.0, 2.5, 2.5, 2.5]
```

##### 用字面值创建数组 (Creating an Array with an Array Literal)

```swift
var shoppingLis = ["Eggs", "Milk"]
// shoppingList has been initialized with two initial items
```

##### 访问和修改数组 (Accessing and Modifying an Array)

获取数组的个数：

```swift
print("The shopping list contains \(shoppingList.count) items.")
// Prints "The shopping list contains 2 items."
```

判断数组元素的个数是否为0：

```swift
if shoppingList.isEmpty {
    print("The shopping list is empty.")
} else {
    print("The shopping list is not empty.")
}
```

追加一个元素：

```swift
shoppingList.append("Flour")
// shoppingList now contains 3 items, and someone is making pancakes
```

使用加法赋值运算符添加更多元素：

```swift
shoppingList += ["Baking Powder"]
// shoppingList now contains 4 items
shoppingList += ["Chocolate Spread", "Cheese", "Butter"]
// shoppingList now contains 7 items
```

使用下标获取元素：

```swift
var firstItem = shoppingList[0]
// firstItem is equal to "Eggs"
```

更改元素：

```swift
shoppingList[0] = "Six eggs"
// the first item in the list is now equal to "Six eggs" rather than "Eggs"
```

使用下标一次性更改多个元素，甚至要更改的元素个数可以不等于新数组的个数：

```swift
shoppintList[4...6] = ["Bananas", "Apples"]
// 用两个替换三个
```

在特定的位置插入元素：

```swift
shoppingList.insert("Maple syrup", at: 0)
// shoppingList now contains 7 items
// "Maple Syrup" is now the first item in the list
```

删除特定位置的元素，并且返回被删除的元素：

```swift
let mapleSyrup = shoppingList.remove(at: 0)
// the item that was at index 0 has just been removed
// shoppingList now contains 6 items, and no Maple Syrup
// the mapleSyrup constant is now equal to the removed "Maple Syrup" string
```

删除最后一个元素：

```swift
let apples = shoppingList.removeLast()
// the last item in the array has just been removed
// shoppingList now contains 5 items, and no apples
// the apples constant is now equal to the removed "Apples" string
```

##### 遍历整个数组 (Iterating Over an Array)

使用`for-in`遍历：

```swift
for item in shoppingList {
    print(item)
}
// Six eggs
// Milk
// Flour
// Baking Powder
// Bananas
```

使用`enumerated()`方法遍历，这个方法返回包含索引和索引对应的元素的多元组：

```swift
for (index, value) in shoppingList.enumerated() {
    print("Item \(index + 1): \(value)")
}
// Item 1: Six eggs
// Item 2: Milk
// Item 3: Flour
// Item 4: Baking Powder
// Item 5: Bananas
```

#### 集合 (Sets)

集合中无顺序地存储了同一类型的值，并且里面的每一个值都是唯一的。在元素的顺序不重要或者要求每一个元素都需要唯一的时候，可以使用集合，而不用数组。

##### 集合类型的哈希值 (Hash Values for Set Types)

集合里面的元素类型必须*hashable*，也就是说，这个元素类型必须提供一个方法来计算他自己的哈希值。一个哈希值是一个用来判断两个对象是否相等的`Int`类型的整数。例如，如果`a == b`，那么`a.hashValue == b.hashValue`。

所有Swift的基本类型(例如`String`、`Int`、`Double`、`Bool`)默认都是*hashable*的，都可以作为集合的值类型或者字典的键类型。没有关联值的枚举值默认也是*hashable*的。

我们可以自定义类型，并且遵循`Hashable`协议，作为集合或者字典键的值类型。自定义的类型必须提供一个能读取的`Int`类型的属性，并命名为`hashValue`。在不同的程序或者同一个程序运行多次中，不要求每次`hashValue`属性返回的值都相等。

因为`Hashable`协议遵循`Equatable`协议，所以我们自定义的类型还需要提供一个相等运算符(`==`)的实现。`Equatable`协议要求每一个`==`的实现是一个等价关系。也就是说，`==`的实现必须满足下面三个条件：

- `a == a` (自反性)
- `a == b`，说明`b == a` (对称性)
- `a == b && b == c`，说明 `a == c` (传递性)

##### 集合类型语法 (Set Type Syntax)

使用`Set<Element>`来设置集合类型，`Element`是集合存储的元素类型。

##### 创建和初始化一个空集合 (Creating and Initializing an Empty Set)

```swift
var letters = Set<Character>()
print("letters is of type Set<Character> with \(letters.count) items.")
// Prints "letters is of type Set<Character> with 0 items."
```

`letters`被推断为`Set<Character>`类型。

同样地，如果上下文提供了集合的类型信息，可以使用`[]`来创建一个空的集合：

```swift
letters.insert("a")
// letters now contains 1 value of type Character
letters = []
// letters is now an empty set, but is still of type Set<Character>
```

##### 使用数组字面值来创建一个集合 (Creating a Set with an Array Literal)

```swift
var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"]
// favoriteGenres has been initialized with three initial items
```

集合的类型不能通过数组的字面值来推断，所以`Set`的类型必须明确声明。但是因为Swift的类型推断，如果用一个包含相同类型字面值的数组来初始化集合，我们可以不写集合的类型。例如：

```swift
var favoriteGenres: Set = ["Rock", "Classical", "Hip hop"]
```

因为数组的全部字面值都是同一类型，所以Swfit能推断出`Set<String>`是`favoriteGenres`的正确类型。

##### 访问和修改集合 (Accessing and Modifying a Set)

使用`count`属性获取集合元素个数：

```swift
print("I have \(favoriteGenres.count) favorite music genres.")
// Prints "I have 3 favorite music genres."
```

使用`isEmpty`属性判断集合中元素的个数是否为0：

```swift
if favoriteGenres.isEmpty {
    print("As far as music goes, I'm not picky.")
} else {
    print("I have particular music preferences.")
}
// Prints "I have particular music preferences."
```

使用`insert(_:)`方法添加元素：

```swift
favoriteGenres.insert("Jazz")
// favoriteGenres now contains 4 items
```

使用`remove(_:)`删除一个元素，并返回被删除的元素，如果元素不存在，返回`nil`；使用`removeAll()`删除全部元素：

```swift
if let removedGenre = favoriteGenres.remove("Rock") {
    print("\(removedGenre)? I'm over it.")
} else {
    print("I never much cared for that.")
}
// Prints "Rock? I'm over it."
```

判断是否包含某个元素：

```swift
if favoriteGenres.contains("Funk") {
    print("I get up on the good foot.")
} else {
    print("It's too funky in here.")
}
// Prints "It's too funky in here."
```

##### 遍历整个集合 (Iterating Over a Set)

```swift
for genre in favoriteGenres {
	print("\(genre)")
}
// Jazz
// Hip hop
// Classical
```

Swift的集合类型没有定义顺序，我们可以使用`sorted()`方法来排序，这个方法使用`<`运算符将元素从小到大排列：

```swift
for genre in favoriteGenres.sorted() {
	print("\(genre)")
}
// Classical
// Hip hop
// Jazz
```

#### 执行集合操作 (Performing Set Operations)

##### 基本集合操作 (Fundamental Set Operations)

下图是集合`a`和`b`执行了不同的方法之后，得出的结果图：


![Fundamental Set Operations](http://upload-images.jianshu.io/upload_images/2057254-ba7541df6311d387.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 使用`intersection(_:)`方法得到两个集合共有的元素，并用这些相同的元素创建一个新的集合
- 使用`symmetricDifference(_:)`方法得到除了两个集合共有的元素外的所有元素，并用这些相同的元素创建一个新的集合
- 使用`union(_:)`方法得到两个集合的所有元素，并用这些相同的元素创建一个新的集合
- 使用`subtracting(_:)`方法减去与指定集合相同的元素后剩下的元素，并用剩下的元素创建一个新的集合

```swift
let oddDigits: Set = [1, 3, 5, 7, 9]
let evenDigits: Set = [0, 2, 4, 6, 8]
let singleDigitPrimeNumbers: Set = [2, 3, 5, 7]
 
oddDigits.union(evenDigits).sorted()
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
oddDigits.intersection(evenDigits).sorted()
// []
oddDigits.subtracting(singleDigitPrimeNumbers).sorted()
// [1, 9]
oddDigits.symmetricDifference(singleDigitPrimeNumbers).sorted()
// [1, 2, 9]
```

##### 集合关系和相等性 (Set Membership and Equality)

下图演示了三个集合：`a`、`b`和`c`，重叠区域代表有相同的元素。集合`a`是集合`b`的父集合，因为`a`包含了`b`的所有元素；相反，`b`是`a`的子集合。集合`b`和集合`c`互不相交，因为他们没有相同的元素。


![Set Membership and Equality](http://upload-images.jianshu.io/upload_images/2057254-ec4de3a928d64fbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 使用“是否相等”运算符 (`==`)来判断两个集合的所有元素是否相等
- 使用`isSubset(of:)`方法判断集合的所有元素是否包含于指定集合
- 使用`isSuperset(of:)`方法判断集合是否包含指定集合的所有元素
- 使用`isStrictSubset(of:)`或者`isStrictSuperset(of:)`方法判断集合是否子集合或者父集合，但是不等于指定的集合
- 使用`isDisjoint(with:)`方法判断两个集合是否有相同的元素

```swift
let houseAnimals: Set = ["🐶", "🐱"]
let farmAnimals: Set = ["🐮", "🐔", "🐑", "🐶", "🐱"]
let cityAnimals: Set = ["🐦", "🐭"]
 
houseAnimals.isSubset(of: farmAnimals)
// true
farmAnimals.isSuperset(of: houseAnimals)
// true
farmAnimals.isDisjoint(with: cityAnimals)
// true
```

#### Dictionaries (字典)

字典是一个无序集合中相同类型的键和相同类型的值的关联。每一个值关联着一个唯一的键。

##### 字典类型速记语法 (Dictionary Type Shorthand Syntax)

使用`Dictionary<Key, Value>`来指定字典的类型。

**注意：**字典的`Key`类型必须遵循`Hashable`协议，就像集合的值一样。

还可以是用简短的形式`[Key: Value]`来指定字典的类型.

##### 创建一个空字典 (Creating an Empty Dictionary)

```swift
var namesOfIntegers = [Int: String]()
// namesOfIntegers is an empty [Int: String] dictionary
```

如果上下文已经提供了类型信息，可以使用`[:]`来创建一个空字典：

```swift
namesOfIntegers[16] = "sixteen"
// namesOfIntegers now contains 1 key-value pair
namesOfIntegers = [:]
// namesOfIntegers is once again an empty dictionary of type [Int: String]
```

##### 利用字典字面值来创建字典 (Creating a Dictionary with a Dictionary Literal)

```swift
var airports = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]
```

##### 访问和修改字典 (Accessing and Modifying Dictionary)

获取字典键值对的个数：

```swift
print("The airports dictionary contains \(airports.count) items.")
// Prints "The airports dictionary contains 2 items."
```

判断字典中键值对的个数是否为0：

```swift
if airports.isEmpty {
    print("The airports dictionary is empty.")
} else {
    print("The airports dictionary is not empty.")
}
// Prints "The airports dictionary is not empty."
```

使用下标语法添加新的键值对：

```swift
airports["LHR"] = "London Heathrow"
// the value for "LHR" has been changed to "London Heathrow"
```

还可以使用`updateValue(_:forKey:)`方法来设置或更新一个键对应的值，并返回一个可选类型的值。如果这个键不存在，那么就添加一个新的键值对，并返回`nil`；如果这个键存在，那么就更新这个键对应的值，并返回之前的旧值。这可以让我们检查键对应的值是否更新成功。

```swift
if let oldValue = airports.updateValue("Dublin Airport", forKey: "DUB") {
    print("The old value for DUB was \(oldValue).")
}
// Prints "The old value for DUB was Dublin."
```

使用下标语法来获取键对应的值：

```swift
if let airportName = airports["DUB"] {
    print("The name of the airport is \(airportName).")
} else {
    print("That airport is not in the airports dictionary.")
}
// Prints "The name of the airport is Dublin Airport."
```

使用下标语法并把键对应的值设置为`nil`来删除一个键值对：

```swift
airports["APL"] = "Apple International"
// "Apple International" is not the real airport for APL, so delete it
airports["APL"] = nil
// APL has now been removed from the dictionary
```

另外，还可以使用`removeValue(forKey:)`方法来删除一个键值对，如果存在，返回键对应的值；如果不存在，返回`nil`：

```swift
if let removedValue = airports.removeValue(forKey: "DUB") {
    print("The removed airport's name is \(removedValue).")
} else {
    print("The airports dictionary does not contain a value for DUB.")
}
// Prints "The removed airport's name is Dublin Airport."
```

##### 遍历整个字典 (Iterating Over a Dictionary)

```swift
for (airportCode, airportName) in airports {
    print("\(airportCode): \(airportName)")
}
// YYZ: Toronto Pearson
// LHR: London Heathrow
```

使用`keys`和`values`属性来遍历字典的所有键和所有值：

```swift
for airportCode in airports.keys {
    print("Airport code: \(airportCode)")
}
// Airport code: YYZ
// Airport code: LHR

for airportName in airports.values {
    print("Airport name: \(airportName)")
}
// Airport name: Toronto Pearson
// Airport name: London Heathrow
```

如果要使用字典的所有键和所有值，可以利用数组的API来创建：

```swift
let airportCodes = [String](airports.keys)
// airportCodes is ["YYZ", "LHR"]

let airportNames = [String](airports.values)
// airportNames is ["Toronto Pearson", "London Heathrow"]
```

Swift的字典类型没有定义顺序，为了遍历经过排序的所有键和所有值，需要使用`keys`和`values`属性的`sorted()`方法。
