# Homebrew相关问题

##### 简介：
>Homebrew也称brew，macOS下基于命令行的最强大软件包管理工具，使用Ruby语言开发。类似于CentOS的yum或者Ubuntu的apt-get，brew能方便的管理软件的安装、更新、卸载，省去了手动编译或拖动安装的不便，更使得软件的管理更加集中化，解决了依赖包管理的烦恼。

#####资料
- [官网](https://brew.sh)
- [官方文档](https://docs.brew.sh)
- [官方软件包仓库](https://formulae.brew.sh)
- [brew的详细指南介绍](https://shockerli.net/post/macos-homebrew-manual/)
- homebrew的GUI工具Cakebrew


##### 查看当前shell
`echo $SHELL`

##### zsh切换bash 
`chsh -s /bin/bash`

##### bash切换zsh
`chsh -s /bin/zsh`

##### 安装
Homebrew 依赖于Xcode Command Line Tools，所以需先执行：
`xcode-select --install`

> 错误记录：xcode-select: error: command line tools are already installed, use "Software Update" to install

> 解决办法：(如果提示权限不够那么加上sudo)
`rm -rf /Library/Developer/CommandLineTools`
`xcode-select --install`


在终端中执行：
`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

执行中.....巨大
![image.png](https://upload-images.jianshu.io/upload_images/1401554-f6a097bcb5944422.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**注意:**<br>

- 提示1：是否翻墙成功
- 提示2：ping github.com 是否有timeout




**各种JB错误解决**
> Error: Failed to link all completions, docs and manpages:
  Permission denied @ rb_file_s_symlink - (../../../Homebrew/completions/zsh/_brew, /usr/local/share/zsh/site-functions/_brew)
Failed during: /usr/local/bin/brew update --force



检查是否已安装成功：

`brew -v`



##### 卸载(换成uninstall)

`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"`



有个坑：
在配置ls配色的时候,会报brew警告：
在`.bash_profile` 或 `.zshrc`中配置了如下
```
 #ls 配色
 if brew list | grep coreutils > /dev/null ; then
	PATH="$(brew --prefix coreutils)/libexec/gnubin:$PATH"
	alias ls='ls -F --show-control-chars --color=auto'
	eval `gdircolors -b $HOME/.dir_colors`
 fi
```
![image.png](http://upload-images.jianshu.io/upload_images/1401554-2cec16efd5476410.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 查看自身版本
`brew -v` 

##### 第一步：更新self

`brew update`

##### 第二步：找出已过期的软件包,需要更新的软件

` brew outdated`

##### 第三步：升级所有过期软件包

`brew upgrade`

##### 升级指定的过期软件包

`brew upgrade XXX`

##### 升级过程中要暂停/恢复软件包的安装过程
- 暂停安装过程
`brew pin $FORMULA`

- 恢复安装过程
`brew unpin $FORMULA`

##### 卸载掉旧的软件包

>默认情况下，Homebrew不会自动卸载掉旧的软件包，故随着时间的积累，电脑中会积累起很多老版本的软件包，甚至是同一个软件包的多个老版本，那么要移除这些软件包的老版本，只需这么做：

##### 第一种：清除指定软件包的所有老版本
`brew cleanup $FORMULA`

##### 第二种：清除所有软件包的所有老版本
`brew cleanup`

##### 第三种：查看哪些软件包要被清除
`brew cleanup -n`

>对于Homebrew来说，如果没有卸载掉软件包的所有版本，那么Homebrew会继续尝试安装这个软件包的最新版本。要想彻底卸载某个软件包，需要执行命令：
`brew uninstall formula_name --force`


##### 更新pip
`pip install -U pip`

To upgrade all local packages; you could 

```
$ pip install pip-review

$ pip-review --local --interactive
```



参考地址：
[stackoverflow](https://stackoverflow.com/questions/34617452/how-to-update-xcode-from-command-line)



