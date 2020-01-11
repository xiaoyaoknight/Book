# iOS底层原理总结篇-- 深入理解 KVC 实现机制

**面试题：**

```
KVC相关： 
1\. 通过KVC修改属性会触发KVO么？
2\. KVC的赋值和取值过程是怎样的？原理是什么？
```
## KVC的实现原理[Demo](https://github.com/xiaoyaoknight/KVC-Demo)

### 1\. 什么是KVC？

```
KVC的全称key - value - coding，俗称"键值编码",可以通过key来访问某个属性
```

常见的API有：

```
- (void)setValue:(id)value forKey:(NSString *)key;
- (void)setValue:(id)value forKeyPath:(NSString *)keyPath;

- (id)valueForKey:(NSString *)key; 
- (id)valueForKeyPath:(NSString *)keyPath;
```

简单的代码实现: DLPerson 和 DLCat

```
///> DLPersin.h 文件

#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

/**
 DLCat
 */
@interface DLCat : NSObject
@property (nonatomic, assign) int weight;
@end

/**
 DLPerson
 */
@interface DLPerson : NSObject
@property (nonatomic, assign) int age;
@property (nonatomic, strong) DLCat *cat;
@end

NS_ASSUME_NONNULL_END
```

#### 1.1 - (void)setValue:(id)value forKey:(NSString *)key;

```
///> ViewController.m 文件

- (void)viewDidLoad {
    [super viewDidLoad];
    DLPerson *person = [[DLPerson alloc]init];
    [person setValue:@20 forKey:@"age"];
    NSLog(@"%d",person.age);
}
```

#### 1.1 - (void)setValue:(id)value forKeyPath:(NSString *)keyPath;

```
///> ViewController.m 文件

- (void)viewDidLoad {
    [super viewDidLoad];
    DLPerson *person = [[DLPerson alloc]init];
    person.cat = [[DLCat alloc]init];
    [person setValue:@20 forKeyPath:@"cat.weight"];
    NSLog(@"%d",person.age);
    NSLog(@"%d",person.cat.weight);
}
```

**`setValue:(id)value forKeyPath:(NSString *)keyPath`** 和 **`setValue:(id)value forKey:(NSString *)key`** 的区别：

*   keyPath 相当于根据路径去寻找属性，一层一层往下找，
*   key 是直接哪去属性的名字设置，如果按路径找会报错

#### 1.3 - (id)valueForKey:(NSString *)key;

```
///> ViewController.m 文件

- (void)viewDidLoad {
    [super viewDidLoad];
    DLPerson *person = [[DLPerson alloc]init];
    person.age = 10;
    NSLog(@"%@",[person valueForKey:@"age"]);
}
```

#### 1.4 - (id)valueForKeyPath:(NSString *)keyPath;

```
///> ViewController.m 文件

- (void)viewDidLoad {
    [super viewDidLoad];
    DLPerson *person = [[DLPerson alloc]init];
    person.age = 10;
    NSLog(@"%@",[person valueForKey:@"cat.weight"]);
}
```

**`(id)valueForKey:(NSString *)key;`** 和 **`(id)valueForKeyPath:(NSString *)keyPath`** 的区别：

*   同理1.2-1.3

### 2\. setValue:forKey:的原理

![image.png](https://upload-images.jianshu.io/upload_images/1401554-ea8712ce7776f692.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


*   当我们设置setValue:forKey:时
*   首先会查找setKey:、_setKey: (按顺序查找)
*   如果有直接调用
*   如果没有，先查看accessInstanceVariablesDirectly方法

    ```
    + (BOOL)accessInstanceVariablesDirectly{
          return YES;   ///> 可以直接访问成员变量
      //    return NO;  ///>  不可以直接访问成员变量,  
      ///> 直接访问会报NSUnkonwKeyException错误  
      }
    ```

*   如果可以访问会按照 _key、_isKey、key、iskey的顺序查找成员变量
*   找到直接复制
*   未找到报错NSUnkonwKeyException错误

_key、_isKey、key、iskey复制顺序

```
///> DLPerson.h 文件

@interface DLPerson : NSObject{
    @public
    int _age;
    int _isAge;
    int age;
    int isAge;
}

@end
```

```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.person1 = [[DLPerson alloc]init];
    ///> person1添加kvo监听
    NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
    [self.person1 addObserver:self forKeyPath:@"age" options:options context:nil];

    ///> 通过KVC修改person.age的值
    [self.person1 setValue:@20 forKey:@"age"];

     NSLog(@"------");
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{
    NSLog(@"监听到了%@的%@属性发生了改变%@",object,keyPath,change);
}

- (void)dealloc{
    ///> 使用结束后记得移除
    [self.person1 removeObserver:self forKeyPath:@"age"];
}
```

*   在NSLog(@"------");位置打下短点注释方式去查看各个值得复制顺序
*   然后在控制台中查看复制查找的顺序就OK了，
*   DLPerson文件中的顺序即使是调换了也无所谓。

### 3\. valueForKey:的原理

![image.png](https://upload-images.jianshu.io/upload_images/1401554-b681a2102953b9df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

*   kvc取值按照 getKey、key、iskey、_key 顺序查找方法
*   存在直接调用
*   没找到同样，先查看accessInstanceVariablesDirectly方法

    ```
    + (BOOL)accessInstanceVariablesDirectly{
          return YES;   ///> 可以直接访问成员变量
      //    return NO;  ///>  不可以直接访问成员变量,  
      ///> 直接访问会报NSUnkonwKeyException错误  
      }
    ```

*   如果可以访问会按照 _key、_isKey、key、iskey的顺序查找成员变量
*   找到直接复制
*   未找到报错NSUnkonwKeyException错误

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

复制代码
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

**然后我们打入断点去查看实现的方法：**

![image.png](https://upload-images.jianshu.io/upload_images/1401554-eae82b1a226b8972.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


在控制台中使用 **`p (IMP)方法地址`** 来打印得到方法的名称。 所以我们在添加KVO之后的 **`setAge:`**方法调用了 **`_NSSetIntValueAndNotify()`** 。

```
如果定义的属性是类型是double则调用的是_NSSetDoubleValueAndNotify()
大家可以自己测试一下。
此方法在Foundtion框架中有对应的
NSSetDoubleValueAndNotify()
NSSetIntValueAndNotify()
NSSetCharValueAndNotify()
...
```

**`目前还未深入接触到逆向工程。等以后学到了在给大家详解解释吧。`**

### 2\. NSKVONotifying_DLPerson的isa指针指向哪里？

在 **KVO的本质分析** 中我们得知，添加了KVO监听的实例对象isa指针指向了NSKVONotifying_DLPerson类， 那么NSKVONotifying_DLPerson的isa指针的指向？

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

结果发现：每一个类对象的地址是不一样的，而且元类对象的地址也不一样的，所以我们可以认为 NSKVONotifying_DLPerson类有自己的元类对象， NSKVONotifying_DLPerson.isa指向着自己的元类对象。

## 四. 面试题答案

1.  iOS用什么方式实现对一个对象的KVO？(KVO的本质是什么？)

    ```
     利用RuntimeAPI动态生成一个子类，并且让instance对象的isa指向这个全新的子类
     当修改instance对象的属性时，会调用Foundation的_NSSetXXXValueAndNotify函数
     willChangeValueForKey:
     父类原来的setter
     didChangeValueForKey:
     内部会触发监听器（Oberser）的监听方法(observeValueForKeyPath:ofObject:change:context:）
    复制代码
    ```

2.  如何手动触发KVO？

    ```
     手动调用willChangeValueForKey:和didChangeValueForKey:
    ```

3.  直接修改成员变量会触发KVO么？

    ```
     不会触发KVO，因为直接修改成员变量并没有走set方法。
    ```

KVC相关：

1.  通过KVC修改属性会触发KVO么？

    ```
     会触发KVO，如上流程图
    ```

2.  KVC的赋值和取值过程是怎样的？原理是什么？

    ```
     如上流程图
    ```

* * *

*   文章总结自MJ老师底层视频。