# Mac 配置多个ssh-key

### 应用场景
我们经常将代码托管到github、gitlab这样的网站上。为了避免每次push代码时都要输入用户名和密码，通常会选择使用ssh协议，将公钥保存到托管网站上。在实际开发中，往往要将代码托管到多个不同的网站上。比如，公司的代码需要托管到coding上，自己的开源代码托管到GitHub上，私有代码托管到gitlab上等等，每个托管网站都对应一个git账户。默认情况下，一台电脑的Git只对应一个账户，只能往一个网站push代码，非常不便。这篇博客将介绍如何在一个Git终端中配置多个账户，同时管理多个托管网站的代码。

### 准备工作
首先，需要准备好对Git的全局用户进行配置。在初次安装Git时，往往会使用如下的命令配置全局用户名和邮箱：

```
git config --global user.name "xxx" // 配置全局用户名，如Github上注册的用户名
git config --global user.email "yyy@mail.com" // 配置全局邮箱，如Github上配置的邮箱
```

这个`--global`选项，是指这里配置的`user.name`和`user.email`是相对于全局进行配置的，即不同的Git仓库默认的用户名和邮箱都是这个值。由于需要管理多个账户，所以仅仅使用这个全局值是不够的，需要在每个仓库中单独配置。对此，有两种处理方法：

如果之前已经使用该命令进行配置，则先使用如下命令清除

```
git config --global --unset user.name
git config --global --unset user.email
```

如果不确定是否已经配置过，可以使用下面的命令查看

```
git config --global user.name
git config --global user.email
```

### 创建ssh-key

```
ssh-keygen -t rsa -f ~/.ssh/id_rsa.别名 -C “邮箱地址“
```



```
ssh-keygen -t rsa -f ~/.ssh/id_rsa.github -C “xxx@xxx.com“
```

```
ssh-keygen -t rsa -f ~/.ssh/id_rsa.gitee -C “xxx@xxx.com“

```

![image.png](https://upload-images.jianshu.io/upload_images/1401554-1882c8805aa15e8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](https://upload-images.jianshu.io/upload_images/1401554-a49bb032a96ecd07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](https://upload-images.jianshu.io/upload_images/1401554-29aa75adbc8ef22b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 复制公钥

```
$ cat id_rsa_hostname.pub

```

![image.png](https://upload-images.jianshu.io/upload_images/1401554-4f7bf802980bd1f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1401554-875f2f05f6c7ff76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


ssh-key文件已经生成到指定路径了，那么接下来我们来配置一下。


### 重复上述步骤，创建好多个秘钥对

**但是还没有完，ssh服务器默认是去找id_rsa，现在需要把这个key添加到ssh-agent中**

这样ssh服务器才能认识`my_id_rsa`
在终端执行:(这里为什么加上了一个-K参数呢？因为在Mac上，当系统重启后会“忘记”这个密钥，所以通过指定-K把SSH key导入到密钥链中。)

```
ssh-add -K ~/.ssh/my_id_rsa
```

查看添加结果：

```
ssh-add -l
```


### 创建config文件

```
cd .ssh
touch config
```

### 配置

>参数说明：<br>
Host :分别为公司自己的gitlab托管的服务地址（一般公司都有自己的git托管服务）<br>
Hostname :随便起<br>
User :随便<br>
PreferredAuthentications :写死publickey<br>
IdentityFile :公钥地址。<br>
注，如果我们用的id_rsa添加在了过个git托管服务器，比如github,我们只要再添加github的配置就行了<br>


```

# second user(xxx@xxx.com)
# 建一个github别名，新建的帐号使用这个别名做克隆和更新
  Host github
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa.github

# second user(xxx@xxx.com)
# 建一个gitee别名，新建的帐号使用这个别名做克隆和更新
  Host gitee
  HostName gitee.com
  User git
  IdentityFile ~/.ssh/id_rsa.gitee

```

Host是别名。如果只是为了区分github、gitee等，为了方便使用，建议和HostName一致，这样在clone git的时候不用考虑修改hostname。


### 4、通过别名来使用

```
ssh -T gitee

```

```
ssh -T github

```

>Hi xinwen-mao! You've successfully authenticated, but GitHub does not provide shell access.<br>
表示成功
![image.png](https://upload-images.jianshu.io/upload_images/1401554-fded8eb8d1eb6fc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### error

![image.png](https://upload-images.jianshu.io/upload_images/1401554-212445137b889fd8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这个问题是由于权限的问题，需要文件设置权限：

```
chmod 600 *
```

![image.png](https://upload-images.jianshu.io/upload_images/1401554-ae52144beb3df1db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




参考文章：

[管理本地多个SSH Key](http://www.jouypub.com/2018/0959250d1c900128efc07cf055dfeb62/)

[Mac下配置多个SSH-Key](https://www.cnblogs.com/520yang/articles/6875260.html)

[同一台Mac上配置多个ssh及git](https://www.jianshu.com/p/c15af45a2519)