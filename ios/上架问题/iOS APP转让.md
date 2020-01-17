# iOS APP转让

#### 1、登录iTunes发布网站，找到app，查看App信息下方有个【转让 App】 按钮。
（如果看不到转让按钮，说明账号可能是子账号没有权限。还有一种情况是苹果协议没有遵守，可以打苹果客服咨询）
![image](http://upload-images.jianshu.io/upload_images/1401554-376524efccde6a02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

#### 2、点进去，是如下界面，里面包含是否满足转让条件，全部符合条件才可以进行转让

![image](http://upload-images.jianshu.io/upload_images/1401554-7915b5a4dfa24aaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

***[移除要转让的 App 的所有构建版本和测试员解决方案](http://www.cnblogs.com/yajunLi/p/6927770.html)***

这个是由于TestFlight里面有遗留历史构建版本导致的，我们只要进去清除掉就可以了。
- 进入TestFlight，查看iOS构建版本，右侧会看到一个列表
- 将列表里的历史构建版本全部设为过期版本就可以了

![移除测试人员.png](http://upload-images.jianshu.io/upload_images/1401554-38aa2aa269db2fe8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
等全部清除了，再次点击app转让就没有这个错了。。。


如果都没问题，点继续，进行下一步
![image](http://upload-images.jianshu.io/upload_images/1401554-5fcbbc6a2c70b1bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

#### 3、找到接收方事先给你的AppID和TeamID（很重要，必须需要这两个账号），注意，这里的两个账号是接收方提供的，不是我们自己的。[查询teamid](https://developer.apple.com/account/#/membership/3DJN5WPK37)

    其中：AppID就是登陆网站的账号，TeamID可在Member信息里查看。

![image](http://upload-images.jianshu.io/upload_images/1401554-5eb5ea9dcd2d6dc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

#### 4、填写完上面的账号，直接继续，会看到这个协议，点同意，并下一步【请求转让】

![image](http://upload-images.jianshu.io/upload_images/1401554-5620d2c13744924b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

#### 5、这样，一套流程就走完了，还是很快的。

然后，回到主页，你会看到这个，可以点进去【查看详情】或【撤销转让】。

![image](http://upload-images.jianshu.io/upload_images/1401554-7d3f2cae93aee9de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

#### 6、最后，提醒接收方，登录iTunes网站，就可以在首页看的提醒有app转让等提示通知了，点进去就可以进行接收了，整个流程基本在半小时左右解决啦







<br>
参考链接：
[app转让参考链接](https://www.jianshu.com/p/57bc6d229be2)
[[iOS转让app-您必须移除要转让的 App 的所有构建版本和测试员，并清除“测试信息”下的所有信息字段解决方案](http://www.cnblogs.com/yajunLi/p/6927770.html)](http://www.cnblogs.com/yajunLi/p/6927770.html)
[App转让遇到如下错误：您必须接受最新版的主协议，才能开始转移协议
](http://www.cocoachina.com/bbs/read.php?tid=317559)
