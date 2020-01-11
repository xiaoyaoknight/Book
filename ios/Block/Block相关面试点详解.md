# Block相关面试点详解
#### block的原理是怎样的？本质是什么？

![image.png](https://upload-images.jianshu.io/upload_images/1401554-593d1db9e6aae513.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1401554-189f8dc4403a9529.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1401554-656a47920aba434c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> auto:代表自动变量，离开作用域就销毁
> static:将变量的地址传到block

![image.png](https://upload-images.jianshu.io/upload_images/1401554-c8774a4ef5138fd7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> block分为三种类型。global类型不需要太过注意，需要注意stack类型转换为malloc类型。只有block在堆上时我们才可以对其进行管理。

![image.png](https://upload-images.jianshu.io/upload_images/1401554-9b19591f58baddbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1401554-d7430cda16fe8000.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1401554-278ccce8dbf86698.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1401554-d1e31106ec3da209.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1401554-64c985a9d6112194.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 封装了函数调用以及调用环境的OC对象

#### __blcok的作用是什么？有什么使用注意点？

![image.png](https://upload-images.jianshu.io/upload_images/1401554-b0bc3c83ea55be7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1401554-dea80c2fe347b8ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1401554-eb28d11306dd212b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### block的属性修饰词为什么是copy？使用block有哪些使用注意？

> 如果不copy的话，那么block就不会在堆空间上，无法对你生命周期进行控制。需要注意循环引用(ARC环境下 strong 、copy没有区别)

#### block在修改NSMutableArray内容时，需不需要添加__blcok?

> 不需要。修改内容也是对数组的使用，只有对对象赋值的时候才需要__block。

原文链接：https://www.jianshu.com/p/e9189f550192
