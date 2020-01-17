# 解决pod lib lint/repo push不支持i386编译&只能真机运行的库


#####**问题：pod lib lint/repo push不支持i386编译&只能真机运行的库**
1. 终端 `gem which cocoapods`
输出：/Users/xxxx/.rvm/gems/ruby-2.5.0/gems/cocoapods-1.5.0/lib/cocoapods.rb

2. 终端 `/Users/xxxx/.rvm/gems/ruby-2.5.0/gems/cocoapods-1.5.0/lib`

3. 在当前lib目录下有个cocoapods文件夹，进入，`validator.rb`文件就在这个文件夹里

![image.png](https://upload-images.jianshu.io/upload_images/1401554-7922c2c743a13050.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800/h/600)


找到下面的代码
![image.png](https://upload-images.jianshu.io/upload_images/1401554-ccc1b8d476857840.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

修改成
`command += %w(--help)`
![image.png](https://upload-images.jianshu.io/upload_images/1401554-0342f382290320ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.执行验证库的命令
**注意点：source后面引用到的库地址，逗号分隔，都写上。**
```
pod lib lint --verbose  --allow-warnings --sources="https://github.com/xxx/Specs.git,https://git.xxx.com/spec.git,https://github.com/CocoaPods/Specs.git"
```

5.创建本地的配置文件仓库 
使用下面的命令就可以在本地生成配置文件的仓库 
wshSpecs 是你远端创建的配置文件仓库的名字,后面是配置文件仓库远端的地址 
注意此时不需要cd进入任何目录,从默认位置输入这个命令即可,即在点击终端快捷方式打开的状态下直接输入下面的命令即可 `pod repo add wshSpecs http://njGitrepo/wushuanghong/wshSpecs.git `

可以使用下面的命令查看是否成功 open ~/.cocoapods 

![image.png](https://upload-images.jianshu.io/upload_images/1401554-e174873672c8401a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/760)

6.将配置文件push到远端专门存储配置文件的仓库中 
此时需要cd进入本地的code repository,否则会找不到podspec文件 
使用下面的命令即可,有问题可以参考报错信息去修改 
wshSpecs 是你创建的spec repository的名字,后面是你本地创建的podspec文件,成功之后你可以在远端仓库中看到这个文件,并且远端配置文件仓库中有且只有这一个文件 
`pod repo push wshSpecs caculatormaker.podspec`
![image.png](https://upload-images.jianshu.io/upload_images/1401554-c075761647f1c557.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/760)


参考文章：
[解决pod lib lint/repo push不支持i386编译&只能真机运行的库](https://www.jianshu.com/p/88180b4d2ab7)
[github issue](https://link.jianshu.com/?t=https://github.com/CocoaPods/CocoaPods/issues/5854)
[待解决 pod lib lint pod不支持i386编译环境，如何避开](https://www.jianshu.com/p/9d32368ae3b2)
[ios创建自己的私有库](https://www.aliyun.com/jiaocheng/359612.html)