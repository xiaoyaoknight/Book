# iOS底层原理总结篇-- 深入理解 KVO 实现机制

## 一. KVO的实现原理

**面试题：**

```
KVO相关：
1\. iOS用什么方式来实现对一个对象的KVO？（KVO的本质是什么？）
2\. 如何手动出发KVO？
3\. 直接修改成员变量会触发KVO么？
```

### 1\. 什么是KVO？

```
KVO的全称是`Key-Value Observing`，俗称"键值监听"，可以用于监听摸个对象属性值得改变。
```

![image.png](https://upload-images.jianshu.io/upload_images/1401554-a1b5003a66d38887.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


要监听Person中的age属性，我们就创建一个`observer`用来监听age的变化，当我们age一旦发生改变，就会通知observer。

### 2\. KVO简单的实现

我们先简单的回顾一下 KVO的代码实现。[Demo](https://github.com/xiaoyaoknight/KVO-Demo)

```
///> DLPerson.h 文件

#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface DLPerson : NSObject

@property (nonatomic, assign) int age;

@end

NS_ASSUME_NONNULL_END
```

```
///> ViewController.m 文件

#import "ViewController.h"
#import "DLPerson.h"
@interface ViewController ()
@property (nonatomic, strong) DLPerson *person1;
@property (nonatomic, strong) DLPerson *person2;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.person1 = [[DLPerson alloc]init];
    self.person1.age = 1;

    self.person2 = [[DLPerson alloc]init];
    self.person2.age = 2;

    ///> person1添加kvo监听
    NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
    [self.person1 addObserver:self forKeyPath:@"age" options:options context:nil];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    self.person1.age = 20;
    self.person2.age = 30;
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{
    NSLog(@"监听到了%@的%@属性发生了改变%@",object,keyPath,change);
}

- (void)dealloc{
    ///> 使用结束后记得移除
    [self.person1 removeObserver:self forKeyPath:@"age"];
}

@end
```

```
输出结果：
监听到了<DLPerson: 0x6000033d4e40>的age属性发生了改变- {
kind = 1;
new = 20;
old = 1;
}

///>  因为我们只监听了person1  所以只会输出person1的改变。
```

### 3\. KVO存在的疑问

```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    // self.person1.age = 20;
    [self.person1 setAge:20]; ///> 等同于self.person1.age = 20;

    self.person2.age = 30;
    [self.person2 setAge:20];///> 等同于self.person2.age = 20;
}
```

因为当我们在DLPerson中使用@property声名一个属性的时候会自动生成声名属性的set和get方法。如下代码：

```
///> DLPerson.m文件

#import "DLPerson.h"

@implementation DLPerson

- (void)setAge:(int)age{
    _age = age;
}

- (int)age{
    return _age;
}

@end
```

既然person1和person2的本质都是在调用set方法，就一定都会走在DLPerson类中的setAge这个方法。

那么问题来了，同样走的是DLPerson类中的setAge方法，为什么person1就会走到

```
- (void)observeValueForKeyPath:(nullable NSString *)keyPath ofObject:(nullable id)object change:(nullable NSDictionary<NSKeyValueChangeKey, id> *)change context:(nullable void *)context;
```

方法中而person2就不会呢？

### 4\. KVO的本质分析

如果不是很了解OC对象的isa指针相关知识的同学，建议先请前往[iOS底层原理总结篇-- 探寻OC对象的本质](https://juejin.im/post/5c1b5db55188250850605c44) 文章了解一下先。

接下来我们探究一下两个对象的本质，首先我们`person1`和`person2`的`isa`打印出来查看一下他们的实例对象isa指向的类对象是什么？

![image.png](https://upload-images.jianshu.io/upload_images/1401554-ad37b338ce2949f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


我们会发现person1的isa指针打印出的是: **`NSKVONotifying_DLPerson`**

而person2的isa指针打印出来的是: **`DLPerson`**

person1和person2都是实例对象 所以person1和person2的isa指针指向的都是类对象，

**`所以说，如果对象没有添加KVO监听那么它的isa指向的就是自己原来的类对象，如下图`**

```
    person2.isa == DLPerson
```

![image.png](https://upload-images.jianshu.io/upload_images/1401554-02ffbd5843e27a49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


**`当一个对象添加了KVO的监听时，当前对象的isa指针指向的就不是你原来的类，指向的是另外一个类对象，如下图`**

```
    person1.isa == NSKVONotifying_DLPerson
```

![image.png](https://upload-images.jianshu.io/upload_images/1401554-755824ad4a47fcb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


*   NSKVONotifying_DLPerson类是 Runtime动态创建的一个类，在程序运行的过程中产生的一个新的类。
*   NSKVONotifying_DLPerson类是DLPerson的一个子类。
*   NSKVONotifying_DLPerson类存在自己的 setAge:、class、dealloc、isKVOA...方法。
    **`当我们DLperson的实例对象调用setAge方法时，`**
    **`实例对象的isa指针找到类对象，然后在类类对象中寻找相应的对象方法，如果有则调用，`**
    **`如果没有则去superclass指向的父类对象中寻找相应的对象方法，如果有则调用，`**
    **`如果未找到相应的对象方法则会报：unrecognized selector sent to instance 错误`**
*   由于上图可分析出我们的person1的isa指针指向的类对象是`NSKVONotifying_DLPerson`，并且`NSKVONotifying_DLPerson`中还带有setAge: 方法，所以：
    ```
    ///> person1的setAge方法走的是NSKVONotifying_DLPerson类中的setAge方法，
    ///> 并且在KVO监听中实现了[superclass setAge:age];，
    [self.person1 setAge:20]; 

    ///> person2的setAge方法走的是DLPerson中的setAge:方法，
    [self.person2 setAge:30]; 
    ```
    上次代码所示，两个实例对象person1和person2都走了DLPerson的setAge:方法，只是添加了KVO的person1在自己的setAge方法中添加了 **`其他操作`**。

* 传递person真正的类，查看方法列表
```
 [self printMethodNameOfClass:object_getClass(self.person1)];
 [self printMethodNameOfClass:object_getClass(self.person2)];

- (void)printMethodNameOfClass:(Class)cls {
    unsigned int count;
    // 获得方法数组
    Method *methodList = class_copyMethodList(cls, &count);
    // 存储方法名
    NSMutableString *methodNames = [NSMutableString string];
    // 遍历所有方法
    for (int i = 0; i < count; i++) {
        // 获得方法
        Method method = methodList[i];
        
        // 获得方法名
        NSString *methodName = NSStringFromSelector(method_getName(method));
        
        // 拼接方法
        [methodNames appendString:methodName];
        [methodNames appendString:@","];
    }
    // 释放
    free(methodList);
    
    NSLog(@"%@, %@", cls, methodNames);
}
```
* 打印结果：
![image.png](https://upload-images.jianshu.io/upload_images/1401554-62351d7ca4c272d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

*   其他操作猜想 **伪代码！**：

    ```
    ///> NSKVONotifying_DLPerson.m 文件

    #import "NSKVONotifying_DLPerson.h"

    @implementation NSKVONotifying_DLPerson

    - (void)setAge:(int)age{
      _NSSetIntValueAndNotify();  ///> 文章末尾 知识点补充小结有此方法来源
    }

    void _NSSetIntValueAndNotify(){
      [self willChangeValueForKey:@"age"];
      [super setAge:age];
      [self didChangeValueForKey:@"age"];
    }

    - (void)didChangeValueForKey:(NSString *)key{
      ///> 通知监听器 key发生了改变
      [observe observeValueForKeyPath:key ofObject:self change:nil context:nil];
    }

    @end
    ```

    **`_NSSetIntValueAndNotify();`**




### 5\. KVO的调用顺序

由于我们的NSKVONotifying_DLPerson类不能参与编译所以可以在 DLPerson.m中重写它父类的方法代码如下：

```
///> ViewController.m文件

#import "ViewController.h"
#import "DLPerson.h"
@interface ViewController ()
@property (nonatomic, strong) DLPerson *person1;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.person1 = [[DLPerson alloc]init];
    self.person1.age = 10;

    ///> person1添加kvo监听
    NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
    [self.person1 addObserver:self forKeyPath:@"age" options:options context:nil];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    [self.person1 setAge:20];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{
    NSLog(@"监听到了%@的%@属性发生了改变%@",object,keyPath,change);
}

- (void)dealloc{
    ///> 使用结束后记得移除
    [self.person1 removeObserver:self forKeyPath:@"age"];
}

@end
```

```
///> DLPerson.m文件

#import "DLPerson.h"

@implementation DLPerson

- (void)setAge:(int)age{
    _age = age;
}

- (void)willChangeValueForKey:(NSString *)key{
    [super willChangeValueForKey:key];
    NSLog(@"willChangeValueForKey");
}

- (void)didChangeValueForKey:(NSString *)key{
    NSLog(@"didChangeValueForKey - begin");
    [super didChangeValueForKey:key];
    NSLog(@"didChangeValueForKey - end");
}
@end
```

输出结果：

```
willChangeValueForKey
didChangeValueForKey - begin
监听到了<DLPerson: 0x60000041afe0>的age属性发生了改变{
    kind = 1;
    new = 20;
    old = 10;
}
didChangeValueForKey - end
```

*   调用willChangeValueForKey:
*   调用原来的setter实现
*   调用didChangeValueForKey:

**`总结：didChangeValueForKey:内部会调用observer的observeValueForKeyPath:ofObject:change:context:方法`**

## 三. 知识点补充

### 1\. _NSSetIntValueAndNotify();方法来源

```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.person1 = [[DLPerson alloc]init];
    self.person1.age = 10;

    self.person2 = [[DLPerson alloc]init];
    self.person2.age = 20;

    ///> person1添加kvo监听
    NSLog(@"person添加KVO之前 - person1：%@， person2：%@",object_getClass(self.person1), object_getClass(self.person2));

    NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
    [self.person1 addObserver:self forKeyPath:@"age" options:options context:nil];

    NSLog(@"person添加KVO之前 - person1：%@， person2：%@",object_getClass(self.person1), object_getClass(self.person2));
}
```

输出结果：

```
person添加KVO之前 - person1：DLPerson， person2：DLPerson

person添加KVO之后 - person1：NSKVONotifying_DLPerson， person2：DLPerson
```

由此可见在没有 为person1添加KVO之前 person1.isa指针仍然是DLPerson

**`那么我们就可以使用- (IMP)methodForSelector:(SEL)aSelector;去查看实现方法的地址，的具体方法代码如下：`**

```
///> person1添加kvo监听
    NSLog(@"person添加KVO之前 - person1：%p， person2：%p \n",[self.person1 methodForSelector:@selector(setAge:)], [self.person2 methodForSelector:@selector(setAge:)]);

    NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
    [self.person1 addObserver:self forKeyPath:@"age" options:options context:nil];

    NSLog(@"person添加KVO之后 - person1：%p， person2：%p \n",[self.person1 methodForSelector:@selector(setAge:)], [self.person2 methodForSelector:@selector(setAge:)]);
}
```

输出结果：

```
person添加KVO之前 - person1：0x10852a560， person2：0x10852a560 
person添加KVO之后 - person1：0x108883fc2， person2：0x10852a560 
```

由此可见，在添加之前person1和person2实现的setAge方法是一个，添加之后person1的setAge方法就有了变化。

**然后我们打入短点去查看实现的方法：**

![image.png](https://upload-images.jianshu.io/upload_images/1401554-0cd116000de94cf6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在控制台中使用 **`p (IMP)方法地址`** 来打印得到方法的名称。 所以我们在添加KVO之后的 **`setAge:`** 方法调用了 **`_NSSetIntValueAndNotify()`** 。

```
如果定义的属性是类型是double则调用的是_NSSetDoubleValueAndNotify()
大家可以自己测试一下。
此方法在Foundtion框架中有对应的
NSSetDoubleValueAndNotify()
NSSetIntValueAndNotify()
NSSetCharValueAndNotify()
...
```

### 2\. NSKVONotifying_DLPerson的isa指针指向哪里？

在 **KVO的本质分析** 中我们得知，添加了KVO监听的实例对象isa指针指向了NSKVONotifying_DLPerson类， 那么NSKVONotifying_DLPerson的isa指针的指向？

```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.person1 = [[DLPerson alloc]init];
    self.person1.age = 10;

    self.person2 = [[DLPerson alloc]init];
    self.person2.age = 20;

    ///> person1添加kvo监听
    NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
    [self.person1 addObserver:self forKeyPath:@"age" options:options context:nil];

    NSLog(@"类对象 - person1: %@<%p>   person2: %@<%p>",
          object_getClass(self.person1),  ///> self.person1.isa  类名
          object_getClass(self.person1),  ///> self.person1.isa
          object_getClass(self.person2),  ///> self.person1.isa  类名
          object_getClass(self.person2)  ///> self.person1.isa
          );

    NSLog(@"元类对象 - person1: %@<%p>   person2: %@<%p>",
          object_getClass(object_getClass(self.person1)),  ///> self.person1.isa.isa 类名
          object_getClass(object_getClass(self.person1)),  ///> self.person1.isa.isa
          object_getClass(object_getClass(self.person2)),  ///> self.person2.isa.isa 类名
          object_getClass(object_getClass(self.person2))  ///> self.person2.isa.isa
          );
}
```

输出结果：

```
类对象 - person1: NSKVONotifying_DLPerson<0x6000002cef40>   person2: DLPerson<0x1002c9048>
元类对象 - person1: NSKVONotifying_DLPerson<0x6000002cf210>   person2: DLPerson<0x1002c9020>
```

结果发现：每一个类对象的地址是不一样的，而且元类对象的地址也不一样的，所以我们可以认为 `NSKVONotifying_DLPerson`类有自己的`元类对象`， `NSKVONotifying_DLPerson.isa`指向着自己的元类对象。


