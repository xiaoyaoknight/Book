# Cocoapods创建私有库

>引言：<br>
区区一个私有库问题，让老夫耽误了半天时间，MMP！一直以来都是脚本帮忙做后续发布操作，老夫习惯性的创建仓库，用仓库，脱离公司脚本，却没法搜到自己的库。未能深入了解原理，可悲啊。

[官方地址](http://guides.cocoapods.org/making/making-a-cocoapod.html)

发布到cocoapods上的公有仓库，该文章说的已经十分清楚了。

[CocoaPods公有仓库的创建](http://qiubaiying.top/2017/03/08/CocoaPods%E5%85%AC%E6%9C%89%E4%BB%93%E5%BA%93%E7%9A%84%E5%88%9B%E5%BB%BA/)

但是一般公司不会把自己的代码公开，经常选择一些私有地址，gitlab、码云等等。
所以这就用到了私有库的创建。
[CocoaPods私有仓库的创建](https://www.jianshu.com/p/0c640821b36f)

总结补充两点：
`~/.cocoapods/repos` 此目录下存放着自己cocoapods库的索引，用于找到你库的仓库地址。

以下图为例：
![image.png](https://upload-images.jianshu.io/upload_images/1401554-dcb22e95b6101e1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
`ZL_Common`是项目代码仓库,用于其他项目pod导入
`ZL_CommonSpec`是存放ZL_Common仓库地址的索引，用于放在`~/.cocoapods/repos`,让你能搜索到自己的库

```
// ZL_CommonSpec存放项目的地址
pod repo add ZL_CommonSpec https://gitlab.com/xxxx/ZL_CommonSpec.git 
```

```
// 把仓库`ZL_Common`关联索引仓库`ZL_CommonSpec`上
pod repo push ZL_CommonSpec ZL_Common.podspec
```
![image.png](https://upload-images.jianshu.io/upload_images/1401554-0f00ca596b60255a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)



1指定目录下执行命令：
 `pod lib create XXXX` 

![image.png](http://upload-images.jianshu.io/upload_images/1401554-a07504f0f1c315bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

成功之后

![image.png](http://upload-images.jianshu.io/upload_images/1401554-89484ce880c82224.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)


修改图中2处，增加自己的文件后
![image.png](http://upload-images.jianshu.io/upload_images/1401554-c68da6856c2db103.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)



cd到spec所在的目录
执行`pod lib lint` 命令校对文件是否可用,不行使用如下命令
__ `pod lib lint --verbose --use-libraries` __

出现坑：

![image.png](http://upload-images.jianshu.io/upload_images/1401554-3b53a0ab400e2fb7.png)


Xcode > Preferences > Locations 
![image.png](http://upload-images.jianshu.io/upload_images/1401554-59849cff4dbb2849.png)

依赖自己定义的私有库时：
```
pod lib lint XXXX.podspec --sources='https://git.XXXX.com/xxx.git,https://github.com/CocoaPods/Specs' --allow-warnings --use-libraries --verbose --no-clean
```

指定路径才可以验证通过，可能就是自己指定路径的时候使用你自己定义的路径，结果我只写一个sources，没有＋后面的cocoapods的路径，憋了一天没过
![image.png](https://upload-images.jianshu.io/upload_images/1401554-cc127a2a8e5befaf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




![](http://upload-images.jianshu.io/upload_images/1401554-50b7bdeeeccb660f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


图片拖入assets里面，`pod update` 之后就有resource文件