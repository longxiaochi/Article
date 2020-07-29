## 分类设置关联值
分类可以定义属性， 但是不能定义成员变量(这是有分类的结构决定的)。
分类的结构如下：
```c
struct _category_t {
	const char *name;  // 类名称
	struct _class_t *cls;
	const struct _method_list_t *instance_methods; // 对象方法列表
	const struct _method_list_t *class_methods;    // 类方法列表
	const struct _protocol_list_t *protocols;      // 协议列表
	const struct _prop_list_t *properties;         // 属性列表
};
```
由上，分类的结构体中没有保存成员变量的地方。

要想在分类中间接实现类似成员变量的效果，可以通过关联值来实现。如下：

分类头文件.h
```swift
@interface Person (Test)

@property (nonatomic, strong) NSString *name;

- (void)setName:(NSString *)name;
- (NSString *)name;

@end
```

分类实现.m：
```swift
#import "Person+Test.h"
#import <objc/runtime.h>

@implementation Person (Test)

const void * LCKey = &LCKey;


- (void)setName:(NSString *)name {
    // 间接实现类似于成员变量的感觉。
    objc_setAssociatedObject(self, @selector(name), name, OBJC_ASSOCIATION_COPY_NONATOMIC); 
}

- (NSString *)name {
    // _cmd 就相当于@selector(name), 当前方法的选择器
    return objc_getAssociatedObject(self, _cmd);   
}
@end

```

关联的策略：
```c
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,                // 相当于 atomic assign
    OBJC_ASSOCIATION_RETAIN_NONATOMIC =  1,     // 相当于 nonatomic strong                                     
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,        // 相当于 nonatomic copy                          
    OBJC_ASSOCIATION_RETAIN = 01401,            // 相当于 atomic strong         
    OBJC_ASSOCIATION_COPY = 01403               // 相当于 atomic copy              
};
```

底层的实现：

- AssociationsHashMap             
    - 对象1的地址                  
        - ObjectAssociationMap：
            - const void *   //设定的key
            - ObjcAssociation 
                - _policy
                - value 
    - 对象2的地址
        - ObjectAssociationMap：
            - const void *
            - ObjcAssociation
                - _policy
                - value

存储结构如下：
```swift
// 底层的实现: Person 的地址作为key -> {
//{
//    person1地址: {
//        key1: {协议策略，value},
//        key2: {协议策略，value},
//        ....
//    },
//    person2地址: {
//        key1: {协议策略，value},
//        key2: {协议策略，value},
//        ....
//    },
//    ...
//}
```

先要实现同样的效果，不使用关联值的话，可以自定义一个全局的可变字典来实现。