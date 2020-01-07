# Gem 

### 升级Gem
Gem是来管理Ruby标准包<br>
```
sudo gem update --system//升级gem
```

#####更新gem报错
> ERROR:  While executing gem … (Errno::EPERM)
    Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/bin/gem

#####更新ruby源报错 
> bad response Not Found 404

原因是 ruby-china 更换了域名

命令替换为 `gem sources --add https://gems.ruby-china.com`
![image.png](https://upload-images.jianshu.io/upload_images/1401554-4f48eef65218a6ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> ERROR:  While executing gem ... (OptionParser::InvalidOption)
    invalid option: --system

#####更换更新方法

`gem update --system`

#####查看版本
```
gem -v 　
```
**不升级在第三步可能,会报很多错误的,因为Gem版本太低,无法安装**


### Gem 常用命令

```
$ gem -v # 查看 gem 版本
$ gem source # 查看 gem 配置源
$ gem source -l # 查看 gem 配置源目录
$ gem sources -a url # 添加 gem 配置源（url 需换成网址）
$ gem sources --add url # 添加 gem 配置源（url 需换成网址）
$ gem sources -r url # 删除 gem 配置源（url 需换成网址）
$ gem sources --remove url # 删除 gem 配置源（url 需换成网址）
$ gem update # 更新 所有包
$ gem update --system # 更新 Ruby Gems 软件

$ gem install rake # 安装 rake，从本地或远程服务器
$ gem install rake --remote # 安装 rake，从远程服务器
$ gem install watir -v 1.6.2 # 安装 指定版本的 watir
$ gem install watir --version 1.6.2 # 安装 指定版本的 watir
$ gem uninstall rake # 卸载 rake 包
$ gem list d # 列出 本地以 d 打头的包
$ gem query -n ''[0-9]'' --local # 查找 本地含有数字的包
$ gem search log --both # 查找 从本地和远程服务器上查找含有 log 字符串的包
$ gem search log --remoter # 查找 只从远程服务器上查找含有 log 字符串的包
$ gem search -r log # 查找 只从远程服务器上查找含有log字符串的包

$ gem help # 提醒式的帮助
$ gem help install # 列出 install 命令 帮助
$ gem help examples # 列出 gem 命令使用一些例子
$ gem build rake.gemspec # 把 rake.gemspec 编译成 rake.gem
$ gem check -v pkg/rake-0.4.0.gem # 检测 rake 是否有效
$ gem cleanup # 清除 所有包旧版本，保留最新版本
$ gem contents rake # 显示 rake 包中所包含的文件
$ gem dependency rails -v 0.10.1 # 列出 与 rails 相互依赖的包
$ gem environment # 查看 gem 的环境

$ sudo gem -v # 查看 gem 版本（以管理员权限）
$ sudo gem install cocoa pods # 安装 CocoaPods（以管理员权限）
$ sudo gem install cocoapods # 安装 CocoaPods（以管理员权限）
$ sudo gem install cocoapods --pre # 安装 CocoaPods 至预览版（以管理员权限）
$ sudo gem install cocoapods -v 0.39.0 # 安装 CocoaPods 指定版本（以管理员权限）
$ sudo gem update cocoapods # 更新 CocoaPods 至最新版（以管理员权限）
$ sudo gem update cocoapods --pre # 更新 CocoaPods 至预览版（以管理员权限）
$ sudo gem uninstall cocoapods -v 0.39.0 # 移除 CocoaPods 指定版本（以管理员权限）
```