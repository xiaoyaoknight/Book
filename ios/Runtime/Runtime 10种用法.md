# Runtime 10种用法

##### runtime简介：
Runtime 又叫运行时，是一套底层的 C 语言 API，其为 iOS 内部的核心之一，我们平时编写的 OC 代码，底层都是基于它来实现的。

人在江湖飘，身不由己啊。循环不停的业务，都几乎没有时间去总结。就是会用一些，但是真正想下runtime这个。自己还真说不出个所以然来。

爬爬帖子，给自己留个印儿。

 [OC最实用的runtime总结，面试、工作你看我就足够了！](http://www.jianshu.com/p/ab966e8a82e2)
[让你快速上手Runtime](http://www.jianshu.com/p/e071206103a4)
[runtime详解](http://www.jianshu.com/p/46dd81402f63)
[详解runtime运行时机制](http://www.jianshu.com/p/1e06bfee99d0) 
[万能控制器跳转](http://www.cocoachina.com/ios/20150824/13104.html)

看一张图：
![image.png](https://upload-images.jianshu.io/upload_images/1401554-73a312da012b3813.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

##十大用法：
- 1.将某些OC代码转为运行时代码，探究底层，比如block的实现原理
- 2.**拦截系统自带的方法调用（Swizzle 黑魔法）,也可以说成对系统的方法进行替换**，比如拦截imageNamed:、viewDidLoad、alloc
- 3.实现分类也可以增加属性；
- 4.实现NSCoding的自动归档和自动解档；(不用对每个属性edcode和decode了,如果几十个属性一个个的encode和decode真的很麻烦啊,使用运行时可以遍历出每个对象的属性,数组的方式遍历eccode,decode)
- 5.实现字典和模型的自动转换(核心就是可以遍历出字典中的每个属性,json解析中大牛框架都用了这个特性,包括MJEXtension,YYModel，jsonModel都是将json转换为字典,再遍历字典中的每个属性来进行modle的转换)。
- 6.动态增加方法  (动态的为某个类或对象增加一个方法,摘录文章中有详细介绍)
- 7.动态变量控制  (动态对某个对象的变量的值进行替换,摘录文章有详细介绍)
- 8.**实现万能控制器跳转**
>产品来一变态需求,推送过来的消息,要跳转到任意控制器.利用runtime动态生成对象、属性、方法这特性，我们可以先跟服务端商量好，定义跳转规则，比如要跳转到A控制器，需要传属性id、type，那么服务端返回字典给我，里面有控制器名，两个属性名跟属性值，客户端就可以根据控制器名生成对象，再用kvc给对象赋值，这样就搞定了。

>组件化其实也用到了runtime，根据url，动态获取到类，参数等。可以参考蘑菇街开源项目
- 9.插件开发 
- 10.Jspath 热更新 也是使用运行时，jspatch 基本上算是黑科技，在线修复版本bug，虽然被苹果爸爸禁止使用了一段时间，后来好像谈妥了，又可以付费使用了。

#####总结：
综上所述，runtime核心用法有以下三点。
```
Method class_getClassMethod(Class cls , SEL name) //获得某个类的类方法
```

```
Method class_getInstanceMethod(Class cls , SEL name) // 获得某个类的实例对象方法
```
```
void method_exchangeImplementations(Method m1 , Method m2) // 交换两个方法的实现
```

##### 1、方法简单的交换
下面通过runtime 实现方法交换，类方法用class_getClassMethod ，对象方法用class_getInstanceMethod
```
// 获取两个类的类方法
Method m1 = class_getClassMethod([Person class], @selector(run));
Method m2 = class_getClassMethod([Person class], @selector(study));
// 开始交换方法实现
method_exchangeImplementations(m1, m2);
// 交换后，先打印学习，再打印跑！
[Person run];
[Person study];
```
##### 2、拦截系统方法
步骤：
1、为UIImage建一个分类（UIImage+Category）
2、在分类中实现一个自定义方法，方法中写要在系统方法中加入的语句，比如版本判断
```
+ (UIImage *)xh_imageNamed:(NSString *)name {
    double version = [[UIDevice currentDevice].systemVersion doubleValue];
    if (version >= 7.0) {
        // 如果系统版本是7.0以上，使用另外一套文件名结尾是‘_os7’的扁平化图片
        name = [name stringByAppendingString:@"_os7"];
    }
    return [UIImage xh_imageNamed:name];
}
```
3、分类中重写UIImage的load方法，实现方法的交换（只要能让其执行一次方法交换语句，load再合适不过了）
```
+ (void)load {
    // 获取两个类的类方法
    Method m1 = class_getClassMethod([UIImage class], @selector(imageNamed:));
    Method m2 = class_getClassMethod([UIImage class], @selector(xh_imageNamed:));
    // 开始交换方法实现
    method_exchangeImplementations(m1, m2);
}
```
>注意：自定义方法中最后一定要再调用一下系统的方法，让其有加载图片的功能，但是由于方法交换，系统的方法名已经变成了我们自定义的方法名（有点绕，就是用我们的名字能调用系统的方法，用系统的名字能调用我们的方法），这就实现了系统方法的拦截！

##### 3、获得一个类的所有成员变量
```
Ivar *class_copyIvarList(Class cls , unsigned int *outCount) //获得一个类的所有成员变量
```
```
const char *ivar_getName(Ivar v) //获得成员变量的名字 
```
```
const char *ivar_getTypeEndcoding(Ivar v) //获得成员变量的类型
```

```
//获取Person类中所有成员变量的名字和类型
unsigned int outCount = 0;
Ivar *ivars = class_copyIvarList([Person class], &outCount);

// 遍历所有成员变量
for (int i = 0; i < outCount; i++) {
    // 取出i位置对应的成员变量
    Ivar ivar = ivars[i];
    const char *name = ivar_getName(ivar);
    const char *type = ivar_getTypeEncoding(ivar);
    NSLog(@"成员变量名：%s 成员变量类型：%s",name,type);
}
// 注意释放内存！
free(ivars);
```
参考文章：
https://blog.csdn.net/qq_30513483/article/details/51312848
 