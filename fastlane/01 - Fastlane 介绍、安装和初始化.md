# 01 - Fastlane 介绍、安装和初始化

## 介绍

Fastlane 是一个命令行工具的集合，用来改进和自动化与 App Store 生态的交互。它包含了大量的原子操作，每个操作用于处理一个特定的任务。他不进让很多事情变得更简单，更是简化了你和你的团队的整个 iOS 开发周期。例如，如果我们手动去准备 screenshots 和相关的元数据，在 Fastlane 中我们可以定义一个 lane 去处理这件事情，然后只需要执行一个命令就可以处理手动处理的工作。

Fastlane 是 2014 年开源的，我们可以在 GitHub 中找到它，[GitHub - fastlane/fastlane](https://github.com/fastlane/fastlane)，使用 Ruby 编写。我们可能会问：为什么不用 Swift 编写？其实 Fastlane 团队有使用 Swift 编写了 beta 版本，但团队还是建议在生产环境中使用 Ruby 来编写。目前 Swift 版本的 Fastlane 还没有开发完成。我们需要使用 Ruby 来编写 lane，但不太懂 ruby 也没太大关系。

## 安装

### 安装 command line developer tools

执行 `xcode-select --install` 命令：
	- 如果提示 `xcode-select: error: command line tools are already installed, use “Software Update” to install updates`，说明已经安装好；
	- 否则会弹出一个窗口，点击 install 来安装。

安装好之后，执行 `xcode-select --print-path` 查看 command line 的路径，如果存在，说明安装好了。

### 安装 homebrew

如果已经安装好了 homebrew，则可以跳过这一步。还没安装的，则运行这个命令来安装：`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

### 安装 ruby

目前的 mac 电脑都应该自带了 ruby，我们可以执行 `ruby -v` 检查一下：
	- 如果版本号大于 `2.3.7`，那就可以了；
	- 否则执行 `brew install ruby` 来安装 ruby，ruby 安装好之后，要把刚刚安装好的 ruby 的路径指定到 shell 的配置文件，shell 的配置文件的路径可能是 `.profile` 和 `.bash_profile`，或者是其他；在配置文件中加上 `export PATH = "usr/local/opt/rby/bin:$PATH”`并且保存。 

最后执行 `gem install bundler` 完成 ruby 的安装。

## 安装 git

一般 Mac 电脑自带 git，如果没有的，执行 `brew install git`。

## 安装 fastlane

执行以下其中一条命令：

```ruby
# 使用 RubyGems
sudo gem install fastlane

# 使用 Homebrew
brew cask install fastlane
```

## 初始化

在项目的根目录中执行 `fastlane init`：
	- 首先会询问你用 fastlane 来干嘛，我们选择第四个 `Manual setup`，输入 4，然后按 enter 键
	- 剩下的其他提示，都按 enter 键

打开根目录，你会看到一些自动生产的文件，后续的自定义 lanes 会保存在 `fastlane/Fastfile` 文件中。