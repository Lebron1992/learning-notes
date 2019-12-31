# 05 - Class Definitions

## 1. 在 Java 和 C# 中，定义了一个类之后，如果不创建实例并访问它的成员，实际上并不会运行任何代码；而在 Ruby 中，使用 class 定义类的时候，它真的是在运行代码。

```ruby
class MyClass
	puts 'Hello World'
end
# => Hello World
```

## 2. 定义类的时候，就像 methods 和 blocks 一样，也会返回最后一行代码的值：

```ruby
result = class MyClass
	self
end
result   # => MyClass
```

## 3. `class_eval()`  （`module_eval()`） : 在一个 class 的环境下运行 block。

```ruby
def add_method_to(a_class)
	a_class.class_eval do
		def m
			'Hello!'
		end
	end
end

# 使用 instance_eval 定义单件类
s1 = "abc"
s2 = "def"
s1.instance_eval do
	def swoosh!
		reverse
	end
end
s1.swoosh!      				# => "cba"
s2.respond_to?(:swoosh!)     # => false
```

- `instance_eval()` 只改变`self`(其实也会修改当前类，会把当前类修改为接收者的单件类)，而 `class_eval()`改变 `self` 和当前的类
- 和  `instance_eval()` 一样，也会有对应的 `class_exec()`
- 如果想 open a object, 使用 `instance_eval()`；如果想 open a class，使用 `class_eval()`

## 4. Class Instance Variables

```ruby
class MyClass
	@my_var = 1   # an instance variable of the `MyClass` object

	def self.read
		@my_var
	end

	def write
		@my_var = 2    # an instance variable of the `obj` object
	end

	def read
		@my_var
	end
end

obj = MyClass.new
obj.read        # => nil
obj.write
obj.read        # => 2
MyClass.read    # => 1
```

## 5. 给单个对象增加一个方法：

```ruby
str = "just a regular string"
def str.title?
	self.upcase == self
end
```

这段代码为`str` 添加了一个 `title?`方法。其他对象（即使是 String 对象）没有这个方法。只对单个对象生效的方法，称为**单件方法（Singleton Method）**。

## 6. 类方法的本质其实就是单件方法。

## 7. 像 `attr_accessor`这样的方法成为**类宏（Class Macro）**。

## 8. 单件类是一个对象的单件方法的存活之所。

## 9. 如果把单件类、普通类和模块放在一起，Ruby 对象模型一共有七条规则：

	- 只有一种对象 ———— 要么是普通对象，要么是模块。
	- 只有一种模块 ———— 可以是一个普通模块、一个类或者一个单件类。
	- 只有一种方法，它存在于一个模块中 ———— 通常是在一个类中。
	- 每个对象（包括类）都有自己的 “真正的类” ———— 要么是普通类，要么是单件类。
	- 除了 BasicObject 类没有超类外，每个类有且只有一个祖先  ———— 要么是一个类，要么是一个模块。这意味着任何类只有一条向上的、直到 BasicOjbect 的祖先链。
	- 一个对象的单件类的超类是这个对象的类；一个类的单件类的超类是这个类的超类的单件类。
	- 调用一个方法是，Ruby 先向右迈一步进入接收者真正的类，然后向上进入祖先链。这就是Ruby 的查找方法的方式。

## 10. 类方法的三种语法

```ruby
def MyClass.a_class_method
end

class MyClass
	def self.another_class_method
	end
end

class MyClass
	class << self
		def yet_another_class_method
		end
	end
end
```

一般不使用第一种，因为这种方式重复了类的名字，会给重构带来不便；一般采用第二种；而第三种方式明确表明单件类才是类方法真正的所在之处。

## 11. 当一个类包含一个模块时，他获得的是该模块的实例方法，而不是类方法。例如：

- Class Extension

```ruby
module MyModule
	def self.my_method
		'hello'
	end
end
class MyClass
	include MyModule
end
MyClass.my_method   # NoMethodError!

# 解决办法
class MyClass
	class << self
		include MyModule
	end
end
# 或者
class MyClass
	extend MyModule
end

MyClass.my_method   #  => "hello"

# 这种技巧称为 Class Extension。
```

- Ojbect Extension

```ruby
module MyModule
	def my_method
		'hello'
	end
end

obj = Object.new

class << obj
	include MyModule
end
# 或者
obj.extend MyModule

obj.my_method           # => "hello"
obj.singleton_methods   # => [:my_method]
```

## 12. 方法别名

```ruby
class MyClass
	def my_method
		'my_method()'
	end
	alias_method :m, :my_method
end
obj = MyClass.new
obj.my_method           # => "my_method()"
obj.m			          # => "my_method()"
```

Ruby还提供了一个 `alias` 关键字，可以替代 `Module#alias_method` 方法。当你要在顶级作用域中修改方法时，需要使用`alias`，因为`Module#alias_method`方法不可用。

## 13. 可以通过如下三步来编写 Around Alias：

- 给方法定义一个别名
- 重新定义这个方法
- 在新的方法中调用老的方法

Around Alias 的一个缺点在于它污染了你的类，为他添加了一个额外的名字。要解决这个问题，可以在添加别名之后，想办法把老版本的方法变成私有。
另外一个潜在危险与加载有关。除非你希望在调用方法时出现异常，否则永远不要尝试加载两次Around Alias。
Around Alias最主要的问题在于它的猴子不定。像所有的猴子补丁一样，它有可能破坏已有的代码。

## 14. 更多的方法包装器

- 细化包装器（Refinement Wrapper）：作用范围只到文件的末尾处

```ruby
module StringRefinement
	refine String do
		def length
			super > 5 ? 'long' : 'short'
		end
	end
end

using StringRefinement
"War and Peace".length    # => "long"
```

- 下包含包装器（Prepended Wrapper）：与细化包装器相比，它不是一种局部化的技巧，但是一般认为它比细化包装器和环绕别名都更清晰。

```ruby
module ExplicitString
	def length
		super > 5 ? 'long' : 'short'
	end
end
String.class_eval do
	prepend ExplicitString
end

"War and Peace".length    # => "long"
```
