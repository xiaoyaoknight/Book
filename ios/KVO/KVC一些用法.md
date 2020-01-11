# KVC一些用法

##setObject forKey 和 setValue forKey
`setObject:ForKey:` 是`NSMutableDictionary`特有的；
`setValue:ForKey:`是`KVC`的主要方法；

```
// setObject:ForKey:中object对象不能为nil,不然会报错,key的参数只要是对象就可以，并局限于 NSString；
[dict setObject:nil forKey:@"key"];  //崩溃！

// setValue:ForKey:中Value值可以为nil，此时会自动调用removeObject:forKey:方法；key的参数只能是NSString类型；
[dict setValue:nil forKey:@"key"];  // 不会蹦，自动调用removeObject:forKey方法
```
nil与null是不同的,[NSNull null]表示是一个空的对象,并不是nil；
·setValue:ForKey·:是在NSObject对象中创建的,即所有的对象都有这个方法，可以用于任何类(方法调用者是对象的时候);

**总结：**
所以使用的时候为了安全尽量使用`setValue`,还可以对setObject中的object使用宏定义替换，如果值为nil则使用如下宏定义替换成@""

```
//NSString  nil --> @""
#define NilString(a)  (((a)==nil)?@"":(a))
[dict setObject:NilString(nil) forKey:@"key"]; 
```

##objectForKey:和valueForKey:取值区别与联系
NSDictioary取值的时候有两个方法,`objectForKey:`和`valueForKey:`(建议用objectForKey:)
1.若key值不是以@符合开头, 两者是相同的；
2.若key值是以@开头, 例如：@“key”,则valueForKey:会去掉@,然后用剩下的部分执行[super valueForKey];
3.例子：
```
Person *person = [Person alloc] init];
person.name = @"Leo;
```
则通过：`[person valueForKey:@“name”]；`取出的值是Leo。这是KVC的方法。
4.`valueForKey：`取值是找和指定key同名的`property accessor(属性访问)`没有找到的时候执行`valueForUndefinedKey:`方法，而`valueForUndefinedKey:`方法默认是抛出`crash`异常；

**两者都是键值对应，区别是valueforkey 只允许使用NSString类型，objectforkey可以是任意类型.**

##valueForKeyPath 
```
NSArray *array = @[@"name", @"w", @"aa", @"jimsa"];
NSLog(@"%@", [array valueForKeyPath:@"uppercaseString"]);
// 输出(NAME,W,AA,JIMSA)
相当于数组中的每个成员执行了`uppercaseString`
```
```
//对NSNumber数组快速计算数组求和、平均数、最大值、最小值
NSArray *array = @[@1, @2, @3, @4, @10]; 
NSNumber *sum = [array valueForKeyPath:@"@sum.self"]; 
NSNumber *avg = [array valueForKeyPath:@"@avg.self"]; 
NSNumber *max = [array valueForKeyPath:@"@max.self"]; 
NSNumber *min = [array valueForKeyPath:@"@min.self"];
```

```
// 指定输出类型 
NSNumber *sum = [array valueForKeyPath:@"@sum.floatValue"]; 
```
```
// 剔除重复数据
NSArray *array = @[@"name", @"w", @"aa", @"jimsa", @"aa"];
NSLog(@"%@", [array valueForKeyPath:@"@distinctUnionOfObjects.self"]);
```
 
```
//直接改变对象隐藏属性的值
[searchField setValue:[UIColor whiteColor] forKeyPath:@"_placeholderLabel.textColor"];
//比起重写 - (void)drawPlaceholderInRect:(CGRect)rect; 要方便太多！
```