# Mac Ruby

前言：<br>
>正确安装Mac系统下的ruby<br>
Ruby安装方式有两种,一个是 rvm多环境安装, 一种是homebrew安装

[RubyGem](https://rubygems.org)
### 一 rvm多环境安装（高手推荐，我没整太明白）
>RVM是一个命令行工具，它允许您轻松地安装，管理和使用从解释器到多组宝石的多个ruby环境。

>虽然 macOS 自带了一个 ruby 环境，但是是系统自己使用的，所以权限很小，只有 system。而/Library 目录是 root 权限,所以很多会提示无权限。

>使用自带ruby更新,管理不方便

>一系列无原因的报错

>删除系统ruby方法[删除容易出现问题，尽量不要删除，不要删除，不要删除]



##### 检查您当前正在使用系统Ruby，请打开终端并输入以下内容：

`which ruby`

##### 如果您使用的是Ruby系统，OS X将回应：

`/usr/bin/ruby`

##### 您可以检查使用哪个版本的Ruby OS X：

`ruby -v`

##### 安装RVM

###### 1.安装mpapis公钥。但是，正如安装页面所记录的，您可能需要gpg。Mac OS X不附带gpg，因此在安装公钥之前，您需要安装gpg。我用Homebrew安装了gpg ：

```
brew install gnupg 
```

###### 2.安装完gpg之后，你可以安装mpapis公钥：

```
gpg --keyserver hkp://pgp.mit.edu --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

```

###### 3.安装最新版本的Ruby的RVM

```
\curl -sSL https://get.rvm.io | bash -s stable --ruby

```

###### 安装成功，推出重新打开终端

###### 列出可供RVM使用的Ruby版本
```
rvm list
```


###### 安装失败
> gpg: 从公钥服务器接收失败：No route to host
> Mac没有自带 gpg 所以每次都失败, 之后曾经安装了gpg, 然而还是发现显示没有rvm资源,尝试更换服务器
```
gpg –keyserver hkp://pgp.mit.edu –recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
```
仍然失败!!!

离线安装RVM方式<br>
[官方离线安装](https://rvm.io/rvm/offline)<br>
下载 官方离线安装包rvm-stable.tar.gz 后续步骤如下:

```
// 离线包
curl -sSL https://github.com/rvm/rvm/tarball/stable -o rvm-stable.tar.gz
// 创建文件夹
mkdir rvm && cd rvm
// 解包
tar --strip-components=1 -xzf ../rvm-stable.tar.gz
// 安装 
./install --auto-dotfiles
// 加载
source ~/.rvm/scripts/rvm
// if --path was specified when instaling rvm, use the specified path rather than '~/.rvm'

```


#####选择使用的版本

`rvm use 2.4.1`

##### 检查现在使用RVM管理的Ruby版本
`ruby -v`

##### 将系统的ruby切换为下载的版本
`rvm use 2.6.0  --default`


##### 卸载rvm

```
$ rvm  implode
```

> Could not remove '/Users/wangzelong/.rvm/', please try removing it manually.
Failed to completely remove /Users/wangzelong/.rvm -- You will have to do so manually.




### 二 Homebrew 安装 Ruby 
**自己试了很久，还是推荐brew安装**
安装Ruby

```
// 1.先更新homebrew
brew update 
// 2. 使用homebrew的安装指令
brew install ruby
brew reinstall ruby 再次安装
```

到这可以查看 通过查看ruby是否安装成功

```
// 检查一下是否安装正确
$ rvm -v
rvm 1.22.17 (stable) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]

// 列出已知的ruby版本
$ rvm list known
// 查询已经安装的ruby
$ rvm list

```

###### 错误 ruby版本没有更新

```
$ which -a ruby
```

>/Users/matthew/.rvm/rubies/ruby-1.9.3-p0/bin/ruby<br>
/Users/matthew/.rvm/bin/ruby<br>
/usr/bin/ruby<br>

删除rvm相关路径.profile .bashrc .bash_login


如果是用iTerm 在 .bash_profile 中追加
```
export PATH=/usr/local/opt/ruby/bin:$PATH
```

如果是用iTerm2 在 .zshrc 中追加
```
export PATH=/usr/local/opt/ruby/bin:$PATH
```

[参考stackouverflow]( https://stackoverflow.com/questions/8730676/how-can-i-switch-to-ruby-1-9-3-installed-using-homebrew)


---
###ruby rvm 常用指令

```
$ ruby -v # 查看ruby 版本
$ rvm list known # 列出已知的 ruby 版本
$ rvm install 2.3.0 # 选择指定 ruby 版本进行更新
$ rvm get stable # 更新 rvm
$ rvm use 2.2.2 # 切换到指定 ruby 版本
$ rvm use 2.2.2 --default # 设置指定 ruby 版本为默认版本
$ rvm list # 查询已安装的 ruby 版本
$ rvm remove 1.9.2 # 卸载移除 指定 ruby 版本

$ curl -L https://get.rvm.io | bash -s stable # 安装 rvm 环境
$ curl -sSL https://get.rvm.io | bash -s stable --ruby # 默认安装 rvm 最新版本
$ curl -sSL https://get.rvm.io | bash -s stable --ruby=2.3.0 # 安装 rvm 指定版本
$ source ~/.rvm/scripts/rvm # 载入 rvm
```


参考文章：

[mac cocoa pods 安装过程中出现问题](https://blog.csdn.net/tongwei117/article/details/79789111)