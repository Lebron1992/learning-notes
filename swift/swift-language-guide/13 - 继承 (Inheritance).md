# 继承 (Inheritance)

## 定义基类 (Defining a Base Class)

没有继承其他类的类，叫做基类。

**注意：** Swift的类不用继承一个通用的基类。

下面是定义一个`Vehicle`基类，定义了任意机动车的共同特征：

```swift
class Vehicle {
    var currentSpeed = 0.0
    var description: String {
        return "traveling at \(currentSpeed) miles per hour"
    }
    func makeNoise() {
        // do nothing - an arbitrary vehicle doesn't necessarily make a noise
    }
}

let someVehicle = Vehicle()

print("Vehicle: \(someVehicle.description)")
// Vehicle: traveling at 0.0 miles per hour

```

## 子类化 (Subclassing)

子类继承了父类的特征，子类也可以添加新的特征。

继承的通用形式：

```swift
class SomeSubclass: SomeSuperclass {
    // subclass definition goes here
}
```

下面是定义了一个`Vehicle`的子类`Bicycle`:

```swift
class Bicycle: Vehicle {
    var hasBasket = false
}
```

`Bycycle`自动获得了`Vehicle`的`currentSpeed`和`description`属性，还有`makeNoise()`方法。另外还添加了一个存储属性`hasBasket`,默认值为`false`（被推断为Bool类型）。

默认情况下自行车是没有篮子的，创建一个自行车实例后，把`hasBasket`设置为`true`：

```swift
let bicycle = Bicycle()
bicycle.hasBasket = true
```

还可以修改通过继承得到的属性：

```swift
bicycle.currentSpeed = 15.0
print("Bicycle: \(bicycle.description)")
// Bicycle: traveling at 15.0 miles per hour
```

子类又可以被子类化。下面创建了一个`Bicycle`的子类两轮自行车`Tandem`:

```swift
class Tandem: Bicycle {
	var currentNumberOfPassangers = 0
}

let tandem = Tandem()
tandem.hasBasket = true
tandem.currentNumberOfPassengers = 2
tandem.currentSpeed = 22.0
print("Tandem: \(tandem.description)")
// Tandem: traveling at 22.0 miles per hour
```

`Tandem`继承了`Bicycle`的所有属性和方法，并依次继承了`Vehicle`的所有属性和方法，还另外添加了一个属性`currentNumberOfPassengers`，默认为0。

## 重写 (Overriding)

子类可以自定义从父类继承的实例方法、类方法、实例属性、类属性或者下标的实现，称为重写。使用`override`关键字进行重写。

### 访问父类的方法、属性和下标 (Accessing Superclass Method, Properties and Subscripts)

当重写父类的方法、属性或者下标时，把已经存在的父类实现作为重写的一部分是非常有用的。

使用`super`来访问父类的方法、属性或者下标的实现：

- 子类的`someMethod()`方法可以通过`super.someMethod()`来访问父类的`somemMethod()`实现。
- 子类的`someProperty`属性可以通过`super.someProperty`来访问父类的`someProperty`属性。
- 子类的`someIndex`下标可以通过`super[someIndex]`来访问父类的同一个下标实现。

### 重写方法 (Overriding Methods)

创建`Vehicle`的一个子类`Train`，并重写`makeNoise()`方法：

```swift
class Train: Vehicle {
    override func makeNoise() {
        print("Choo Choo")
    }
}
```

创建一个`Train`实例，并调用`makeNoise()`方法，实际调用的是子类的版本：

```swift
let train = Train()
train.makeNoise()
// Prints "Choo Choo"
```

### 重写属性 (Overriding Properties)

我们可以重写通过继承得到的实例属性和类属性，或者添加属性观察者来监测属性值的变化。

### 重写属性的getter和setter方法 (Overriding Property Getters and Setters)

子类是不知道通过继承得到的存储属性和计算属性的本质，他只知道这些属性的名字和类型。

我们可以把继承得到的只读属性重写为可读可写属性，但是不能把继承得到的可读可写属性重写范围只读属性。

**注意：** 如果在重写属性时，提供了setter方法，我们必须也提供一个getter方法。在重写getter方法时，如果不想改变继承得到的属性值，我们可以在getter方法中返回`super.someProperty`，`someProperty`是正在重写的属性名字。

```swift
class Car: Vehicle {
    var gear = 1
    override var description: String {
        return super.description + " in gear \(gear)"
    }
}
```

### 重写属性观察者 (Overriding Property Observers)

我们可以使用属性重写来把属性观察者添加到继承得到的属性中。当继承得到的属性值发生改变时，我们可以做出响应。

**注意：** 不能把属性观察者添加到继承得到的常量存储属性或者只读计算属性。因为这些属性不能被设置新的值，所有不能使用`willSet`和`didSet`观察者。另外，也不能在重写属性时同时重写setter方法和属性观察者。如果要监测属性值的变化，可以在自定义的setter方法监测。

```swift
class AutomaticCar: Car {
	override var currentSpeed: Double {
		didSet {
			gear = Int(currentSpeed / 10.0) + 1
		}
	}
}

let automatic = AutomaticCar()
automatic.currentSpeed = 35.0
print("AutomaticCar: \(automatic.description)")
// AutomaticCar: traveling at 35.0 miles per hour in gear 4
```

当设置`currentSpeed`属性时，`didSet`观察者内部给`gear`设置了一个新的值。

## 防止重写 (Preventing Overrides)

我们可以在定义方法、属性或者下标时，在最前面加上`final`关键字来阻止子类重写(例如`final var`、`final func`、`final class func`、`final subscript`)。

如果尝试修改`final`标记的方法、属性或者下标，会报编译错误。在扩展(Extension)中定义的方法、属性或者下标都可以使用`final`。

在定义类时，在`class`前面加上`final`来阻止这个类被继承。
