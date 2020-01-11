# iOS底层原理总结 - 探寻Runloop本质

## 什么是RunLoop？RunLoop的本质：
[Runloop-Demo](https://github.com/xiaoyaoknight/Runloop-Demo)
RunLoop 的概念：顾名思义，运行循环，在程序运行过程中循环做一些事情。
![01.png](https://upload-images.jianshu.io/upload_images/1401554-12ec11071c47e4e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####思考：我们应用程序中的main函数为什么可以保持无退出呢？
![image.png](https://upload-images.jianshu.io/upload_images/1401554-73bc9f182fc83736.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![02.png](https://upload-images.jianshu.io/upload_images/1401554-310a1b56b4336038.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实际上呢，在我们的main函数中会调用UIApplicationMain函数，在这个函数中会启动一个运行循环（也就是我们所说的RunLoop），在这个运行循环中可以处理很多事件，例如屏幕的点击，滑动列表，或者网络请求的返回等等，在处理完事件之后，会进入等待，在这个循环中，并不是一个单纯的for循环或者while循环，而是从用户态到内核态的切换，以及再从内核态到用户态的切换，这里面的等待也不等于死循环，这里面最重要的是状态的切换

## Runloop对象
iOS中提供了两个这样的对象：`NSRunLoop` 和 `CFRunLoopRef`
`CFRunLoopRef` 是在 `CoreFoundation` 框架内的，它提供了纯 C 函数的 API，所有这些 API 都是线程安全的。
`NSRunLoop` 是基于 `CFRunLoopRef `的封装，提供了面向对象的 API，但是这些 API 不是线程安全的。
`CFRunLoopRef `的代码是[开源](http://opensource.apple.com/source/CF/CF-855.17/CFRunLoop.c)的，你可以在[这里](http://opensource.apple.com/tarballs/CF/) 下载到整个 `CoreFoundation` 的源码来查看。
Swift 开源后，苹果又维护了一个跨平台的 [CoreFoundation 版本](https://github.com/apple/swift-corelibs-foundation/)，这个版本的源码可能和现有 iOS 系统中的实现略不一样，但更容易编译，而且已经适配了 Linux/Windows。
![image.png](https://upload-images.jianshu.io/upload_images/1401554-08e6d30b6db800b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

## [RunLoop的数据结构](https://www.jianshu.com/p/b3bfe5261a5d)
##[Runloop对外的接口](https://www.jianshu.com/p/d6137639b278)
## [RunLoop 的 Mode](https://www.jianshu.com/p/8648a64ea562)
##[RunLoop 的内部逻辑即事件循环机制](https://www.jianshu.com/p/7db5b2775623)
##[RunLoop 的底层实现](https://www.jianshu.com/p/e62c3d90343d)
##[RunLoop 与线程的关系](https://www.jianshu.com/p/d2023ae441a1)
##[苹果用 RunLoop 实现的功能](https://www.jianshu.com/p/d0e8c13e3da8)
##[RunLoop 的实际应用举例](https://www.jianshu.com/p/e9b4fafcbd22)
##[RunLoop相关问题总结](https://www.jianshu.com/p/ebb3e025133b)
##[Run Loops 官方文档](https://www.jianshu.com/p/fdba4c990387)
##[Runloop图解](https://www.jianshu.com/p/182e8d124da7)


参考文章：
[RunLoop本质、数据结构以及常驻线程实现](https://juejin.im/post/5c1cb81be51d45745728ef9f)
[深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)
