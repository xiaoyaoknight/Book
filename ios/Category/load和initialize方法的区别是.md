# load和initialize方法的区别是什么?

## [load](https://www.jianshu.com/p/b73ee3a91c7d)和[initialize](https://www.jianshu.com/p/e79a22e87c10)方法的区别是什么?[Demo地址](https://github.com/xiaoyaoknight/TestCategroy.git)
#### 调用方式
1、load是根据函数地址直接调用
2、initialize是通过objc_msgSend调用


#### 调用时刻
1、load是runtime加载类、分类的时候调用(只会调用一次)
2、initialize是类第一次接收到消息的时候调用, 每一个类只会initialize一次(如果子类没有实现initialize方法, 会调用父类的initialize方法, 所以父类的initialize方法可能会调用多次)

#### load和initializee的调用顺序

1、load:
先调用类的load, 在调用分类的load
先编译的类, 优先调用load, 调用子类的load之前, 会先调用父类的load
先编译的分类, 优先调用load
![image.png](https://upload-images.jianshu.io/upload_images/1401554-d3e39f333497354d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

2、initialize
先初始化分类, 后初始化子类
通过消息机制调用, 当子类没有initialize方法时, 会调用父类的initialize方法, 所以父类的initialize方法会调用多次
![image.png](https://upload-images.jianshu.io/upload_images/1401554-cfef0a47ad241c5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
