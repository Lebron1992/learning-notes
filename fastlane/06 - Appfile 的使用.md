# 06 - Appfile 的使用

在上一篇文章中，我们把 `cert` 和 `sigh` 一起使用，代码如下：

```ruby
lane :sanbox do
  cert(
    username: "lebron@test.com",
    team_name: "Lebron Team",
    development: true
  )
  sigh(
    username: "lebron@test.com",
    team_name: "Lebron Team",
    app_identifier: "com.lebron.iosapp”,
	  development: true
  )
end
```

我们发现这里有很多参数是重复的。这篇文章的 Appfile 就是用来解决这个问题的。Appfile 的文档：[Appfile - fastlane docs](https://docs.fastlane.tools/advanced/Appfile/#appfile)

在 Appfile 中保存相同的参数，代码如下：

```ruby
team_id "Q2CBPJ58CA" # team_name 为 Lebron Team 的 id
apple_id "lebron@test.com"
app_identifier "com.lebron.iosapp"
```

然后 Fastfile 可以修改为：

```ruby
lane :sanbox do
  cert(development: true)
  sigh(development: true)
end
```
