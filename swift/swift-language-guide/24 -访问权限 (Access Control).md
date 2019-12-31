### 【Swift 3.1】24 -访问权限 (Access Control)

访问权限可以限制我们的部分代码被其他文件或模块访问。这可以隐藏有些代码的实现细节。我们可以指定类型（如类、结构和枚举）的访问权限，也可以指定属性、方法、初始化器和下标的访问权限。

Swift也提供了默认的访问权限来减少写明访问权限的需要。

#### 模块和源文件 (Modules and Source Files)

Swift的访问控制模型是基于模块和源文件的概念。

一个模块是一个代码分配单元，一个框架或应用就是一个代码单元，可以使用`import`关键字被其他模块导入。

在Swift中，在Xcode的每个构建版本(例如一个应用程序包或者框架)都被认为是一个分离的模块。

一个源文件是一个模块里面的代码文件(实际上就是应用或框架里的一个文件)。虽然通常来说把每个独立的类型写在不同的源文件中，但是一个源文件可以包含多个类型、方法等等。

**注意：**和访问权限相关的属性、类型和方法等等都统称为实体(entities)。

#### 访问级别 (Access Levels)

Swift提供了5个访问级别，这些访问级别与源文件和模块有关：

- `open`和`public`修饰的实体可以被模块内的任何文件访问，也可以被导入了这个模块的另一个模块的文件使用。当我们定义框架的公开接口时，通常使用`open`或者`pubilc`。至于这两者的区别，下面会讲到。
- `internal`修饰的实体可以被模块内的任何文件访问，模块之外的其他文件不能访问。当定义应用或者模块内的结构时，通常使用`internal`。
- `fileprivate`修饰的实体可以在这个文件内被访问。当实现细节只在文件内使用时，通常使用`fileprivate`对其他文件隐藏实现细节。
- `private`修饰的实体只能被实体所在的声明内部访问。当实现细节只在声明内部使用时，使用`private`。

`open`访问权限是最高的，而`private`访问权限是最低的。

`open`访问权限只适用于class和class的成员，与`public`的不同在于：

- `public`修饰的class，或被其他更严格的访问级别修饰，这个class只能在当前模块内被继承。
- `public`修饰的class成员，或被其他更严格的访问级别修饰，这些成员只能被当前模块内的子类重写。
- `open`修饰的class，能在当前模块内被继承，也可以在导入了这个模块的另一个模块内被继承。
- `open`修饰的class成员，能被当前模块内的子类重写，也可以被导入了这个模块的另一个模块内的子类重写。

##### 访问级别指导原则 (Guiding Principle of Access Levels)

Swift的访问级别遵循一个总的原则：一个实体不能定义在访问权限比它更低的实体里面。例如：

- 一个`public`变量不能定义在`internal`、`fileprivate`或者`private`修饰的类型中。
- 一个方法的访问权限不能高于他们参数和返回值类型

##### 默认访问权限 (Default Access Levels)

如果我们没有明确指定实体的权限，代码中的所有实体都有一个默认的权限`internal`。

##### 单目标应用的访问级别 (Access Levels for Single-Target Apps)

当我们编写一个单目标应用时，我们不必让外部访问应用的模块。默认的`internal`权限都已经满足要求。所以我们不必自定义访问级别。但是，有是有我们需要使用`fileprivate`或者`private`来隐藏一些功能实现细节。

##### 框架的访问级别 (Access Levels for Frameworks)

当我们在开发一个框架时，使用`open`或者`public`来标记那些想要公开的API，其他模块导入这个框架就可以访问公开的API。

##### 单元测试目标的访问级别 (Access Levels for Unit Test Targets)

当使用单元测试目标编写应用时，为了测试，应用的代码需要向那个模块公开。默认情况下，只有`open`和`public`修饰的实体才可以被其他模块访问。然而，如果我们在导入产品模块时，使用`@testable`属性标记，单元测试目标可以访问`internal`修饰的实体。

#### 访问权限语法 (Access Control Syntax)

各个访问级别的语法如下：

```swift
public class SomePublicClass {}
internal class SomeInternalClass {}
fileprivate class SomeFilePrivateClass {}
private class SomePrivateClass {}
 
public var somePublicVariable = 0
internal let someInternalConstant = 0
fileprivate func someFilePrivateFunction() {}
private func somePrivateFunction() {}
```

下面两个实体的访问权限默认是`internal`：

```swift
class SomeInternalClass {}              // implicitly internal
let someInternalConstant = 0            // implicitly internal
```

#### 自定义类型 (Custom Types)

一个类型的访问权限会影响类型成员（如类型的属性、方法、初始化器和下标）的默认权限。如果类型的访问权限为`private`或者`fileprivate`，那么他的成员的默认权限是`private`或者`fileprivate`；如果类型的权限是`internal`或者`public`，那么他的成员的默认权限是`internal`。

**注意：**一个`public`类型，他的成员默认是`internal`，如果想让成员的权限也是`public`，我们必须明确写出。

```swift
public class SomePublicClass {                  // class明确标记为public
    public var somePublicProperty = 0            // class成员明确标记为public
    var someInternalProperty = 0                 // class成员默认是internal
    fileprivate func someFilePrivateMethod() {}  // class成员明确标记为fileprivate
    private func somePrivateMethod() {}          // class成员明确标记为private
}
 
class SomeInternalClass {                       // class默认是internal
    var someInternalProperty = 0                 // class成员默认是internal
    fileprivate func someFilePrivateMethod() {}  // class成员明确标记为fileprivate
    private func somePrivateMethod() {}          // class成员明确标记为private
}
 
fileprivate class SomeFilePrivateClass {        // class明确标记为fileprivate
    func someFilePrivateMethod() {}              // class成员默认是fileprivate
    private func somePrivateMethod() {}          // class成员明确标记为private
}
 
private class SomePrivateClass {                // class明确标记为private
    func somePrivateMethod() {}                  // class成员默认是private
}
```

##### 多元组类型 (Tuple Types)

多元组的访问权限是以多元组内元素访问权限最低的为准。例如，一个元素的权限是`internal`，另一个是`private`，那么这个多元组的访问权限是`private`。

##### 方法类型 (Function Types)

方法的访问权限是以参数类型和返回值类型的最低权限为准。如果参数类型和返回值类型的最低权限不能满足要求，需要我们明确写出方法的权限。

下面是一个例子：

```swift
func someFunction() -> (SomeInternalClass, SomePrivateClass) {
    // function implementation goes here
}
```

实际上这个例子是不能编译通过的。方法的返回值是一个多元组，其中一个元素的权限为`internal`，另一个是`private`，所以多元组的权限是`private`。因为返回值的权限是`private`，所以必须明确写出方法的权限是`private`：

```swift
private func someFunction() -> (SomeInternalClass, SomePrivateClass) {
    // function implementation goes here
}
```

把`someFunction()`标记为`public`或者`internal`，或者使用默认的`internal`权限都是不行的。

##### 枚举类型 (Enumeration Types)

枚举的每一个case的权限自动地与枚举的权限相同，我们不能为单个case定义不同的权限。

下面的`CompassPoint`被标记为`public`，那么每个case的权限也是`public`。

```swift
public enum CompassPoint {
    case north
    case south
    case east
    case west
}
```

###### 原始值和关联类型 (Raw Values and Associated Values)

枚举的原始值和关联类型的访问权限不能低于枚举的访问权限。例如，不能把`private`修饰的类型作为`internal`修饰的枚举的关联值类型。

##### 嵌套类型 (Nested Types)

在`private`类型里定义的嵌套类型，那么嵌套类型的权限也是`private`；在`fileprivate`类型里定义的嵌套类型，那么嵌套类型的权限也是`fileprivate`；在`public`或者`internal`类型里定义的嵌套类型，那么嵌套类型的权限也是`internal`。如果想要嵌套类型是`public`的，那么需要明确使用`public`修饰。

#### 子类化 (Subclassing)

子类的权限不能高于父类的权限。

下面是一个例子，`B`重写了父类的`someMethod()`方法，并且权限是`internal`，高于这个方法在父类的权限`fileprivate`：

```swift
public class A {
	fileprivate func someMethod() {}
}

internal class B: A {
    override internal func someMethod() {}
}
```

子类成员可以调用父类的比子类成员访问权限更低的成员，只要调用父类成员的位置满足父类成员的权限要求（也就是说，在同一个源文件内调用父类的`fileprivate`成员，或者是在同一个模块内调用`internal`成员）。

```swift
public class A {
    fileprivate func someMethod() {}
}
 
internal class B: A {
    override internal func someMethod() {
        super.someMethod()
    }
}
```

因为父类`A`和子类`B`定义在同一个文件，所以在`B`的`someMethod()`方法中调用`super.someMethod()`是有效的。

#### 常量、变量、属性和下标 (Constants, Variables, Properties and Subscripts)

常量、变量或者属性的权限不能高于他们的类型。

如果常量、变量、属性或者下标的类型是`private`，那么常量、变量、属性或下标也必须用`private`修饰：

```swift
private var privateInstance = SomePrivateClass()
```

##### Getters and Setters

常量、变量、属性和下标的getter和setter方法的权限，默认情况下与常量、变量、属性和下标的权限相同。

我们可以把setter的权限设置成低于对应getter的权限。可以用`fileprivate(set)`、`private(set)`或者`internal(set)`来设置更低的权限。

**注意：**这个规则适用于存储属性和计算属性。虽然我们没有明确写出存储属性的getter和setter，但是Swift会默认提供的。

例如下面这个例子：

```swift
struct TrackedString {
	private(set) var numberOfEdits = 0
	var value: String = "" {
		didSet {
			numberOfEdits += 1
		}
	}
}
```

`numberOfEdits`属性被`private(set)`修饰，setter的权限是`private`，所以`numberOfEdits`只能在`TrackedString`内部被修改，对于`TrackedString`外部来说，`numberOfEdits`是一个只读属性。而getter的权限默认是`TrackedString`一样，为`internal`。

创建`TrackedString`实例，并修改`value`属性：

```swift
var stringToEdit = TrackedString()
stringToEdit.value = "This string will be tracked."
stringToEdit.value += " This edit will increment numberOfEdits."
stringToEdit.value += " So will this one."
print("The number of edits is \(stringToEdit.numberOfEdits)")
// Prints "The number of edits is 3"
```

`numberOfEdits`能被外部访问，但是不能被修改。

注意：我们可以分别设置getter和setter的权限。下面的例子是`TrackedString`的另外一个版本，被`public`修饰，所以这个结构的成员默认是`internal`的。我们可以结合`public`和`private(set)`使得`umberOfEdits`属性的getter是`public`，而`setter`是`private`。

```swift
public struct TrackedString {
    public private(set) numberOfEdits = 0
    public var value: String = "" {
        didSet {
            numberOfEdits += 1
        }
    }
    public init() {}
}
```

#### 初始化器 (Initializers)

自定义初始化器的权限可以低于或等于类型的权限，除了`required`初始化器。`required`初始化器的权限只能等于类型的初始化器。

对于方法和参数，初始化器的参数权限不能低于初始化器的权限。

##### 默认初始化器 (Default Initializers)

默认初始化器的权限与它要初始化的类型权限相同，除非这个类型的权限是`public`。对于`public`的类型，他的默认初始化器的权限是`internal`。如果想让其他模块能使用`public`类型的无参数的初始化器，我们必须使用`public`标记无参数的初始化器。

##### 结构类型的默认逐一成员初始化器 (Default Memberwise Initializers for Structure Types)

如果结构的任何存储属性都是`private`的，那么默认逐一成员初始化器也是`private`的；如果结构的任何存储属性都是`fileprivate`的，那么默认逐一成员初始化器也是`fileprivate`的。否则默认逐一成员初始化器是`internal`的。

如果想让默认逐一成员初始化器是`public`的，我们必须自己定义一个逐一成员初始化器，并用`public`修饰。

#### 协议 (Protocols)

如果想用访问级别修饰协议，那么需要在定义协议的时候写上访问级别。

协议里的每一个要求的权限默认是与当前协议权限相同，并且不能设置成与当前协议不一样的权限。

**注意：**如果定义了一个`public`协议，那么协议里面的所有要求的权限也是`public`。这个规则不同于其他类型，如果其他类型定义为`public`，那么它的成员默认是`internal`的。

##### 协议继承 (Protocol Inheritance)

如果定义了一个新的协议继承于一个已经存在的协议，新的协议权限不能高于已经存在的协议。不能定义一个`public`协议继承于`internal`协议。

##### 协议一致性 (Protocol Conformance)

一个类型可以遵循权限比它低的协议。例如，可以定义一个`public`类型，然后遵循一个`internal`协议。

如果一个类型是`public`的，遵循于`internal`协议，那么这个协议在`public`类型的实现也是`internal`的。

#### 扩展 (Extensions)

在扩展中新添加的类型成员的权限默认与原类型定义的成员权限相同。如果扩展了一个`public`或者`internal`的类型，那么新添加的成员权限是`internal`；如果扩展了一个`fileprivate`的类型，那么新添加的成员权限是`fileprivate`；如果扩展了一个`private`的类型，那么新添加的成员权限是`private`。

同样地，我们可以明确的指定扩展的访问权限(例如使用private extension)，那么新指定的权限将会替代原类型的默认权限。

##### 使用扩展遵循协议 (Adding Protocol Conformance with an Extension)

如果我们使用扩展来遵循协议，那么我们不能明确指定扩展的权限。扩展对协议要求的实现的权限与协议的权限相同。

#### 泛型 (Generics)

泛型类型和泛型方法的访问权限，是泛型类型或者泛型方法与类型约束的最低权限。

#### 类型别名 (Type Aliases)

类型别名的权限可以小于或等于它原本的类型。例如，`private`的类型别名可以是`private`、`fileprivate`、`internal`、`public`或者`open`类型的别名。

**注意：**这个规则也适用于关联值类型的类型别名。
