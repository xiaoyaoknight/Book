# CocoaPods
[CocoaPods](https://guides.cocoapods.org/)


### 第一步:先把ruby源搞定 

TODO [Ruby安装方式]()<br>

Gem 查看可用的Source<br>
目前，淘宝的source已经不维护了，这已是明日黄花。<br>
默认ruby 源 https://rubygems.org/ 墙内访问不到<br>
`gem source -a https://gems.ruby-china.org`
`gem source -a https://gems.ruby-china.com`

######查看当前source：
`gem source -l`

######移除当前source：
`gem source -r XXXXX`

######添加可用的source：
`gem sources --add https://gems.ruby-china.com/`


######更新cache：
`gem source -u`


###第二步:升级Gem
[Gem]()

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


---- 

### 第三步: cocoapods卸载
###### 1.在装之前最好先卸载点老版本

```
$ sudo gem uninstall cocoapods
```

###### 2.查看本地安装过的cocopods相关东西

```
$ gem list --local | grep cocoapods
```

> 显示如下:<br>
cocoapods (1.0.1)<br>
cocoapods-core (1.0.1)<br>
cocoapods-deintegrate (1.0.1)<br>
cocoapods-downloader (1.1.1)<br>
cocoapods-plugins (1.0.0)<br>
cocoapods-search (1.0.0)<br>
cocoapods-stats (1.0.0)<br>
cocoapods-trunk (1.0.0)<br>
cocoapods-try (1.1.0)<br>


![image.png](http://upload-images.jianshu.io/upload_images/1401554-3e0fda007d3f21a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500/h/1000)

按提示卸载
```
gem uninstall -i /Users/xxx/.rvm/gems/ruby-2.5.0@global cocoapods
```

##### 查看当前cocoapods使用版本
```
pod --version 
```

### 第四步：安装CocoaPods

```
sudo gem install cocoapods　// Mac OS X 10.11前  输入这一条
sudo gem install -n /usr/local/bin cocoapods  //Mac OS X 10.11后   输入这一条
```

##### 报错
> Ignoring executable-hooks-1.6.0 because its extensions are not built.  Try: gem pristine executable-hooks --version 1.6.0
Ignoring gem-wrappers-1.4.0 because its extensions are not built.  Try: gem pristine gem-wrappers --version 1.4.0

运行`gem pristine --all`即可，如果一遍不行，再运行一遍
报权限问题，加sudo


```
pod setup 
```
> 这条命令是将Github上的开源库都托管都安装Podspec索引安装到到本地,<br>
这一步,<br>
很慢.....<br>
很慢..........<br>
很慢...............<br>

>这个时候要去把整个specs仓库clone一下，下载到 ~/.cocoapods里；
cd  到该目录里，用du -sh *命令来查看文件大小，每隔一会看看。

依然失败的话
```
git clone https://git.coding.net/CocoaPods/Specs.git ~/.cocoapods/repos/master 
```
该过程作用与 pod setup作用相同

##### 再次查看版本
`pod --version`


### 第五步：Cocoapods 安装指定版本

由于一些pod版本造成的异常问题，建议安装稳定版本。

```
$ sudo gem install cocoapods --version 1.7.4
```

#####  执行pod repo update 总是失败

>rm -rf ~/.cocoapods/repos/master<br>
拷贝最新的master 到~/.cocoapods/repos/master/下<br>
再执行 pod repo update<br>
这个速度快<br>


##### cocoapods报错 [!] Couldn't determine repo type for URL: `https://cdn.cocoapods.org/`: execution expired


>cocoapods 1.7.2版本后CDN为默认值
使用1.8，CocoaPods不再需要克隆现在巨大的主规格repo才能运行，用户几乎可以立即将他们的项目与CocoaPods集成。

>编辑Podfile以将CDN设置为主要来源：
source 'https://cdn.cocoapods.org/'

##### cocoaPods 版本1.8.4，pod install失败问题

```
Adding spec repo `trunk` with CDN `https://cdn.cocoapods.org/`

[!] CDN: trunk Repo update failed - 58 error(s):

网上看了半天，说在podfile中添加source ‘https://github.com/CocoaPods/Specs.git’
```

问题仍然没解决

```
（1）podfile添加source 'https://github.com/CocoaPods/Specs.git'

（2）pod repo list 查看一下源列表

（3）pod repo remove trunk 移除trunk源
```

[cocoapods1.8CDN介绍](https://www.91kuzhan.com/article-3310-1.html)

### 解决ping github.com超时问题

```
# GitHub地址
125.120.42.110 github.com git  
13.229.188.59 github.global.ssl.fastly.net  
```
**125.120.42.110这个IP地址需要修改成你的IP地址**

### pod 常用命令

```
$ pod setup # CocoaPods 将信息下载到~/.cocoapods/repos 目录下。如果安装 CocoaPods 时不执行此命令，在初次执行 pod intall 命令时，系统也会自动执行该指令
$ pod --version # 检查 CocoaPods 是否安装成功及其版本号
$ pod install # 安装 CocoaPods 的配置文件 Podfile
$ pod repo list # 查看一下源列表
$ pod repo remove trunk # 移除trunk源
```

### CocoaPods 删除本地缓存

下载不易，清理需谨慎。

```
// 移除本地master
sudo rm -fr ~/.cocoapods/repos/master
// 移除本地缓存
sudo rm -fr ~/Library/Caches/CocoaPods/
```

---

参考文章：

[更新ruby源报错bad response Not Found 404](https://www.jianshu.com/p/3bbbd139d2ce)

[CocoaPods最新安装及跳过pod setup快速安装教程](https://www.cnblogs.com/zhuyanboyue/p/6118950.html)

[CocoaPods多版本](http://blog.csdn.net/focusjava/article/details/51325802)

[CocoaPods操作常见问题](https://www.jianshu.com/p/f046cb769d5a)

[iOS开发 - CocoaPods的常见使用方式](https://www.cnblogs.com/hs-funky/p/6759977.html)