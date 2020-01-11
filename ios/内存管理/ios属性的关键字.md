# ios属性的关键字


| attributes |    | 
| --------   | -----| 
| Attributes        | Attributes      |  
| atomic        |nonatomic     | 
| strong        | weak     | 
|readwrite	|readonly|
|getter	|setter|
|copy	|assign|
|retain|	unsafe_unretained|

####atomic
如果没有指明是nonatomic的，那么**默认**就是atomic。atomic的意思是数据的`读写是线程安全的`，这在多线程的时候非常有用，但是与nonatomic相比的话效率会低很多。所以为了提升效率，除非是在明确知道对象会在多线程中使用，否则尽量使用nonatomic关键字。

####nonatomic
nonatomic不能保证数据的原子性，也就是说在多线程访问的时候不能保证能够正确读取数据，甚至有可能发生崩溃。如果使用了nonatomic关键字，这时候如果存在多线程访问的话，需要自己来处理线程的安全问题。

####retain
ARC之前使用的关键字，ARC之后用strong代替。

####strong
这是一个ARC之后才有的属性，在使用ARC的时候，这是一个默认属性。与它对应的是weak属性。strong使类能够`拥有对象的所有权`，会增加对象的 `retain count` 以保证在类使用该对象的时候，对象没有被销毁。strong可能会导致`retain cycle`，要谨慎使用，`常使用在继承自NSObject的类和大部分自定义类`。
[官方文档](https://developer.apple.com/library/archive/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)介绍：
```
// The following declaration is a synonym for: @property(retain) MyClass *myObject;
@property(strong) MyClass *myObject;
```
表示了strong和retain是同义词。

####unsafe_unretained
ARC之前使用的关键字，类似于weak，`但与weak不同的是在对象被销毁后，指针不会被置为空，所以有可能会造成崩溃`
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1401554-3fda2270a9972189.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


#### weak （weak 对象释放的时候，怎么置为nil的呢？请阅读[iOS 底层解析weak的实现原理](http://www.cocoachina.com/ios/20170328/18962.html)）
只是将`指针指向该对象`，但是并`没有对象的所有权`，在对象被释放的时候，weak指针会被置为nil，这保证了指针的安全， 在`OC里是可以给nil对象发送消息而不会崩溃的`，但是如果`给一个被释放的对象发送消息就会崩溃了`。weak不会增加对象的 `retain count`。

####assign
`简单数据类型`（int，long，double， float，Bool）默认就是assign的，`不改变对象的retainCount`。最好是只在简单数据类型时使用，对于对象类型，最好不要使用 

#### readwrite
所有的属性默认都是readwrite的，使用readonly关键字可以覆盖该默认值。

#### readonly
表示对象是只读的。当我们不希望外面的类可以修改该属性的时候，可以在头文件中将属性定位readonly，但是在实现文件中，我们可以将它声明为readwrite，这样就只能在类内部才能修改这个属性了。

#### getter    
指定get方法, 并需要实现这个方法. 必须返回与声明类型相同的变量,没有参数.

####setter    
指定 set 方法, 并需要实现这个方法. 带一个与声明类型相同的参数, 没有返回值(返回nil)当声明为 readonly 的时候, 不能指定 set 方法.
`默认情况下，系统会生成一个与属性同名的getter、setter 方法，`

####copy  [copy还是strong？用copy还是mutablecopy](https://www.jianshu.com/p/b3ef37d3e03c)
与strong不同的是，copy`不会增加对象的 retain count`，
而是会`重新复制`一份对象，然后将`指针`指向`新复制`的对象。
使用copy关键字的属性的类型必须遵循`NSCopying`协议。
含有`可深拷贝`的`NSCopy`子类的类，如`NSArray`,` NSSet`, `NSDictionary`, `NSData`, `NSCharacterSet`, `NSIndexSet`, `NSString` 等一般都用copy来修饰. 这是因为`父类的指针可以指向子类对象`, 使用copy的目的就是为了`对象本身不受外界影响`, 无论传给我一个可变还是不可变的对象, 我本身持有的都是一个不可变对象.

>假如有一个属性是NSString类型的，但是我们却将一个NSMutableString赋值给了它（这是合法的），如果我们使用strong关键字，那么现在这个对象是一个NSMutableString的对象，如果在别的地方修改了这个对象的值，那么该属性也跟着变了，这可能会带来意想不到的后果。但如果我们使用copy关键字的话，就不会存在这个问题了，因为它会拷贝一份NSMutableString的值，这时属性依然是 immutable 的，即使NSMutableString的对象修改了也不会影响属性的值。

[Block为什么使用copy修饰](https://www.jianshu.com/p/64125b5617a4)
>DemoBlock内部没有调用外部变量时存放在全局区(ARC和MRC下均是)
DemoBlock2使用了外部变量,这种情况也正式我们平时所常用的方式,Block的内存地址显示在栈区,栈区的特点就是创建的对象随时可能被销毁,一旦被销毁后续再次调用空对象就可能会造成程序崩溃,在对block进行copy后,block存放在堆区.所以在使用Block属性时使用Copy修饰,而在ARC模式下,系统也会默认对Block进行copy操作



#### @synthesis
`告诉编译器自动生成属性的 getter 和 setter 方法，并且为属性绑定一个成员变量`，也就是说将属性的访问器方法作用于绑定的成员变量。正常情况下可以不使用@synthesis关键字，编译器也会默认生成访问器方法，在一些特殊情况下，需要明确使用@synthesis关键字才能生成访问器方法，包括以下情况：

- 绑定非默认的实例变量。
默认情况下，系统会生成一个_ 的成员变量与属性绑定在一起，如果不想使用系统生成的成员变量的话，可以通过@synthesis name = customName来绑定一个自定义的成员变量名。

- 协议中定义的属性。
当类实现的协议中包含属性时，必须在类中明确使用@synthesis才能生成访问器方法

- 重写访问器方法。
在重写 getter 和 setter 的时候需要用与属性相关的成员变量，由于重写了 getter 和 setter ，系统不会在默认合成这两个方法，也不会默认生成一个带下划线的成员变量，这时需要使用@synthesis关键字来定一个与属性对应的成员变量。

- 类别（category）中定义的所有属性

- 重载的属性
重写了 setter 和 getter 或者只读属性的getter时

####@dynamic
目的是告诉编译器访问器方法没有本类中实现，而是在别的地方（比如父类）中实现了，编译器就不会再发出警告了。主要是用在CoreData的NSManagedObject子类中。