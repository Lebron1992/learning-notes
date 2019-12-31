
开源的分布式版本控制工具，在所有的分布式版本控制工具中，git是最快、最简单、最流行的。

## GIT和SVN对比

- 在很多情况下，git的速度远远比SVN快
- SVN是集中式管理，git是分布式管理
- SVN使用分之比较笨拙，git可以轻松拥有无限个分支
- SVN必须联网才能正常工作，git支持本地版本控制工作
- 旧版本的SVN会在每一个目录放置一个.svn，git只会在根目录拥有一个.git

## 本地

- `git config user.name Lebron`: 配置用户名
- `git config user.email youremail@exmaple.com`: 配置邮箱
- `git config --global user.name Lebron`: global，将此设置应用到整个系统中
- `git config --global user.email youremail@exmaple.com`: global，将此设置应用到整个系统中
- `git --help`: git指令帮助
- `git config -l`: 查看配置信息
- `git config -e`: 编辑配置信息 （用vim编辑，输入i进行编辑，:wq是退出编辑）
- `git config alias.别名 原指令名称`: 设置指令别名
- `git config -alias.别名 原指令名称 参数`: 设置带参数指令的别名
- `git status`: 查看文件状态
- `git log`: 查看文件的修改日志；1）用一行的方式查看简单的日志信息：`git log --pretty=oneline`；2）查看最近n次修改：`git log -n`
- `git diff`: 查看文件最新的改动地方
- `git init`: 初始化一个空的本地仓库
- `git add`: 将工作区的文件保存到缓存区；保存当前路径的所有文件到缓存区：`git add .` (注意后面的点)
- `git commit -m "注释" 文件名称`：将缓存区的文件提交到当前分支      
- `git reflog`: 查看分支引用记录，能查看所有版本号
- `git reset`: 版本回退（建议加上--hard参数，git支持无限次后悔）；2）`git reset --hard HEAD^`：会退到上一个版本；3）`git reset --hard HEAD^^`：回退到上上一个版本；4）`git reset --hard HEAD~n`：回退到上n个版本；5）`git reset --hard 版本号`：回退到任意一个版本，版本号用7位即可
- `git rm`: 删除文件，删除完之后要进行commit操作，才能同步到版本库
- `git clone`: 下载远程仓库到本地
- `git pull`: 下载远程仓库的最新信息到本地仓库
- `git push`: 将本地仓库信息推送到远程仓库
- `git rebase -i`: 合并
- `git checkout test`: 切换分支到test
- `git push origin lz-flurry-99:lz-flurry-99`: 推送本地分支到远程分支
- `git branch -d xxxxx`: 删除本地分支
- `git cherry-pick 9f63dd6`: 把commit放到另一个分支上
- `git push origin --delete <branchName>`: 删除远程分支，或者用 `git push origin :<branchName>`
- `git branch -m 原分支名 新分支名`: 分支重命名

## `git add -A`、`git add -u` 和 `git add .` 的区别

- `git add .`: 他会监控工作区的状态树，使用它会把工作时的所有变化提交到暂存区，包括文件内容修改(modified)以及新文件(new)，但不包括被删除的文件。
- `git add -u`: 他仅监控已经被add的文件（即tracked file），他会将被修改的文件提交到暂存区，不会提交新文件（untracked file）
- `git add -A`: 上面两个功能的合集（`git add --all`的缩写)

**总结:**

- `git add -A`: 提交所有变化
- `git add -u`: 提交被修改(modified)和被删除(deleted)文件，不包括新文件(new)
- `git add .`: 提交新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件

## 上传本地代码到github

- `git remote add origin 地址`
- `git pull origin master`，如果提示`refusing to merge unrelated histories`，使用 `git pull origin master --allow-unrelated-histories`
- `git push -u origin master`

## Git的文件状态

```
M = Locally modified  本地修改
  
U = Updated in repository  在仓库中被更新
  
A = Locally added  在本地添加
  
D = Locally deleted  在本地删除
  
I = Ignored  被忽略
  
R = Replaced in the repository  在仓库中被替换
  
– = The contents of the folder have mixed status; display the contents to see individual status  // 文件夹中的文件有多种状态（被修改、更新、添加、删除等等）
  
? = Not under source control // 没有加入版本控制
```
