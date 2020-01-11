# 模拟实现通知中心 iOS
**一、概述**

系统的通知中心NSNotificationCenter相信大家都比较熟悉，也就是一种观察者模式，当我关心某件事的时候我就注册这个通知，在收到通知时做相应的处理，是一种多对多机制，非常方便的实现多个对象的联动操作，那通知中心是怎么实现的呢？

第一反应可能用一个单例维护一个数组，注册的时候将当前对象加到数组中，当postNotification时去数组中遍历然后调用相应方法，确实可以，但是也许会遇到一个问题，就是对象加入数组后，引用计数会加一，拿控制器来说就是当我们pop的时候将不能销毁这个控制器，自然不会走dealloc方法

我们将用三种形式讨论通知中心的可行性，至于有什么作用，可能更多的是一种思想上的借鉴吧。gitHub地址：[Demo](https://github.com/xiaoyaoknight/NotificationCenter-Demo.git)

**二、实现原理**

至于哪三种方式呢，思路其实都是一样的，通过单例维护注册的对象集合，发出通知的时候去遍历并调用相应的方法，这里我们采用三种方式去维护这些对象集合：

1、通过数组形式

2、通过OC模拟链表形式

3、通过C语言链表形式

接下来就是实现代码，首先看一下.h中的方法：

```
NS_ASSUME_NONNULL_BEGIN

@interface ZLNotificationCenter :NSObject

+ (ZLNotificationCenter *)defaultCenter;

- (void)addObserver:(id)observer selector:(SEL)aSelector name:(nullableNSString*)aName object:(nullableid)anObject;

- (void)postNotification:(ZTNotification*)notification;

- (void)postNotificationName:(NSString*)aName object:(nullableid)anObject;

- (void)postNotificationName:(NSString*)aName object:(nullableid)anObject userInfo:(nullableNSDictionary*)aUserInfo;

- (void)removeObserver:(id)observer;

- (void)removeObserver:(id)observer name:(nullableNSString*)aName object:(nullableid)anObject;

- (id)addObserverForName:(nullableNSString*)name object:(nullableid)obj queue:(nullableNSOperationQueue*)queue usingBlock:(void(^)(ZTNotification*note))blockNS_AVAILABLE(10_6,4_0);

@end

NS_ASSUME_NONNULL_END
```

可以说完全照搬系统NSNotificationCenter中的方法，本来就是要实现他嘛。
而要想达到通知的目的，有三个属性是必不可少的:

- 注册的对象observer、
- 要执行的方法selector
- 注册的通知名notificationName

由此我们创建了一个保存这些信息的对象ZLObserverModel，看下有哪些属性：

```
typedefvoid(^OperationBlock)(ZTNotification*notification);

@interfaceZTObserverModel :NSObject

@property (nonatomic, weak) id observer;

@property (nonatomic, assign) SEL selector;

@property(nonatomic, copy) NSString*notificationName;

@property (nonatomic, weak) id object;

@property (nonatomic, strong) NSOperationQueue *operationQueue;

@property (nonatomic, copy) OperationBlock block;
```

前四个是注册时的参数，最后两个意思是如果对象指定了队列则在队列中执行block。同时定义这个对象也是为了解决强引用加一不释放的问题，我们并没有直接将对象加入数组或链表中，而是使用自定义的一个对象，其中observer使用的是weak，这就避免了强引用加一问题，不释放问题就解决了。

接下来看下三种方式的具体代码：

###第一种数组：注册时就是把对象加到数组中，接收到通知的时候去数组中遍历

```
//注册方法

- (void)addObserver:(id)observer selector:(SEL)aSelector name:(NSString*)aName object:(id)anObject {

    ZTObserverModel *observerModel = [[ZTObserverModel alloc] init];

    observerModel.observer = observer;

    observerModel.selector = aSelector;

    observerModel.notificationName = aName;

    observerModel.object = anObject;

    [self.observers addObject:observerModel]; //采用数组形式

}

- (void)postNotification:(ZTNotification*)notification {

    //采用数组形式

    [self.observers enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {

        ZTObserverModel *observerModel = obj;

        id observer = observerModel.observer;

        SEL selector = observerModel.selector;

        if (![notification.name isEqualToString:observerModel.notificationName]) {

            return;

        }

        if (!observerModel.operationQueue) {

#pragma clang diagnostic push

#pragma clang diagnostic ignored "-Warc-performSelector-leaks"

        [observer performSelector:selector withObject:notification];

#pragma clang diagnostic pop

        } else {

            NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{

                observerModel.block(notification);

            }];

            NSOperationQueue *operationQueue = observerModel.operationQueue;

            [operationQueue addOperation:operation];

        }

    }];

}
```

###第二种OC模拟链表：首先我们需要先定义一个节点和一个链表

节点：


```
@interface ZTNode : NSObject

@property id data; //数据域

@property ZTNode *next; //指针域，指向下个节点

- (id)initWithData:(id)data;//结点初始化

@end
```

链表：


```
@interface ZTList : NSObject

@property ZTNode *head; //定义链表的头

+ (void)insert:(id)data linkList:(ZTList*)linkList; //插入结点

+ (id)listWithData:(id)data; //工厂方法

@end
```


**看下链表中的实现：插入方法用的是尾插法**


```
@implementation ZTList

- (instancetype)initWithData:(id)data {

    self= [super init];

    if(self) {

        self.head = [[ZTNode alloc] init];

        self.head.data = data;

        self.head.next = nil;

    }

    return self;

}

+ (id)listWithData:(id)data {

    ZTList *firstNode = [[ZTList alloc] initWithData:data];

    return firstNode;

}

+ (void)insert:(id)data linkList:(ZTList*)linkList {

    ZTNode *p = linkList.head;

    while(p.next) {

        p = p.next;

    }

    ZTNode *node = [[ZTNode alloc] init];

    node.data = data;

    node.next = p.next;

    p.next = node;

}

@end
```


有了链表我们存储和查找就比较简单了，存储时只需将加入数组的代码改成

`[ZTList insert:observerModel linkList:_ztList]; `

就行了，查找时代码如下：

```
//采用oc模拟链表

    ZTNode *p = _ztList.head;

    while (p) {

        p = p.next;

        ZTObserverModel *observerModel = p.data;

        id observer = observerModel.observer;

        SEL selector = observerModel.selector;

        if (![notification.name isEqualToString:observerModel.notificationName]) {

            return;

        }

        if (!observerModel.operationQueue) {

#pragma clang diagnostic push

#pragma clang diagnostic ignored "-Warc-performSelector-leaks"

            [observer performSelector:selector withObject:notification];

#pragma clang diagnostic pop

        } else {

            NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{

                observerModel.block(notification);

            }];

            NSOperationQueue *operationQueue = observerModel.operationQueue;

            [operationQueue addOperation:operation];

        }

    }
```


###第三种C链表：C语言中我们使用结构体定义链表，看下.h文件：**

```
typedef struct Node {

    void *data;

    struct Node *next;

} Node;

typedef struct Node *LinkList;

@interface LinkListC :NSObject

+ (LinkList)insert:(id)data linkList:(LinkList)linkList;//插入结点

+ (LinkList)listInit;

@end
```

**.m中主要看下insert中的数据转换**

```
#import "LinkListC.h"

@implementation LinkListC

//单链表的初始化

LinkList LinkedListInit() {

    Node*L;

    L = (Node*)malloc(sizeof(Node));  //申请结点空间

    if(L == NULL) {//判断是否有足够的内存空间

        printf("申请内存空间失败\n");

    }

    L->next = NULL;                  //将next设置为NULL,初始长度为0的单链表

    return L;

}

- (instancetype)initWithData:(id)data {

    self= [super init];

    if(self) {

        LinkedListInit();

    }

    return self;

}

+ (LinkList)listInit {

    LinkList list = LinkedListInit();

    return list;

}

+ (LinkList)insert:(id)data linkList:(LinkList)linkList {

    Node*head;

    head = linkList;

    Node*p;

    //插入的结点为p

    p = (Node*)malloc(sizeof(Node));

    p->data = (__bridge_retained void *)data;

    p->next = head->next;

    head->next = p;

    return linkList;

}

@end
```

定义过程中void *data标示C的任意类型，

`p->data = (__bridge_retained void *)data`

这里通过桥接将OC数据转成C数据，插入方法这里使用的是头插法。
接下来注册的时候

`[LinkListC insert:observerModel linkList:_linkListC]`

就会将对象插入到了链表中，这里接受到通知时还是要注意C跟OC数据转换的问题，看下代码：

```
- (void)postNotification:(ZTNotification*)notification {

    //采用C链表

    Node*p;

    p = _linkListC;

    while(p->next) {

        p = p->next;

        ZTObserverModel *observerModel = (__bridge id)p->data;

        id observer = observerModel.observer;

        SEL selector = observerModel.selector;

        if(![notification.name isEqualToString:observerModel.notificationName]) {

            return;

        }

        if(!observerModel.operationQueue) {

#pragma clang diagnostic push

#pragma clang diagnostic ignored"-Warc-performSelector-leaks"

            [observerperformSelector:selectorwithObject:notification];

#pragma clang diagnostic pop

        }else{

            NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{

                observerModel.block(notification);

            }];

            NSOperationQueue*operationQueue = observerModel.operationQueue;

            [operationQueue addOperation:operation];

        }

    }

}
```
这样我们三种方式就说完了，话说简书自动取消空格好烦人，整的格式都不对了。

**三、使用方法**

注册：

```
ZTNotificationCenter *notificationCenter = [ZTNotificationCenter defaultCenter];  
[notificationCenter addObserver:selfselector:@selector(test1:)name:@"TextFieldValueChanged"object:nil];
```

发通知：

```
ZTNotification *notification = [ZTNotification notificationWithName:@"TextFieldValueChanged" object:self.textField];

 ZTNotificationCenter *notificationCenter = [ZTNotificationCenter defaultCenter];

 [notificationCenter postNotification:notification];
```

**四、补充**

在以上代码中会用到一个ZTNotification对象，这个系统的也是有的，是通知的对象，.h中的方法也是从系统的NSNotification中完全拿过来的。代码可以看下：

```
NS_ASSUME_NONNULL_BEGIN

/****************    Notifications    ****************/

@class NSString, NSDictionary, NSOperationQueue;

@interfaceZTNotification :NSObject

@property (readonly, copy) NSString *name;

@property (nullable, readonly, retain) id object;

@property (nullable, readonly, copy) NSDictionary *userInfo;

- (instancetype)initWithName:(NSString*)name object:(nullableid)object userInfo:(nullableNSDictionary*)userInfoNS_AVAILABLE(10_6,4_0) ;

- (nullableinstancetype)initWithCoder:(NSCoder*)aDecoder ;

+ (instancetype)notificationWithName:(NSString*)aName object:(nullableid)anObject;

+ (instancetype)notificationWithName:(NSString*)aName object:(nullableid)anObject userInfo:(nullableNSDictionary*)aUserInfo;

@end

NS_ASSUME_NONNULL_END
```
