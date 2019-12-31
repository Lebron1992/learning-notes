### 字符串和字符 (Strings and Characters)

#### 字符串字面值 (String Literals)

使用字符串字面值来初始化一个常量或者变量：

```swift
let someString = "Some string literal value"
```

Swift根据字面值来推断出`someString`是`String`类型。

#### 初始化一个空字符串

为了创建更长的字符串，我们通常要先初始化一个空字符串，我们可以使用下面两种方法：

```swift
var emptyString = "" 				// 空字符串字面值
var anotherEmptyString = String()   // 使用默认构造函数
// 这两个字符串都是空的，并且是相等的
```

通过检查字符串的布尔类型属性`isEmpty`来判断一个字符串是否为空：

```swift
if emptyString.isEmpty {
	print("这里什么都没有")
}
// Prints "这里什么都没有"
```

#### 字符串的可变性 (String Mutability)

使用`let`声明一个不可变的字符串，用`var`声明一个可变的字符串：

```swift
var variableString = "Horse"
variableString += "and carriage"
// variableString 现在是 "Horse and carriage"

let constatString = "Highlander"
constantString += "and another Highlander"
// 会产生一个编译错误，因为constantString是一个常量，不能被改变
```

在OC中，需要使用`NSString`和`NSMutableString`来分别定义不可变和可变字符串。

#### 字符串是值类型 (Strings Are Value Types)

Swift的`String`类型是值类型。如果我们创建一个新的字符串，那么把这个字符串以复制的形式传给一个函数或方法，或者把它赋值给另外一个常量或常量。

在Swift的底层实现，Swift的编译器会优化字符串的使用，在有必要的时候才会进行真正的复制，这就意味着把字符串作为值类型在使用的时候，都有非常好的性能。

#### 与字符交互 (Working with Characters)

使用`for-in`循环遍历字符串的`characters`属性来访问单个字符：

```swift
for character in "Dog!🐶".characters {
    print(character)
}
// D
// o
// g
// !
// 🐶
```

字符串可以通过传递一个字符数组给`String`的构造函数来初始化：

```swift
let catCharacters: [Character] = ["C", "a", "t", "!", "🐱"]
let catString = String(catCharacters)
print(catString)
// Prints "Cast!🐱"
```

####连接字符串和字符 (Concatenating Strings and Characters)

使用加号运算符(`+`)来拼接两个字符串：

```swift
let string1 = "hello"
let string2 = "there"
let welcome = string1 + string2
// welcome 等于 "hello there"
```

使用加法赋值运算符来拼接一个已经存在字符串：

```swift
var instruction = "look over"
instruction += string2
// instruction 等于 "look over there"
```

使用`String`类型的`append()`方法来拼接：

```swift
let exclamationMark: Character = "!"
welcome.append(exclamationMark)
// welcome 等于 "hello there!"
```

**注意：**不能添加一个`String`或者`Character`到一个已经存在的`Character`变量，因为`Character`类型的值只能包含一个字符。

#### 字符串插值 (String Interpolation)

字符串插值可以把常量、变量、字面值和其他表达式混合在一起，拼接成一个新的字符串。

```swift
let multiplier = 3
let message = "\(multiplier) 乘以 2.5 等于 \(Double(multiplier) * 2.5)"
// message is "3 乘以 2.5 等于 7.5"
```

#### Unicode

*Unicode*是在不同系统中编码、展示和处理文本的国际标准。可以让我们以国际化的形式在不同的语言中展示几乎任何字符，从外部文件（例如text文件或网页）读取那些字符或者把那些字符写到外部文件。Swift的`String`和`Character`类型是完全兼容Unicode的。

##### Unicode标量 (Unicode Scalars)

在底层，Swift的原生`String`类型是从Unicode标量值建立的。一个Unicode标量是一个字符或者修饰符唯一的21-bit数字，例如`U+0061`是`LATIN SMALL LETTER A`(`"a"`)，`U+1F425`是`FRONT-FACING BABY CHICK` (`"🐥"`)。

**注意：**一个Unicode标量是在`U+0000`到`U+D7FF`(包含首尾)这个范围中的一个代码点(*code point*)，或者是`U+E000`到`U+10FFFF`(包含首尾)这个范围的一个代码点。Unicode标量不包含Unicode代理对(surrogate pair)代码点，代理对的代码点范围是`U+D800`到`U+DFFF`(包含首尾)。

并不是所有的21-bitUnicode标量都赋值给了一个字符，有一些标量是保留起来给未来使用的。被赋值给字符串的Unicode标量都有一个名字，例如上面例子中的`LATIN SMALL LETTER A`和FRONT-FACING BABY CHICK`。

##### 字符串字面值中的特殊字符 (Special Characters in String Literals)

字符串字面值可以包含以下字符：

- 转义字符：`\0`(空字符)、`\\`(反斜杠)、`\t`(水平制表符)、`\n`(换行)、`\r`(回车)、`\"`(双引号)、`\'`(单引号)
- 一个任意的Unicode标量，写成这个样式`\u{n}`，n是1~8的十六进制数字并且等于一个有效的Unicode代码点

```swift
let wiseWords = "\"Imagination is more important than knowledge\" - Einstein"
// "Imagination is more important than knowledge" - Einstein
let dollarSign = "\u{24}"        // $,  Unicode scalar U+0024
let blackHeart = "\u{2665}"      // ♥,  Unicode scalar U+2665
let sparklingHeart = "\u{1F496}" // 💖, Unicode scalar U+1F496
```

##### 扩展字形集群 (Extended Grampheme Clusters)

每个Swift`Character`类型实例代表着一个扩展字形集群。一个扩展字形集群是一系列的一个或多个Unicode标量，并形成一个人类能够读懂的字符。

例如，字母`é`代表这个Unicode标量`é`(`LATIN SMALL LETTER E WITH ACUTE`, 或者 `U+00E9`)。然而这个字母还可以代表一对标量：一个标准的字母`e`(`LATIN SMALL LETTER`，或者`U+0065`)，接着是`COMBINING ACUTE ACCENT`标量(`U+0301`)。`COMBINING ACUTE ACCENT`标量被图形化的应用于在它前面的标量，Unicode文本渲染系统把`e`渲染成`é`。

```swift
let eAcute: Character = "\u{E9}"                         // é
let combinedEAcute: Character = "\u{65}\u{301}"          // e followed by ́
// eAcute is é, combinedEAcute is é
```

扩展字形集群是一个能把很多复杂脚本字符显示为一个`Character`字符的一种灵活的方式。例如，韩文字母的韩文音节可以用合成或者分解序列来表示：

```swift
let precomposed: Character = "\u{D55C}"                  // 한
let decomposed: Character = "\u{1112}\u{1161}\u{11AB}"   // ᄒ, ᅡ, ᆫ
// precomposed is 한, decomposed is 한
```

扩展字形集群还支持封闭标志(例如`COMBINING ENCLOSING CIRCLE`，或者`U+20DD`)：

```swift
let enclosedEAcute: Character = "\u{E9}\u{20DD}"
// enclosedEAcute is é⃝
```

代表区域标志符号的Unicode标量还可以结成对，形成一个`Character`字符。例如这个组合：`REGIONAL INDICATOR SYMBOL LETTER U (U+1F1FA)`和`REGIONAL INDICATOR SYMBOL LETTER S (U+1F1F8)`：

```swift
let regionalIndicatorForUS: Character = "\u{1F1FA}\u{1F1F8}"
// regionalIndicatorForUS is 🇺🇸
Counting Characters
```


#### 字符计数 (Counting Characters)

使用字符串的`characters`的`count`属性来后去字符的个数：

```swift
let unusualMenagerie = "Koala 🐨, Snail 🐌, Penguin 🐧, Dromedary 🐪"
print("unusualMenagerie has \(unusualMenagerie.characters.count) characters")
// Prints "unusualMenagerie has 40 characters"
```

**注意**：在Swift中，使用扩展字形集群形成的字符，在拼接和修改字符串时不一定会影响字符的个数。例如：初始化一个字符串`cafe`，接着在后面追加`COMBINING ACUTE ACCENT (U+0301)`，最终得到的字符串还是只有4个字符，第四个字符是`é`，不是`e`:

```swift
var word = "cafe"
print("the number of characters in \(word) is \(word.characters.count)")
// Prints "the number of characters in cafe is 4"
 
word += "\u{301}"    // COMBINING ACUTE ACCENT, U+0301
 
print("the number of characters in \(word) is \(word.characters.count)")
// Prints "the number of characters in café is 4"
```

**注意：**扩展字形集群可以有一个或多个Unicode标尺组成，这意味着不同的字符和同一个字符的不同展示方式需要不同大小的内存来存储。所以，Swift中的字符不会在字符串的展示中占用相同的内存量。那么，如果不遍历一个字符串，就无法知道这个字符串的字符数，从而无法确定字符的扩展字形集群边界。如果我们要使用特别长的字符串，必须清楚地知道字符串的`characters`属性必须遍历整个字符串的全部Unicode标量来决定字符串的所有字符。

在拥有相同字符得到情况下，`String`的`characters`属性返回的字符数不总是等于`NSString`的`length`属性。`NSString`的长度是基于在字符串的UTF-16显示之内的16-bit代码单元的数字，而不是基于在字符串之内的Unicode扩展字形集群的数字。

#### 访问和修改字符串 (Accessing and Modifying a String)

##### 字符串索引 (String Indices)

每个`String`都有一个关联的索引类型，`String.Index`，对应着字符串里的没一个字符的位置。

就像上面说到的，不同的字符需要占用不用的内存量，所以为了得到字符的位置，我们需要遍历整个字符串的全部Unicode标量。所以Swift的字符串不能按整数值进行索引。

使用`startIndex`来访问字符串的第一个字符的位置，`endIndex`是字符串最后一个字符的下一个位置。所以，`endIndex`不是一个有效的字符串索引。如果一个字符串是空的，那么`startIndex`和`endIndex`是相等的。

使用`index(before:)`和`index(after:)`来访问索引；使用`index(_:offsetBy:)`来访问离一个给定的索引一定距离的索引。

```swift
let greeting = "Guten Tag!"
greeting[greeting.startIndex]
// G
greeting[greeting.index(before:greeting.endIndex)]
// !
greeting[greeting.index(after:greeting.startIndex)]
// u
let index = greeting.index(greeting.startIndex, offsetBy: 7)
// a
```

下面两行代码将会报错，已经超出了字符串的范围：

```swift
greeting[greeting.endIndex] // Error
greeting.index(after:greeting.endIndex) Error
```

使用`characters`的`indices`属性来访问字符串中每一个字符的索引：

```swift
for index in greeting.characters.indices {
	print("\(greeting[index]) ", terminator: "")
}
// Prints "G u t e n   T a g ! "
```

**注意：**我们可以在遵循了`Collection`协议的任何类型中使用`startIndex`/`endIndex`属性和`index(before:)`/`index(after:)`/`index(_:offsetBy:)`方法，包括：`String`、`Array`、`Dictionary`、`Set`。

##### 插入和移除 (Inserting and Removing)

在特定的位置插入一个字符，使用`insert(_:at:)`；在特定的位置插入另外一个字符串内容，使用`insert(contentsOf:at:)`：

```swift
var welcome = "hello"
welcome.insert("!", at: welcome.endIndex)
// welcome 现在是 hello!

welcome.insert(contentsOf: " there".characters, at: welcome.index(before: welcome.endIndex))
// welcome 现在是 hello there!
```

在特定的位置删除一个字符，使用`remove(at:)`，删除一个范围内的字符串，使用`removeSubrange(_:)`:

```swift
welcome.remove(at: welcome.index(before:welcome.endIndex))
// welcome 现在是 hello there

let range = welcome.index(welcome.endIndex, offsetBy: -6)..<welcome.endIndex
welcome.removeSubrange(range)
// welcome 现在是 hello
```

**注意：**我们可以在遵循了`RangeReplaceableCollection`协议的任何类型中使用`insert(_:at:)`、`insert(contentsOf:at:)`、`remove(at:)`和`removeSubrange(_:)`方法，包括：`String`、`Array`、`Dictionary`、`Set`。

#### 字符串的比较 (Comparing Strings)

Swift提供了三种方式来进行文本比较：1）字符串和字符相等；2）前缀相等；3）后缀相等。

##### 字符串和字符相等 (String and Character Equality)

使用等于`==`和不等于`!=`进行比较：

```swift
let quotation = "We're a lot alike, you and I."
let sameQuotation = "We're a lot alike, you and I."
if quotation == sameQuotation {
    print("These two strings are considered equal")
}
// Prints "These two strings are considered equal"
```

如果两个字符串或者两个字符的扩展字形集群正则等价，那么这两个字符串或者字符相等。如果两个字符串或者两个字符有相同的语言意义和外观，那么他们的扩展字形集群相等，即使他们是不同的Unicode标量合成的。例如,`LATIN SMALL LETTER E WITH ACUTE (U+00E9)`与`LATIN SMALL LETTER E (U+0065)`和`COMBINING ACUTE ACCENT (U+0301)`的组合是相等的：

```swift
// "Voulez-vous un café?" using LATIN SMALL LETTER E WITH ACUTE
let eAcuteQuestion = "Voulez-vous un caf\u{E9}?"
 
// "Voulez-vous un café?" using LATIN SMALL LETTER E and COMBINING ACUTE ACCENT
let combinedEAcuteQuestion = "Voulez-vous un caf\u{65}\u{301}?"
 
if eAcuteQuestion == combinedEAcuteQuestion {
    print("These two strings are considered equal")
}
// Prints "These two strings are considered equal"
```

相反，英语的`LATIN CAPITAL LETTER A`(`U+0041`, 或者 `"A"`)，不等于俄语的`CYRILLIC CAPITAL LETTER A`(`U+0410`, 或者 `"А"`)，他们看起来非常相似，但是语言意义不一样：

```swift
let latinCapitalLetterA: Character = "\u{41}"
 
let cyrillicCapitalLetterA: Character = "\u{0410}"
 
if latinCapitalLetterA != cyrillicCapitalLetterA {
    print("These two characters are not equivalent.")
}
// Prints "These two characters are not equivalent."
```

**注意：**Swift中的字符和字符串对区域不敏感。

##### 前缀和后缀相等 (Prefix and Suffix Equality)

使用`hasPrefix(_:)`和`hasSuffix(_:)`来判断一个字符串是否有特定的前缀或者后缀。

```swift
let romeoAndJuliet = [
    "Act 1 Scene 1: Verona, A public place",
    "Act 1 Scene 2: Capulet's mansion",
    "Act 1 Scene 3: A room in Capulet's mansion",
    "Act 1 Scene 4: A street outside Capulet's mansion",
    "Act 1 Scene 5: The Great Hall in Capulet's mansion",
    "Act 2 Scene 1: Outside Capulet's mansion",
    "Act 2 Scene 2: Capulet's orchard",
    "Act 2 Scene 3: Outside Friar Lawrence's cell",
    "Act 2 Scene 4: A street in Verona",
    "Act 2 Scene 5: Capulet's mansion",
    "Act 2 Scene 6: Friar Lawrence's cell"
]

var act1SceneCount = 0
for scene in romeoAndJuliet {
    if scene.hasPrefix("Act 1 ") {
        act1SceneCount += 1
    }
}
print("There are \(act1SceneCount) scenes in Act 1")
// Prints "There are 5 scenes in Act 1"

var mansionCount = 0
var cellCount = 0
for scene in romeoAndJuliet {
    if scene.hasSuffix("Capulet's mansion") {
        mansionCount += 1
    } else if scene.hasSuffix("Friar Lawrence's cell") {
        cellCount += 1
    }
}
print("\(mansionCount) mansion scenes; \(cellCount) cell scenes")
// Prints "6 mansion scenes; 2 cell scenes"

```

#### Unicode表示的字符串 (Unicode Representations of Strings)

当一个Unicode字符串被写入文本文件或者其他内存时，字符串的Unicode标量会被以一种Unicode定义的编码形式编码。每一种形式将字符串编码到一个小方块，也就是代码单元。这些形式包括：1）UTF-8编码形式(把一个字符串编码为8-bit代码单元)；2）UTF-16编码形式(把一个字符串编码为16-bit代码单元)；3）UTF-32编码形式(把一个字符串编码为32-bit代码单元)。

访问其中一种符合Unicode标准表示的字符串：

- UTF-8代码单元集合（使用`utf8`属性）
- UTF-16代码单元集合（使用`utf16`属性）
- 21-bit Unicode标量，也就是字符串的UTF-32编码形式（使用`unicodeScalars`属性访问）

下面的例子将以不同的展示方式来展示这两个字符或字符串：1）一个由`D`、`o`、`g`和`!!`(`DOUBLE EXCLAMATION MARK`，或者Unicode标量`U+203C`)组成的字符串；2）字符：🐶 (`DOG FACE`，或者Unicode标量`U+1F436`)。

##### UTF-8 展示 (UTF-8 Representation)

遍历`utf8`属性来访问字符串的UTF-8展示，这个属性是`String.UTF8View`类型，是无符号8-bit (`UInt8`)值的集合。


![UTF-8 representation](http://upload-images.jianshu.io/upload_images/2057254-29f17d412621540d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```swift
let codeUnit in dogString.utf8 {
    print("\(codeUnit) ", terminator: "")
}
print("")
// 68 111 103 226 128 188 240 159 144 182
```

##### UTF-16 展示 (UTF-16 Representation)

遍历`utf16`属性来访问字符串的UTF-16展示，这个属性是`String.UTF16View`类型，是无符号8-bit (`UInt8`)值的集合。


![UTF-16 Representation](http://upload-images.jianshu.io/upload_images/2057254-bb0145982303feb7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```swift
for codeUnit in dogString.utf16 {
    print("\(codeUnit) ", terminator: "")
}
print("")
// Prints "68 111 103 8252 55357 56374 "
```

##### Unicode标量展示 (Unicode Scalar Representation)

遍历`unicodeScalars`属性来访问字符串的Unicode标量展示。这个属性是`UnicodeScalarView`类型，是一个`UnicodeScalar`类型的值的集合。

每个`UnicodeScalar`有一个`value`属性，这个属性返回标量的21-bit值。


![Unicode Scalar Representation](http://upload-images.jianshu.io/upload_images/2057254-f3ea0f2c08d5560f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```swift
for scalar in dogString.unicodeScalars {
    print("\(scalar.value) ", terminator: "")
}
print("")
// Prints "68 111 103 8252 128054 "
```

作为查询`value`属性的另外一种方法，每个`UnicodeScalar`的值可以用来构建一个新的字符串，如字符串的插值：

```swift
for scalar in dogString.unicodeScalars {
    print("\(scalar)")
}
// D
// o
// g
// ‼
// 🐶
```
