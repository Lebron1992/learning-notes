# 04 - Blocks

1. Block 的基本使用

```ruby
def a_method(a, b)
  a + yield(a, b) # yield 代表传进来的 block
end
a_method(1, 2) { |x, y| (x + y) * 3 } # => 10

# 判断是否传入了 block
def a_method
	return yield if block_given?
	'no block'
end
```

2. 全局变量

```ruby
def a_scope
	$var = 'some value'
end

def another_scope
	$var
end

a_scope
another_scope # => 'some value'
```

3. Flat Scope：可以用`Class.new` 替换 `class`，`Module.new` 替换 `module`，`define_method` 替换 `def`。

```ruby
# 通过这样修改，class 定义里面可以访问到外部的变量
my_var = 'Success'

MyClass = Class.new do 
	"#{my_var} in the class definition"

	define_method :my_method do
	  "#{my_var} in the method"		
	end
end
```

4. Shared Scope

```ruby
def define_methods
	shared = 0
	
	Kernel.send :define_method, :counter do
		shared
	end

	Kernel.send :define_method, :inc do |x|
		shared += x
	end
end

define_methods
counter  # => 0
inc(4)
counter  # => 4
```

5. `instance_eval()`

```ruby
class MyClass
	def initialize
		@v = 1
	end
end

obj = MyClass.new
obj.instance_val do
	# 执行的时候把 `self` 作为 receiver
	self     # => #<MyClass:0x3340dc @v=1>
	@v       # => 1
end

# instance_val 可以访问它被定义时所在的作用域
v = 2
obj.instance_val { @v = v }
obj.instance_val { v }   # => 2
```

6. `instance_exec()` : 允许传入携带参数的 block

```ruby
Class C
	def initialize
		@x = 1
	end
end

class D
	def twisted_method
		@y = 2
C.new.instance_exec(@y) { |y| "@x: #{@x}, @y: #{y}" }
	end
end

D.new.twisted_method     # => "@x: 1, @y: 2"
```

7. Proc Objects

	- Proc
```ruby
inc = Pro.new { |x| x + 1 }
inc.call(2)   # => 3
```
	- lambda
```ruby
dec = lambda { |x| x - 1 }
dec.class     # => Proc
dec.call(2)   # => 1

# 简写形式
dec = ->(x) { x - 1 }
```

- **&** 运算符：block 与 Proc 互相转换

```ruby
# block 转换成 Proc
def math(a, b)
	yield(a,b )
end
def do_math(a, b, &operation)
	math(a, b, &operation)
end
do_math(2, 3) { |x, y| x * y }  # => 6

# Proc 转换成 block
def my_method(greeting)
	"#{greeting}, #{yield}!"
end
my_proc = proc { 'Bill' }
my_method('Hello', &my_proc)
```

- Procs vs. Lambdas：1）`return`: 在 Lambdas 中,只是返回 lambda 的结果； 在Proc 中，从定义它的作用域返回；2）检查传入的参数规则不同: lambda 要求传入的参数个数要与所需参数相同，否则报错 `ArgumentError`；而 Proc 则无要求；3）一般优先选择 lambda。

```ruby
def double(callable_object)
	callable_object.call * 2
end
l = lambda { return 10 }
double(l)   # => 20

def another_double
	p = Proc.new { return 10 }
	result = p.call
	# 下面这行代码不会运行；如果 `Proc.new { return 10 }`改为 `Proc.new { 10 }` 则会运行
	return result * 2
end
```

```ruby
p = Proc.new { |a, b| [a, b] }
p.arity  # => 2
p.call(1, 2, 3)     # => [1, 2]
p.call(1)           # => [1, nil]
```
