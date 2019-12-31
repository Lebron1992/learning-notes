# 01 - The M Word

```ruby
class Greeting
	def initialize(text) 
		@text = text 
	end

	def  welcome  
		@text
	end
end

my_object = Greeting.new("Hello")
```
- 获取类名：`my_object.class     # => Greeting`
- 获取实例方法：`my_object.class.instance_methods(false)    # => [:welcome]`，参数表示是否包含继承的实例方法
- 获取实例变量：`my_object.instance_variables    # => [:@text]`

**元编程是编写能在运行时操作语言构件的代码。** 
