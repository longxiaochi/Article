### KVO (Key Value Observer)
KVO 是OC用来监听对象属性的变化。

基本用法：
```swift
Person 类
@interface Person : NSObject

@property (nonatomic, strong) NSString *name;  

@end

@implementation Person

@end


ViewController 类

@interface ViewController ()

@property (nonatomic, strong) Person *person;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    self.person = [[Person alloc] init];
    self.person.name = @"longchi";
    
    NSKeyValueObservingOptions option = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
    [self.person addObserver:self forKeyPath:@"name" options:option context:nil];
}

// 点击屏幕时触发
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    self.person.name = @"xiaoyezhi";
}


- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"对象： %@, 的属性 %@ 发生了变化， 值为 %@", object, keyPath, change);
}

// 销毁时需要移除监听
- (void)dealloc {
    [self.person removeObserver:self forKeyPath:@"age" context:nil];
}

```
### 实现原理
```swift
- (void)viewDidLoad {
    [super viewDidLoad];

    self.person = [[Person alloc] init];
    self.person.name = @"longchi";
    
    self.person2 = [[Person alloc] init];
    self.person2.name = @"lalala";
    
     NSLog(@"添加监听前： person1_class: %@, person2_class: %@", object_getClassName(self.person), object_getClassName(self.person2));
    
    NSKeyValueObservingOptions option = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
    [self.person addObserver:self forKeyPath:@"name" options:option context:nil];
    
    NSLog(@"添加监听后： person1_mothod: %@, person2_method: %@", [self.person methodForSelector:@selector(setName:)], [self.person2 methodForSelector:@selector(setName:)]);
}

打印结果：
person添加监听前： Person
person添加监听后： NSKVONotifying_Person
```
由上可知，person在添加observer之后，isa 指向的类对象发生了变化，变为NSKVONotifying_Person， 这个是Rumtime 自动创建的。

### NSKVONotifying_Person 中setName方法实际是调用了Foundation的_NSSetXXXValueAndNodify 方法
证明：
```swift
- (void)viewDidLoad {
    [super viewDidLoad];

    self.person = [[Person alloc] init];
    self.person.name = @"longchi";
    
    self.person2 = [[Person alloc] init];
    self.person2.name = @"lalala";
    
   NSLog(@"添加监听前： person1_mothod: %p, person2_method: %p", [self.person methodForSelector:@selector(setName:)], [self.person2 methodForSelector:@selector(setName:)]);
    
    NSKeyValueObservingOptions option = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
    [self.person addObserver:self forKeyPath:@"name" options:option context:nil];
    
    NSLog(@"添加监听后： person1_mothod: %p, person2_method: %p", [self.person methodForSelector:@selector(setName:)], [self.person2 methodForSelector:@selector(setName:)]);
}

打印结果：
person添加observer前setName的实现地址： xxxx1
person添加observer后setName的实现地址： xxxx2
```
由上可知，person在添加observer 前后setName 方法的实现也不一样。断点在控制台通过lldb指令：p (IMP)方法实现地址  可以知道在添加observer之后，person 调用setName方法时，实际上是调用Foundation中的NSSetXXXValueAndNotify 方法。

通过逆向可查看： _NSSetXXXValueAndNotify 大概实现。
```swift
_NSSetXXXValueAndNotify {
    willChangeValueForKey:("age");
    父类的setter方法。
    didChangeValueForKey:("age");
}

- (void)didChangeValueForKey {
    // 通知观察者
    observer observerValueForKeyPath xxxxxx
}
```


### NSKVONotifying_Person 中的方法
```
- (void)setter{

}

- (void)dealloc() {
    // 收尾工作
}

- (BOOL)isKVOA {
    // 是否实现kvo
}

- (Class)class {
    [super class];
    // 对开发者屏蔽内部实现，当开发者调用 [person class] 依旧返回Person 而不是 NSKVONotifying_Person
}

```
打印类对象方法列表，代码实现
``` swift
- (void)printMethodsOfClass:(Class)cls {
    unsigned int count;
    Method *methods = class_copyMethodList(cls, &count);
    
    NSMutableString *methodNameStr = [[NSMutableString alloc] init];
    for (int i = 0; i < count; i++) {
        Method method = methods[i];
        NSString *methodName =  NSStringFromSelector(method_getName(method));
        [methodNameStr appendString:methodName];
        [methodNameStr appendString:@", "];
    }
    
    free(methods);
    
    NSLog(@"%@: %@", cls, methodNameStr);
}
```