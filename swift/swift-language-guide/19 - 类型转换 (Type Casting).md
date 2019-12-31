### 【Swift 3.1】19 - 类型转换 (Type Casting)

**类型转换**是用来检查实例的类型，或在继承链中把实例作为不同的父类或子类的一种方法。

Swift中的类型转换用`is`和`as`来实现。

#### 为类型转换定义一个类的层次结构 (Defining a Class Hierarchy for Type Casting)

第一个类是`MediaItem`。假设所有的媒体项目，包括电影和音乐，并且有名字：

```swift
class MediaItem {
    var name: String
    init(name: String) {
        self.name = name
    }
}
```

第二个类是`MediaItem`的子类`Movie`。第三个也是`MediaItem`的子类：

```swift
class Movie: MediaItem {
    var director: String
    init(name: String, director: String) {
        self.director = director
        super.init(name: name)
    }
}
 
class Song: MediaItem {
    var artist: String
    init(name: String, artist: String) {
        self.artist = artist
        super.init(name: name)
    }
}
```

最后创建了一个`library`数组，包含了两个`Movie`实例和三个`Song`实例。Swift根据字面值推断出`library`是`[MediaItem]`类型：

```swift
let library = [
    Movie(name: "Casablanca", director: "Michael Curtiz"),
    Song(name: "Blue Suede Shoes", artist: "Elvis Presley"),
    Movie(name: "Citizen Kane", director: "Orson Welles"),
    Song(name: "The One And Only", artist: "Chesney Hawkes"),
    Song(name: "Never Gonna Give You Up", artist: "Rick Astley")
]
// the type of "library" is inferred to be [MediaItem]
```

虽然这个数组装的是`Movie`和`Song`实例，但如果遍历这个数组，取出来的元素是`MediaItem`类型，而不是`Movie`和`Song`类型。为了使用他们的真实类型，我们需要检查他们的类型，或者向下转型。

#### 检查类型 (Checking Type)

使用`is`来检查一个实例是否是一个子类类型。如果是一个子类类型，返回`true`，否则返回`false`。

```swift
var movieCount = 0
var songCount = 0

for item in library {
	if item is Movie {
		movieCount += 1
	}
	else if item is Song {
		songCount += 1
	}
}

print("Media library contains \(movieCount) movies and \(songCount) songs")
// Prints "Media library contains 2 movies and 3 songs"
```

#### 向下转型 (Downcasting)

一个类型的常量或变量实际上可能是子类的实例类型，我们可以使用`as?`或者`as!`来向下转型到子类类型。

因为向下转型可能会失败，所以转型运算符有两种类型。`as?`返回你想转到的那个类型的可选类型；而`as!`如果转型成功，返回你想转到的那个类型，转型失败将会报错。

如果不确定是否能转型成功，使用`as?`；如果能确定转型成功，使用`as!`。

```swift
for item in library {
	if let movie = item as? Movie {
        print("Movie: \(movie.name), dir. \(movie.director)")
	}
	else if let song = item as? Song {
        print("Song: \(song.name), by \(song.artist)")
	}	
}
```

**注意：**转型实际上不会修改实例或者改变它的值，在底层中，还是同一个实例。

#### Any和AnyObject的类型转换 (Type Casting for Any and AnyObject)

Swift提供了两种不确定的类型：

- `Any`可以代表任何类型的实例，包括方法类型
- `AnyObject`代表任何class类型的实例

下面是一个例子：

```swift
var things = [Any]()
 
things.append(0)
things.append(0.0)
things.append(42)
things.append(3.14159)
things.append("hello")
things.append((3.0, 5.0))
things.append(Movie(name: "Ghostbusters", director: "Ivan Reitman"))
things.append({ (name: String) -> String in "Hello, \(name)" })
```

向下转型：

```swift
for thing in things {
    switch thing {
    case 0 as Int:
        print("zero as an Int")
    case 0 as Double:
        print("zero as a Double")
    case let someInt as Int:
        print("an integer value of \(someInt)")
    case let someDouble as Double where someDouble > 0:
        print("a positive double value of \(someDouble)")
    case is Double:
        print("some other double value that I don't want to print")
    case let someString as String:
        print("a string value of \"\(someString)\"")
    case let (x, y) as (Double, Double):
        print("an (x, y) point at \(x), \(y)")
    case let movie as Movie:
        print("a movie called \(movie.name), dir. \(movie.director)")
    case let stringConverter as (String) -> String:
        print(stringConverter("Michael"))
    default:
        print("something else")
    }
}
 
// zero as an Int
// zero as a Double
// an integer value of 42
// a positive double value of 3.14159
// a string value of "hello"
// an (x, y) point at 3.0, 5.0
// a movie called Ghostbusters, dir. Ivan Reitman
// Hello, Michael
```

**注意：** `Any`代表任何类型的值，包括可选类型。如果在需要传入`Any`类型的位置传入一个可选类型的值，Swift会给你一个警告。如果确实需要把可选类型的值作为一个`Any`类型的值，可以使用`as`来明确地转为`Any`类型：

```swift
let optionalNumber: Int? = 3
things.append(optionalNumber)        // Warning
things.append(optionalNumber as Any) // No warning
```
