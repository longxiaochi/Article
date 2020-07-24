## 属性
Swift 中跟实列相关的数据可以分为2大类
- 存储属性(Stored Property)
  - 类似于成员变量的概念
  - 存储在实列的内存中
  - 结构体、类可以定义存储属性
  - 枚举不可以定义存储属性。(枚举的内存只存储枚举成员和关联值)

- 计算属性(Computed Property)
  - 本质就是方法(函数)
  - 不占用实例的内存
  - 枚举、结构体、类都可以定义计算属性

```swift
struct Circle {
    // 存储属性
    var radius: Double
    
    // 计算属性
    var diameter: Double {
        set {
            radius = newValue / 2
        }
        get {
            radius * 2
        }
    }
}

// 打印所占内存
print(MemoryLayout<Circle>.stride)   

var circle = Circle(radius: 12)
print(circle.radius)    // 12
print(circle.diameter)  // 24

circle.diameter = 10
print(circle.radius)    // 5


结果： 8
```
计算属性使用场景： 当存在逻辑关联时可以使用计算属性。如上方的diameter 和 radius

### 存储属性
对于存储属性，Swift官方有一个明确的规定
- 在创建类和结构体的实例时，必须为所有的存储属性提供一个合适的初始值。
    - 可以在初始化器内为存储属性赋值一个初始值。
    - 也可以在定义存储属性时赋值一个初始值。
```swift
struct Circle {
    // 存储属性
    var radius: Double = 10  // 方式一
    
    // 计算属性
    var diameter: Double {
        set {
            radius = newValue / 2
        }
        get {
            radius * 2
        }
    }
    
    init(radius: Double) {
        self.radius = radius  // 方式二
    }
}
```
方式一和方式二本质其实是一样的。 都是在init方法内进行初始化。

### 计算属性
set 传入的默认值名称为newValue，也可以自定义：
```swift
struct Circle {
    // 存储属性
    var radius: Double
    
    // 计算属性
    var diameter: Double {
        set(newDiameter) {   // 自定义输入参数
            radius = newDiameter / 2
        }
        
        get {
            radius * 2
        }
    }
}
```
定义计算属性只能使用var，不能使用let
- let代表常量，值是一成不变的。
- 计算属性的值随时可能发生变化，所以只能使用var。参考上面的diameter，会随着radius发生改变。


#### 只读计算属性
只有get方法，没有set 方法
```swift
struct Circle {
    // 存储属性
    var radius: Double
    
    // 计算属性
    var diameter: Double {
        get {
            radius * 2
        }
    }
}

可以简化为：
struct Circle {
    // 存储属性
    var radius: Double
    
    // 计算属性
    var diameter: Double { radius * 2 }
}
```
注意： 但是不能只有set方法，没有get方法。

### 枚举rawValue的原理
枚举rawValue的本质： 只读计算属性
```swift
enum TestEnum: Int {
    case test1 = 1, test2 = 2, test3 = 3, test4 = 4
   
    var rawValue: Int {
        switch self {
        case .test1:
            return 10
        case .test2:
            return 2
        case .test3:
            return 3
        case .test4:
            return 4
        }
    }
}

var ts = TestEnum.test1
print(ts.rawValue)  // 10
```

### 延迟存储属性
使用lazy关键词可以定义一个延迟存储属性，在第一次用到属性的时候才会被初始化。
```swift
class Car {
    init() {
        print("Car init")
    }
    
    func run() {
        print("Car run")
    }
}

class Person {
    lazy var car = Car()
    init() {
        print("Person init")
    }
    
    func goRun() {
        car.run()
        print("Go Run")
    }
}


var person = Person()
print("--------")
person.goRun()

打印结果：
Person init
--------
Car init
Car run
Go Run
```

- lazy 属性必须是var, 不能是let
- let 必须在实例的初始化方法完成之前就拥有值。
- 如果多条线程同时第一次访问lazy属性，不能保证属性只被初始化一次。
- 应用场景： 如图片的加载。

