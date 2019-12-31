# 02 - 创建 app

Fastlane 的功能非常丰富，教程中不可能涉及到所有的功能，很多时候还是需要自己去看文档。所以在这篇文章，我以创建一个新的app为例，详细讲解下如何使用 fastlane 的文档。

## Actions

每当要把一个手动的任务变成自动化，我们需要使用 fastlane 中与任务对应的工具，这个工具就叫做 action。

打开文档主页[fastlane docs](https://docs.fastlane.tools/)，在左边的侧栏找到 `Available Actions`，点击进去，可以看到很多不同的分类，以后遇到新的自动化需求可以在这里查找。

## produce action

在这篇文章中，我们使用一个 action 来创建 app，这个 action 叫做 `produce`， 属于 `App Store Connect`分类。找到它之后，点击进去可以看到具体文档。`produce` action 其实是 `create_app_online` 的别名。

下面我们在终端应用程序中尝试一下这个 action：

```
# 1. 执行 fastlane produce，这是执行一个 action 的方法：fastlane "action 名称"
fastlane produce

# 2. 提示: Your Apple ID Username (让你输入 Apple Id)
lebron@test.com

# 3. 如果你苹果开发者账号下有多个 team，则选择你需要创建 app 的 team

# 4. 提示: App Identifier (Bundle ID, e.g. com.krausefx.app) (让你输入 bundle id)
com.lebron.iosapp

# 5. 提示: App Name (让你输入应用名称)
My First App

# 6. 如果你的 App Store Connect 账号下有多个 team，则选择你需要创建 app 的 team

# 7. 最终创建成功，输出 app id
```

上面的例子中，我们执行的是最原始的命令，他会提示你输入账号和 app 的各项具体信息。其实我们可以在执行 `fastlane produce` 时带上所有参数。可以通过查看网页文档或者在终端输入 `fastlane produce --help` 来查看如何加上参数。看完文档后，我们可以把上面的例子写成一个命令：

```ruby
fastlane produce \
--username lebron@test.com \
--app_identifier com.lebron.iosapp \
--app_name "My First App" \
--team_name "Lebron Team" \
--itc_team_name "Lebron Team"
```

`\` 意思是把下一行的命令连接起来。

另外，还可以执行 `fastlane actions produce` 来更好的查看各项参数的具体描述。
