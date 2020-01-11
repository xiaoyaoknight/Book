# performSelector传递两个以上参数


当我们有方法名和参数列表，想要动态地给对象发送消息，可用通过反射函数机制来实现，有两种常用的做法：

####一、performSelector

常用的方法有这三个，其中aSelector可以通过 NSSelectorFromString 方法拿到
**但是 performSelector 的缺点是最多只支持传递两个参数**

####二、NSInvocation

```
// 测试反射函数
- (void)printWithString:(NSString *)string withNum:(NSNumber *)number withArray:(NSArray *)array {
    NSLog(@"%@, %@, %@", string, number, array[1]);
}
- (void)test {
    NSString *str = @"哈哈哈";
    NSNumber *num = @20;
    NSArray *arr = @[@"ABC", @"DEF"]; // [self printWithString:str withNum:num withArray:arr];
    SEL sel = NSSelectorFromString(@"printWithString:withNum:withArray:");
    NSArray *objs = [NSArray arrayWithObjects:str, num, arr, nil];
    [self performSelector:sel withObjects:objs];
} 
- (id)performSelector:(SEL)selector withObjects:(NSArray *)objects { // 方法签名(方法的描述)
    NSMethodSignature *signature = [[self class] instanceMethodSignatureForSelector:selector]; 
   if (signature == nil) { //可以抛出异常也可以不操作。
   } 
    // NSInvocation : 利用一个NSInvocation对象包装一次方法调用（方法调用者、方法名、方法参数、方法返回值）
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:signature];
    invocation.target = self;
    invocation.selector = selector; // 设置参数
    NSInteger paramsCount = signature.numberOfArguments - 2; // 除self、_cmd以外的参数个数
    paramsCount = MIN(paramsCount, objects.count); 
   for (NSInteger i = 0; i < paramsCount; i++) { 
        id object = objects[i]; 
        if ([object isKindOfClass:[NSNull class]]) continue;
        [invocation setArgument:&object atIndex:i + 2];
    } // 调用方法
    [invocation invoke]; // 获取返回值
    id returnValue = nil; if (signature.methodReturnLength) { // 有返回值类型，才去获得返回值
        [invocation getReturnValue:&returnValue];
    } return returnValue;
}

```

####三、objc_msgSend

objc_msgSend的写法要复杂一点，具体可以参看[这篇文章](http://www.jianshu.com/p/efeb33712445)，讲的很清楚
但是有个缺点是，需要指定好传递参数的类型，是不是可以直接都用id呢？

经测试id可用
```
// objc_msgSend
SEL sel = NSSelectorFromString(@"printWithString:withNum:withArray:");
((void (*) (id, SEL, NSString *, NSNumber *, NSArray *)) objc_msgSend) (self, sel, str, num, arr);
```

[](https://www.jianshu.com/p/4118793c88df)
 [iOS 反射函数: performSelector, NSInvocation, objc_msgSend](https://www.cnblogs.com/mobilefeng/p/5720479.html)