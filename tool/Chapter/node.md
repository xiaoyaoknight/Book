# Node

目前主流的node版本管理工具有两种，nvm和n。
感兴趣的可以查看一下下面这篇文章，学习一下。

[管理 node 版本，选择 nvm 还是 n？](https://www.cnblogs.com/shengulong/p/9343172.html)

此次主要记录nvm的安装方式，以及卸载降级等，注意自己选择的方式。


## 一、nvm方式
### 1、安装 nvm 
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.30.2/install.sh | bash
```

### 2、配置 nvm 环境变量
nvm被安装在了`~/.nvm`
我使用的是`zsh`，就需要在`~/.zshrc`这个配置文件中配置
否则就自行配置下`~/.bash_profile`或者`~/.profile`

打开`~/.zshrc`，在最后一行加上：

```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```

### 3、让 nvm 生效
`source ~/.zshrc`重新启动一下配置。
输入`nvm`可以看到如下信息：

```
➜  ~  nvm

Node Version Manager

Note: <version> refers to any version-like string nvm understands. This includes:
  - full or partial version numbers, starting with an optional "v" (0.10, v0.1.2, v1)
  - default (built-in) aliases: node, stable, unstable, iojs, system
  - custom aliases you define with `nvm alias foo`

Usage:
  nvm help                                  Show this message
  nvm --version                             Print out the latest released version of nvm
  nvm install [-s] <version>                Download and install a <version>, [-s] from source. Uses .nvmrc if available
    --reinstall-packages-from=<version>     When installing, reinstall packages installed in <node|iojs|node version number>
  nvm uninstall <version>                   Uninstall a version
  nvm use [--silent] <version>              Modify PATH to use <version>. Uses .nvmrc if available
  nvm exec [--silent] <version> [<command>] Run <command> on <version>. Uses .nvmrc if available
  nvm run [--silent] <version> [<args>]     Run `node` on <version> with <args> as arguments. Uses .nvmrc if available
  nvm current                               Display currently activated version
  nvm ls                                    List installed versions
  nvm ls <version>                          List versions matching a given description
  nvm ls-remote                             List remote versions available for install
  nvm version <version>                     Resolve the given description to a single local version
  nvm version-remote <version>              Resolve the given description to a single remote version
  nvm deactivate                            Undo effects of `nvm` on current shell
  nvm alias [<pattern>]                     Show all aliases beginning with <pattern>
  nvm alias <name> <version>                Set an alias named <name> pointing to <version>
  nvm unalias <name>                        Deletes the alias named <name>
  nvm reinstall-packages <version>          Reinstall global `npm` packages contained in <version> to current version
  nvm unload                                Unload `nvm` from shell
  nvm which [<version>]                     Display path to installed node version. Uses .nvmrc if available

Example:
  nvm install v0.10.32                  Install a specific version number
  nvm use 0.10                          Use the latest available 0.10.x release
  nvm run 0.10.32 app.js                Run app.js using node v0.10.32
  nvm exec 0.10.32 node app.js          Run `node app.js` with the PATH pointing to node v0.10.32
  nvm alias default 0.10.32             Set default node version on a shell

Note:
  to remove, delete, or uninstall nvm - just remove the `$NVM_DIR` folder (usually `~/.nvm`)
```

### 使用切换node版本 [nvm文档](https://github.com/creationix/nvm)

#### 1、查看node有哪些版本可以安装
`nvm ls-remote`：

```
  v10.14.2
       v10.15.0
       v10.15.1
        v11.0.0
        v11.1.0
        v11.2.0
        v11.3.0
        v11.4.0
        v11.5.0
        v11.6.0
        v11.7.0
        v11.8.0
        v11.9.0
       v11.10.0
            ...
```

#### 2、安装多个node版本

```
$ nvm install v11.0.0
######################################################################## 100.0%

Now using node v11.0.0
```

```
$ nvm install v11.10.0
######################################################################## 100.0%

Now using node v11.10.0
```
#### 3、使用nvm轻松切换node版本

在介绍使用方法前，简单说明一下nvm的工作原理：
按照我上述安装方法的话，nvm会将各个版本的node安装在`~/.nvm/versions/node`目录下，我们可以打开这个目录看看有些什么东西：
```
➜  ~  ls -a ~/.nvm/versions/node
```
事实上`vXXX`和`vXX`这两个目录分别存放node的binary档，nvm会在`$PATH`前面安插指定版本的目录，透过这种方式在使用node命令时就会用指定版本的node来运行了。
可以确认实际的`$PATH`看看：

```
➜  ~  echo $PATH
/Users/***/.nvm/versions/node/xxx/bin:...
```

由于刚刚我们通过nvm安装node，会自动把最后安装的版本设为当前使用的版本，因此上述路径结尾会是`.../v11.10.0/bin`（还可通过`nvm ls`命令查看当前已安装的所有node版本）。

接下来我们可以使用`nvm use <version>`切换版本：

```
 nvm use vxxx
```

#### 3、使用nvm设置默认node版本
```
➜  ~  nvm alias default v11.10.0
default -> v11.10.0
```

此时再打开一个bash输入`nvm current`就会显示为`v11.10.0`了



----
----
------ 


## Mac 彻底卸载node和npm

### Homebrew安装的

```
brew uninstall node
```

### 官网下载pkg安装包

```
sudo rm -rf /usr/local/{bin/{node,npm},lib/node_modules/npm,lib/node,share/man/*/node.*}

```


### 其他骚操作一

1\. 删除`/usr/local/lib`中的所有node和node_modules

2\. 删除`/usr/local/lib`中的所有node和node_modules的文件夹

3\. 如果是从brew安装的, 运行brew uninstall node

4\. 检查`~/`中所有的`local`, `lib`或者`include`文件夹, 删除里面所有`node`和`node_modules`

5\. 在`/usr/local/bin`中, 删除所有node的可执行文件

6\. 最后运行以下代码:

```
sudo rm /usr/local/bin/npm
sudo rm /usr/local/share/man/man1/node.1
sudo rm /usr/local/lib/dtrace/node.d
sudo rm -rf ~/.npm
sudo rm -rf ~/.node-gyp
sudo rm /opt/local/bin/node
sudo rm /opt/local/include/node
sudo rm -rf /opt/local/lib/node_modules
```

### 其他骚操作二：

先删除方法一文件，在运行以下指令：

```
sudo rm /usr/local/${i}

sudo rm -rf /usr/local/lib/node \
/usr/local/lib/node_modules \
/var/db/receipts/org.nodejs.*

sudo rm -rf /usr/local/include/node /Users/$USER/.npm
```

###  其他骚操作三：

由于之前安装过，在 package.json 中的记录仍然存在

```
npm uninstall npm -g
```

```
npm uninstall lodash
```

--save 参数使用
卸载模块的同时删除在 package.json 文件中的记录

```
 npm uninstall lodash --save
```

卸载指定版本的模块

```
npm uninstall lodash@3.*  // 卸载 lodash 模块 3.* 版本
```


### 检查是否卸载

```
node
npm
```

-----------
-----------
-----------


## [npm 中文文档](https://www.npmjs.cn/)


### 修复权限到 npm's 默认目录
`npm config get prefix`


###查看当前版本
```
npm -v
```

### npm切换源
1.config

```
npm config set registry https://registry.npm.taobao.org
```

2.命令行指定

```
  npm i node --registry https://registry.npm.taobao.org info underscore 
```

3.编辑 ~/.npmrc 加入下面内容

```
  registry = https://registry.npm.taobao.org
```

### npm安装需要的版本（版本号 可以根据已发布的版本随意升级或降级）

```
npm -g install npm@6.7.0
```

### npm更新

```
npm update -g npm
```

```
npm outdated -g --depth=0：找出需要更新的包。

```



