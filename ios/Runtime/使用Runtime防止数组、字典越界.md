# 使用Runtime防止数组、字典越界


### 获取数组class
```
Method oldObjectAtIndex = class_getInstanceMethod(objc_getClass("__NSArrayI"), @selector(objectAtIndex:));
```

### 核心方法
```
#import "NSArray+ArrayBounds.h"
#import <objc/runtime.h>

@implementation NSArray (ArrayBounds)

+ (void)load {
    
    [super load];
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{  //方法交换只要一次就好
        Method oldObjectAtIndex = class_getInstanceMethod(objc_getClass("__NSArrayI"), @selector(objectAtIndex:));
        Method newObjectAtIndex = class_getInstanceMethod(objc_getClass("__NSArrayI"), @selector(change_objectAtIndex:));
        method_exchangeImplementations(oldObjectAtIndex, newObjectAtIndex);
    });
}

- (id)change_objectAtIndex:(NSUInteger)index{
    
    if (index > self.count - 1 || !self.count){
        return nil;
    }
    else{
        return [self change_objectAtIndex:index];
    }
}
@end
```

### 可变数组、字典、可变字典的崩溃问题等（思路和数组一致，获取类名的方法及对应的方法名不同而已）
[Runtime替换字典](https://www.jianshu.com/p/593b7479c8e6)
>可变字典的类名： __NSDictionaryM
     不可变字典的类名：__NSDictionaryI
     可变数组的类名：__NSArrayM
     不可变数组的类名：__NSArrayI


### 可以查看[Runtime-Demo](https://github.com/xiaoyaoknight/Runtime-Demo)

