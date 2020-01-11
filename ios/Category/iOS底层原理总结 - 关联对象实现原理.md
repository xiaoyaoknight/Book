# iOS底层原理总结 - 关联对象实现原理

## 思考： Category能否添加成员变量？如果可以，如何给Category添加成员变量？
答：不能直接添加成员变量，但是可以通过runtime的方式间接实现添加成员变量的效果。
[Demo](https://github.com/xiaoyaoknight/TestCategroy)
## RunTime为Category动态关联对象

使用RunTime给系统的类添加属性，首先需要了解对象与属性的关系。我们通过之前的学习知道，对象一开始初始化的时候其属性为nil，给属性赋值其实就是让属性指向一块存储内容的内存，使这个对象的属性跟这块内存产生一种关联。

那么如果想动态的添加属性，其实就是动态的产生某种关联就好了。而想要给系统的类添加属性，只能通过分类。

这里给NSObject添加name属性，创建NSObject的分类 我们可以使用@property给分类添加属性
```
@property(nonatomic,strong)NSString *name;
```

**通过[iOS底层原理总结篇--  Category的本质](https://www.jianshu.com/p/d3e6839921dd)
我们知道，虽然在分类中可以写@property 添加属性，但是不会自动生成私有属性，也不会生成set,get方法的实现，只会生成set,get的声明，需要我们自己去实现。**

##### 方法一：我们可以通过使用静态全局变量给分类添加属性

```
static NSString *_name;
-(void)setName:(NSString *)name
{
    _name = name;
}
-(NSString *)name
{
    return _name;
}
```

但是这样_name静态全局变量与类并没有关联，无论对象创建与销毁，只要程序在运行_name变量就存在，并不是真正意义上的属性。

##### 方法二：使用RunTime动态添加属性

RunTime提供了动态添加属性和获得属性的方法。

```
-(void)setName:(NSString *)name
{
    objc_setAssociatedObject(self, @"name",name, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
-(NSString *)name
{
    return objc_getAssociatedObject(self, @"name");    
}
```

1.  动态添加属性

```
objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);
```

参数一：**`id object`** : 给哪个对象添加属性，这里要给自己添加属性，用self。 
参数二：**`void * == id key`** : 属性名，根据key获取关联对象的属性的值，在**`objc_getAssociatedObject`**中通过次key获得属性的值并返回。
参数三：**`id value`** : 关联的值，也就是set方法传入的值给属性去保存。 
参数四：**`objc_AssociationPolicy policy`** : 策略，属性以什么形式保存。 有以下几种

```
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,  // 指定一个弱引用相关联的对象
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, // 指定相关对象的强引用，非原子性
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,  // 指定相关的对象被复制，非原子性
    OBJC_ASSOCIATION_RETAIN = 01401,  // 指定相关对象的强引用，原子性
    OBJC_ASSOCIATION_COPY = 01403     // 指定相关的对象被复制，原子性   
};
```

key值只要是一个指针即可，我们可以传入@selector(name)

2.  获得属性

```
objc_getAssociatedObject(id object, const void *key);
```

参数一：**`id object`** : 获取哪个对象里面的关联的属性。 
参数二：**`void * == id key`** : 什么属性，与**`objc_setAssociatedObject`**中的key相对应，即通过key值取出value。

3.  移除所有关联对象

```
- (void)removeAssociatedObjects
{
    // 移除所有关联对象
    objc_removeAssociatedObjects(self);
}
```

此时已经成功给NSObject添加name属性，并且NSObject对象可以通过点语法为属性赋值。

```
NSObject *objc = [[NSObject alloc]init];
objc.name = @"xx_cc";
NSLog(@"%@",objc.name);
```

可以看出关联对象的使用非常简单，接下来我们来探寻关联对象的底层原理

## 关联对象实现原理

实现关联对象技术的核心对象有

1.  `AssociationsManager`
2. ` AssociationsHashMap`
3.  `ObjectAssociationMap`
4.  `ObjcAssociation`
    其中Map同我们平时使用的字典类似。通过key-value一一对应存值。

对关联对象技术的核心对象有了一个大概的意识，我们通过源码来探寻这些对象的存在形式以及其作用

### objc_setAssociatedObject函数

来到runtime源码，首先找到`objc_setAssociatedObject`函数，看一下其实现

![image.png](https://upload-images.jianshu.io/upload_images/1401554-98d5dea309f7e403.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


我们看到其实内部调用的是`_object_set_associative_reference`函数，我们来到`_object_set_associative_reference`函数中

### _object_set_associative_reference函数

![image.png](https://upload-images.jianshu.io/upload_images/1401554-4d992d8f4006b7a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


`_object_set_associative_reference`函数内部我们可以全部找到我们上面说过的实现关联对象技术的核心对象。接下来我们来一个一个看其内部实现原理探寻他们之间的关系。

### AssociationsManager

通过`AssociationsManager`内部源码发现，`AssociationsManager`内部有一个`AssociationsHashMap`对象。

![image.png](https://upload-images.jianshu.io/upload_images/1401554-d95c7abb1d8dc8dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### AssociationsHashMap

我们来看一下`AssociationsHashMap`内部的源码。

![image.png](https://upload-images.jianshu.io/upload_images/1401554-3b68a77fa56d5903.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


通过`AssociationsHashMap`内部源码我们发现`AssociationsHashMap`继承自`unordered_map`首先来看一下`unordered_map`内的源码

![image.png](https://upload-images.jianshu.io/upload_images/1401554-b22d49da01900a44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


从`unordered_map`源码中我们可以看出`_Key`和`_Tp`也就是前两个参数对应着`map`中的`Key`和`Value`，那么对照上面`AssociationsHashMap`内源码发现`_Key`中传入的是`disguised_ptr_t`，`_Tp`中传入的值则为`ObjectAssociationMap*`。

紧接着我们来到`ObjectAssociationMap`中，上图中**`ObjectAssociationMap`**已经标记出，我们发现`ObjectAssociationMap`中同样以`key`、`Value`的方式存储着**`ObjcAssociation`**。

接着我们来到`ObjcAssociation`中

![image.png](https://upload-images.jianshu.io/upload_images/1401554-441ba04f39200fa6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们发现ObjcAssociation存储着`_policy`、`_value`，而这两个值我们可以发现正是我们调用**`objc_setAssociatedObject`**函数传入的值，也就是说我们在调用**`objc_setAssociatedObject`**函数中传入的`value`和`policy`这两个值最终是存储在**`ObjcAssociation`**中的。

现在我们已经对**AssociationsManager、 AssociationsHashMap、 ObjectAssociationMap、ObjcAssociation**四个对象之间的关系有了简单的认识，那么接下来我们来细读源码，看一下objc_setAssociatedObject函数中传入的四个参数分别放在哪个对象中充当什么作用。

### 重新回到_object_set_associative_reference函数实现中

![image.png](https://upload-images.jianshu.io/upload_images/1401554-75833af63263ba7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

细读上述源码我们可以发现，首先根据我们传入的value经过acquireValue函数处理获取new_value。acquireValue函数内部其实是通过对策略的判断返回不同的值

![image.png](https://upload-images.jianshu.io/upload_images/1401554-1cb952c63e376947.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



之后创建`AssociationsManager` 
manager;以及拿到manager内部的AssociationsHashMap即**associations**。 
之后我们看到了我们传入的第一个参数`object`object经过`DISGUISE`函数被转化为了`disguised_ptr_t`类型的**`disguised_object`**。

![image.png](https://upload-images.jianshu.io/upload_images/1401554-052dbde5d732bb48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


`DISGUISE`函数其实仅仅对object做了位运算

**之后我们看到被处理成new_value的value，同policy被存入了ObjcAssociation中。 而ObjcAssociation对应我们传入的key被存入了ObjectAssociationMap中。 disguised_object和ObjectAssociationMap则以key-value的形式对应存储在associations中也就是AssociationsHashMap中。**

![image.png](https://upload-images.jianshu.io/upload_images/1401554-dd7dc891f76c7c83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如果我们value设置为nil的话那么会执行下面的代码

![image.png](https://upload-images.jianshu.io/upload_images/1401554-9f9ae4de859928c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


从上述代码中可以看出，如果我们设置value为nil时，就会将关联对象从`ObjectAssociationMap`中移除。

最后我们通过一张图可以很清晰的理清楚其中的关系

![image.png](https://upload-images.jianshu.io/upload_images/1401554-72841282aa97f126.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


**通过上图我们可以总结为：一个实例对象就对应一个ObjectAssociationMap，而ObjectAssociationMap中存储着多个此实例对象的关联对象的key以及ObjcAssociation，为ObjcAssociation中存储着关联对象的value和policy策略。**

**由此我们可以知道关联对象并不是放在了原来的对象里面，而是自己维护了一个全局的map用来存放每一个对象及其对应关联属性表格。**

### objc_getAssociatedObject函数

`objc_getAssociatedObject`内部调用的是`_object_get_associative_reference`

![image.png](https://upload-images.jianshu.io/upload_images/1401554-f3133a37783ed74b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### _object_get_associative_reference函数

![image.png](https://upload-images.jianshu.io/upload_images/1401554-2a844b423c236135.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


从_object_get_associative_reference函数内部可以看出，向set方法中那样，反向将value一层一层取出最后return出去。

### objc_removeAssociatedObjects函数

`objc_removeAssociatedObjects`用来删除所有的关联对象，`objc_removeAssociatedObjects`函数内部调用的是`_object_remove_assocations`函数

![image.png](https://upload-images.jianshu.io/upload_images/1401554-ca9a536f7626a108.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### _object_remove_assocations函数

![image.png](https://upload-images.jianshu.io/upload_images/1401554-41a05772fe0ca5c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


上述源码可以看出`_object_remove_assocations`函数将object对象向对应的所有关联对象全部删除。

### 总结：

**关联对象并不是存储在被关联对象本身内存中，而是存储在全局的统一的一个AssociationsManager中，如果设置关联对象为nil，就相当于是移除关联对象。**

此时我们我们在回过头来看objc_AssociationPolicy policy 参数: 属性以什么形式保存的策略。

```
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,  // 指定一个弱引用相关联的对象
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, // 指定相关对象的强引用，非原子性
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,  // 指定相关的对象被复制，非原子性
    OBJC_ASSOCIATION_RETAIN = 01401,  // 指定相关对象的强引用，原子性
    OBJC_ASSOCIATION_COPY = 01403     // 指定相关的对象被复制，原子性   
};
```

我们会发现其中只有RETAIN和COPY而为什么没有weak呢？ 总过上面对源码的分析我们知道，object经过DISGUISE函数被转化为了disguised_ptr_t类型的**disguised_object**。

```
disguised_ptr_t disguised_object = DISGUISE(object);
复制代码
```

而同时我们知道，`weak修饰的属性，当没有拥有对象之后就会被销毁，并且指针置位nil`，那么在对象销毁之后，虽然在map中既然存在值object对应的`AssociationsHashMap`，但是因为object地址已经被置位nil，会造成坏地址访问而无法根据object对象的地址转化为`disguised_object`了。

参考文章：https://juejin.im/post/5af86b276fb9a07aa34a59e6
