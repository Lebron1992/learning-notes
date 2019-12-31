# 05 - Provisioning

Fastlane 有一个 action 用于 provisioning profile 的管理，叫做 `sigh`，是 `get_provisioning_profile` 的别名。文档链接：[sigh - fastlane docs](https://docs.fastlane.tools/actions/sigh/#sigh)

`sigh` 通常与 `cert` 一起使用，它支持所有四种 provisioning profiles 类型。

lane的写法如下：

```ruby
lane :sanbox do
  cert(
    username: "lebron@test.com",
    team_name: "Lebron Team",
    development: true # 指定生产还是开发版本，默认是生产
  )
  sigh(
    username: "lebron@test.com",
    team_name: "Lebron Team",
    app_identifier: "com.lebron.iosapp”,
	  development: true
  )
end
```
