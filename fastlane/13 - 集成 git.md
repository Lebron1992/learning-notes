# 13 - 集成 git

Fastlane 很好地支持了 git。与 git 相关的 action：[Source-Control - fastlane docs](https://docs.fastlane.tools/actions/#source-control)

下面是一下常用的 actions：

```ruby
lane :build_appstore do
    # 集成 git
    ensure_git_status_clean # 保证没有未提交的代码
    ensure_git_branch(branch: "master") # 保证在 master 分支
    git_pull # 更新到最新的代码
	  increment_build_number
    gym(
      output_directory: "build_Appstore",
      export_method: "app-store"
    )
    commit_version_bump( # 提交 increment_build_number 修改的代码
      force: true,
      message: "Version bumped by fastlane"
    )
	  push_to_git_remote # push 到 Git仓库
  end
```

更多用法可以查看文档：[Source-Control - fastlane docs](https://docs.fastlane.tools/actions/#source-control)
