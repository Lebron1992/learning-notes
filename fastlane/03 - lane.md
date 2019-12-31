# 03 - lane
这篇文章我们来学习 fastlane 的一个核心内容 `lane`。每一个 lane 都在项目中的 `Fastfile` 文件定义，并且都跟一个特定的 action 绑定。

## Ruby

有些开发者可能对 ruby 不熟悉，这里以一个 Fastfile 文件为例，讲解一下Ruby 的语法：

```ruby
platform :iOS do

  lane :first do
    archive
    sign
    upload
  end

  lane :second do
    build_app(scheme: "myAmazingApp")
    upload_to_testflight
  end

end
```

在这个例子中，可以明显的看到最外层的 block (第一个 `do` 开始到最有一个 `end` 结束)只在系统是 iOS 时才运行，然后里面包含了两个 lane： `first` 和 `second`。 下面看一下具体代码：
	- 代码中的 `platform` 和 `lane` 是 Ruby 方法，在 ruby 方法中，当没有或者只有一个参数时，可以省略括号，`do-end` 表示一个 block。
	- 代码中的`:iOS`、 `:first` 和 `:second` 是 Ruby 中的 symbol，其实可以看做是 Swift 中的常量。
	- 在每一个 lane 里面的代码是 actions，其实是 ruby 方法，可以根据需求带上参数，并且会按照编写顺序执行。

## 编写 lane

了解以上的 Ruby 语法，我们可以把上一篇文章的 produce action 写入 Fastfile 中，代码如下：

```ruby
default_platform(:ios)

platform :ios do

  lane :register_app do
    produce(
      username: "lebron@test.com",
      app_identifier: "com.lebron.iosapp",
      app_name: "My First App",
      team_name "Lebron Team",
      itc_team_name: "Lebron Team"
    )
  end

end
```

代码中 `produce` 方法可以具体添加哪些参数，可以查看produce 文档 [produce - fastlane docs](https://docs.fastlane.tools/actions/produce/#produce) 后面的 Parameters 部分。

然后在项目的根目录中下，就可以在终端执行 `fastlane register_app` 命令来创建 app。

但是在实际开发中，因为对于一个项目，创建 app 只需要做一次，所以通常不会写成 lane，而是直接在终端使用命令执行。需要重复操作的步骤我们才写成lane。
