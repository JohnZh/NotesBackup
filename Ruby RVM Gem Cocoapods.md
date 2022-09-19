# Ruby RVM Gem

Ruby 服务器端脚本语言

RVM：Ruby version manager 版本管理和安装工具

Gem：ruby 软件包和库管理软件（RubyGems）



## ruby 基本命令

- ruby -v 查看版本
- 



## RVM 基本命令

- rvm list 查看已经安装的ruby 版本
- rvm list known 查看可用的 ruby 版本
- rvm install <version> 安装可用版本
- rvm use <version> 使用已安装的可用版本
  - rvm use 2.4.6 --default 使用并设置为默认
- rvm remove <version>



## Gem 基本命令

### Gem 源的命令

- 查看所有源：gem sources -l 
- 添加源：gem sources -a  $url 
- 更新源：gem sources -u 
- 删除源：gem sources -r $url

> 可用 gem 源：
>
> https://rubygems.org/ 
>
> https://gems.ruby-china.com



- gem list 查看所有安装的包

  - gem list j 列出所有 j 开头的包




# Cocoapods

OSX 和 IOS 下第三类库管理工具

- 安装：sudo gem install -n /usr/local/bin cocoapods



