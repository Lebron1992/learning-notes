# 04 - Class & Interface

## Class

### 定义 Class

创建 Class 的写法如下：

```typescript
class User {
  firstName: string;
  lastName: string;

  constructor(firstName: string, lastName: string) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
}
```

`constructor` 是构造函数。

### 创建 Class 对象

使用 `new` 关键字：

```typescript
const lebron = new User("Lebron", "James");
```

### 定义函数

定义函数的写法如下：

```typescript
fullName() {
  return `${this.firstName} ${this.lastName}`;
}

describe() {
  console.log(`My name is: ${this.fullName()}`);
}
```

### 访问修饰符 `private` 和 `public`

在编写 Class 时,有时候我们想禁止外部对某些属性进行修改，或者不想让外部访问某些函数，那我们可以使用 `private` 修饰符。例如：

```typescript
class User {
  firstName: string;
  lastName: string;
  private hobbies: string[] = [];

  constructor(firstName: string, lastName: string) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  addHobby(hobby: string) {
    this.hobbies.push(hobby);
  }
}
```

如果我们不加任何修饰符，默认就是 `public`。

### 类初始化的简写形式

按照上面的类的写法，如果一个类有很多属性，那么就要先定义很多属性，然后在构造函数的参数又得重复一遍对应的属性，这样代码看起来就比较臃肿了。 TypeScript 给我们提供了一个简写的方式，写法如下：

```typescript
class User {
  constructor(public firstName: string, public lastName: string) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
}
```

可以不需要先定义属性，只需要在构造函数参数前面加上访问修饰符，就可以自动定义对应的属性。

### 只读属性

有时候类的某些属性我们不想让外部修改，但可以访问，我们可以使用 `readonly`。例如：

```typescript
class User {
  constructor(
    public readonly firstName: string,
    public readonly lastName: string
  ) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
}
```

### 继承

类的继承存在于很多语言中，TypeScript 也有类的继承.

假设有一个类 `Department`：

```typescript
class Department {
  private employees: string[] = [];

  constructor(public readonly id: string, public readonly name: string) {
    this.id = id;
    this.name = name;
  }
}
```

我们可以通过 `extends` 关键字实现继承。例如：

```typescript
class ITDepartment extends Department {
  constructor(id: string, public readonly admins: string[]) {
    super(id, "IT");
    this.admins = admins;
  }
}
```

`ITDepartment` 继承了 `Department` 的所有属性，并且定义了自己的构造函数。在子类的构造函数中，必须调用 `super`，并且需要在给自己的属性赋值之前调用。如果没有自定义构造函数，就会继承父类的构造函数。

#### 访问修饰符 `protected`

在上面的例子中，`employees` 被 `private` 修饰。这意味着 `ITDepartment` 无法访问 `employees`。如果我们想让 `Department` 的所有子类都能访问 `employees`，则可以使用 `protected`。

`protected` 修饰符跟 `private` 类似，区别在于 `protected` 修饰的属性或者函数可以被它的子类访问。

### getter 和 setter

通过 `get` 和 `set` 关键字来定义 getter 和 setter。例如：

```typescript
class ITDepartment extends Department {
  private todoList: string[] = [];

  get latestTodoItem() {
    return this.todoList[0];
  }

  set latestTodoItem(value: string) {
    this.todoList[0] = value;
  }

  constructor(id: string, public readonly admins: string[]) {
    super(id, "IT");
    this.admins = admins;
  }
}

let department = new ITDepartment("1", []);
department.latestTodoItem = "release app to App Store";
console.log(department.latestTodoItem);
```

### 静态属性和函数

直接使用类而不需要实例化一个类就可以调用的属性和函数，就是静态属性和函数。实例化的对象不能调用静态属性和函数。例如：

```typescript
class Department {
  static fisicalYear = 2020;

  static createEmployee(name: string) {
    return { name };
  }
}

Department.createEmployee("Lebron");
console.log(Department.fisicalYear);
```

### 抽象类

在定义类时加上 `abstract` 关键字，就可以把类定义为抽象类。例如：

```typescript
abstract class Department {}
```

抽象类不能被实例化。

抽象函数的写法如下：

```typescript
abstract class Department {
  abstract describe(): void;
}
```

在函数的前面加上 `abstract`，不能有函数实现，并且需要指定返回值。

继承抽象类的子类必须实现抽象函数：

```typescript
class ITDepartment extends Department {
  describe() {
    console.log(`IT Department: id is ${this.id}`);
  }
}
```

### 单例和私有构造函数

通过把构造函数设置为 `private` 和 静态属性和函数，我们可以实现单例：

```typescript
class ITDepartment extends Department {
  private static instance: ITDepartment;

  static getInstance() {
    if (this.instance) {
      return this.instance;
    }
    this.instance = new ITDepartment("1", []);
    return this.instance;
  }

  private constructor(id: string, public readonly admins: string[]) {
    super(id, "IT");
    this.admins = admins;
  }
}
```

## Interface

### 定义 Interface

使用 `interface` 关键字定义 Interface。例如：

```typescript
interface Greetable {
  name: string;

  greet(phrase: string): void;
}
```

Interface 中的属性不能设置默认值；函数不能有具体实现，并且要指定返回值。

### Class 实现 Interface

实现 Interface 的 class 中，必须包含 Interface 中定义的属性和函数。例如：

```typescript
class Person implements Greetable {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  greet(phrase: string) {
    console.log(`${phrase} ${this.name}`);
  }
}
```

如果要实现多个 Interfaces，可以用逗号分隔 interface。例如：

```typescript
class Person implements Greetable, AnotherInterface {}
```

### Interface 的继承

与 Class 一样，Interface 也可以继承。例如：

```typescript
interface Named {
  name: string;
}

interface Greetable extends Named {
  greet(phrase: string): void;
}
```

而且支持多继承，但是 Class 不行。例如：

```typescript
interface Greetable extends Named, AnotherInterface {}
```

### 可选的属性和参数

在定义 Interface 时，有可能有些属性不要求必须实现。我们可以这样写：

```typescript
interface Named {
  name: string;
  outputName?: string;
}
```

同样地，也可以把函数标记为可选。例如：

```typescript
interface Named {
  name: string;
  outputName?: string;

  myMethod?(): void;
}
```

函数的参数也可以标记为可选。例如：

```typescript
class Person implements Greetable {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  greet(phrase: string) {
    console.log(`${phrase} ${this.name}`);
  }
}
```

### 更多 Interface 相关信息

- ![Interfaces](https://www.typescriptlang.org/docs/handbook/interfaces.html)
