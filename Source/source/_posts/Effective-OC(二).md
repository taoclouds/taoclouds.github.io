title: 【读书笔记】Effective Objective-C 2.0(二)
date: 2018/09/10
categories:
- iOS 开发
tags:
- Objective-C
- 编程
---

# 理解属性的概念
在 Objective-C 等面向对象编程语言中，对象就是基本的构造单元，通过对象存储并传递数据。在对象之间传递数据并执行任务的过程叫做消息传递。支持应用程序运行的一套机制成为“运行时机制(Objective-C runtime)”。它提供了一些使得对象间传递消息的重要函数，包括手动创建类实例的全部逻辑。

`Objective-C`使用属性来存储并封装对象中的数据。一个 `Objective-C`对象会把需要的数据保存为各种实例变量。实例变量通过存取方法（即 getter 和 setter ）来获取和设置对象的实例变量值。而在`Objective-C`2.0中，这个概念已经由属性来帮助开发者完成，编译器会自动添加与属性相关的存取方法。开发者可以使用点语法来访问某个对象的属性。下面通过例子介绍属性是怎么样一个概念：
现在有一个 `person`类，它有一些属性：名字，年龄，住址等，可以在类的 `public`中声明一些实例变量：
```
@interface WTPerson : NSObject {
    @public
        NSString *_name;
        NSString *_address;
        NSInteger _age;
    @private
        NSString *_privateStr;
}
```
这种写法和我们平常在项目的写法很不一样，它自己定义了实例变量的作用域是`public`还是`private`。在 `OC` 中，访问实例变量是通过寻找该类在内存中的地址`addr`，然后根据实例变量定义的先后顺序， 在`addr` 的基础上再加上一个偏移量，这样就把实例变量给找到了，这个过程是在编译期间就已经确定了的。

所以如果我们添加了一个类的实例变量，那么就需要对其进行重新编译，否则在访问该类的实例变量时就会出错。为了避免这种情况，`Objective-C`采用的方法是将实例变量与类绑定起来，在书中的介绍是声明实例变量为一种特殊的变量交给类对象来保管，这样当类发生变化的时候，存储的实例变量相应地也会发生改变，保证能得到正确的实例变量。

另外一种解决办法就是间接地访问实例变量，通过 `setter`和`getter`方法来访问实例变量。其实最终还是要对实例变量进行处理，但是这种方式允许自己编写对应的存取方法，同时`Objective-C`也可以通过`@property`关键字自动生成对应的存取方法。
```
@interface Person : NSObject

@property NSString *name;
@property NSString *address;
@property NSInteger age;

@end

```
访问属性的方法可以使用点语法，类似`C 语言`访问结构体内的数据成员。编译器将点语法转化成对`setter`和`getter`方法的调用。使用`@property`声明的属性，编译器会自动为其添加存取方法，并且会通过在属性名前加下划线的方式在类中添加适当类型的实例变量，就像下面这样(`@synthesize`关键字指定实例变量的名字)：
```
@implementation Person
@synthesize name = _name;
@synthesize address = _address;
@synthesize age = _age;
@end
```
`@synthesize`关键字可以让你修改实例变量的名称。默认就是你声明的`property`的名称前加下划线。一般不用修改。
对于不希望编译器帮助自动生成`setter`和`getter`方法的情况，可以使用`@dynamic`关键字，使用此关键字声明后，编译器将不会为你生成`setter`和`getter`方法。这需要开发者自己去实现。
## 属性特质
属性可以设置特质，即我们在声明属性时常常会对其设定一些特性(nonatomic之类)，它们分别为：
### 原子性
默认情况下，编译器自动生成的方法会通过加锁的方式对属性设置原子性。即它会自动地添加`atomic`作为关键字（当然也可以自己加上，但是没有必要）。如果不想要你的属性拥有这种同步锁机制，则应该使用`nonatomic`关键字。
### 读写权限
* `readwrite` 表示可读也可写，编译器对其既会生成`setter`方法，也会生成`getter`方法。
* `readonly` 只读属性，只会生成`getter`方法。
### 属性与内存管理
属性还有一些关键字：`stong、assign、weak`等，不同的关键字声明的变量在`setter`方法中所做的事情也不同，对于一个属性的复制，到底是简单的赋值操作还是需要保留该值呢：
* `assign` 针对简单的数值类型(NSInteger,CGFloat)等类型。
* `strong` 定义了一种拥有关系，在`setter`方法内先保留新值，然后将旧值释放，最后赋予新值。
* `weak` 定义了一种非拥有关系，在`setter`方法内既不保留新值，也不释放旧值。在它指向的对象被销毁时，属性值置为 nil。
* `unsafe_unretained` 与`assign`相同，不过它适用于对象类型。当目标对象销毁后，属性的值不会自动清空。
* `copy` 和`strong`类似，但是在`setter`方法内不保留新值，而是做一次深拷贝。
### 在`init`方法中不调用`setter`方法
第一种情况：假设在 A 类的初始化中使用了 `self.XXX`来设置`XXX`这个属性，B 类继承自 A 类，在 B 类中重写了`XXX`的`setter`方法，那么在 A 类的默认的初始化方法中`self.XXX`调用的将会是子类的`setter`方法，这并不符合预期。
懒加载就不说了。没什么好说的。
# 判断对象是否相等
在`Objective-C`中，对于对象的相等比较不是可以通过`==`来比较的。使用`==`比较的是两个指针本身，并非对象。对于对象的比较应该使用 `NSObject` 协议中声明的`isEqual`方法来判断两个对象是否相等。`NSObject`协议中有两个用于判断相等的关键方法：
```
- (BOOL)isEqual:(id)object;
- (NSUInteger)hash;
```
如果对于自己的对象进行比较是否相等需要重写`isEqual`方法，那么就要先了解在基类`NSObject`中是怎么做的:`NSObject`类对于这两个方法的默认实现是：当且仅当它们的指针指向的内容相同时，两个对象才相等。“如果`isEqual`方法判定两个对象相等，那么它们的`hash`也一定相同，但是反之不一定成立”。
对于一个`Person`类：
```
#interface Person : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *describeStr;
@end
```
判断两个`Person`类的对象是否相等在于判断其所有属性字段是否相等：
```
- (BOOL)isEqual:(id)object {

    // 如果指针相同，即指向的内容一致，则认为相等。
    if(self == object)  return YES;

    // 如果两个对象都不是一个类属，则认为一定不相等
    if(![self isKindOfClass:[object class]])    return NO;

    //依次比较其属性，如果有一个属性不相同，则认为两个对象不相等
    Person *person = (Person *)object;
    if(![_name isEqualToString:person.name])
        return NO;
    if(![_describeStr isEqualToString:person.describeStr])
        return NO;

    return YES;

}
```
接下来就是对于`hash`方法的处理：“如果`isEqual`方法判定两个对象相等，那么它们的`hash`也一定相同，但是反之不一定成立”。
第一种重写`hash`方法的做法是：
```
- (NSUInteger)hash {
    return 12222;
}
```
这种方式是可行的，但是在`collection`（`NSArray` `NSDictionary` `NSSet`等数据结构）中，对象的索引是通过对象的 `hash`码确定的，对于`set`来说，它不允许有重复的两个对象同时存在，因此再向其中插入一个对象时，会根据该对象的`hash`码去查找所有该类对象，再在该类对象中一一检索是否相同。这种方式必然是低效的。
第二种重写`hash`方法的方式是：
```
- (NSUInteger)hash {
    NSString *stringToHash = [NSString stringWithFormat:@"%@:%@:%i", _name, _describeStr, _age];
    return [stringToHash hash];
}
```
这么做也是符合上述对于`hash`方法的要求的。也可以避免第一种方法提到的情况，但是处理字符串肯定比处理一个常量数值要来得复杂，随之而来的就是性能的问题，尤其是大量此类对象存到`collection`中时，因此还有第三种方式：
```
- (NSUInteger)hash {
    NSUInteger nameInteger = [_name hash];
    NSUInteger describeStrInteger = [_describeStr hash];
    NSUInteger ageIntger = _age;
    return nameInteger ^ describeStrInteger ^ ageIntger;
}
```
这种方法既保证了高效率也保证产生的`hash`码符合之前提到的要求。当然还是有可能会产生重复，但是重复冲突的情况并不普遍。
# 类簇隐藏实现细节
类簇类似于工厂模式，它对外暴露一个基类，但是具体该类可能是基类的某个子类。例如`UIKit`中有一个创建`UIButton`的方法：
```
+ (UIButton *)buttonWithType:(UIButtonType)type;
```
该方法返回的对象取决于传入的按钮类型。但是不论传入的类型是什么，产生的对象均是继承自基类`UIButton`。在使用`UIButton`时，并不需要关心创建出来的`UIButton`对象到底属于哪个子类，也不用考虑按钮的绘制方式等细节，只需要把注意力放在怎样设置`Button`的样式就好。
下面通过一个雇员(Employee)的例子来说明这种类簇的方式。假设有一个雇员的类，每个雇员都有名称和薪水这两个属性，但是不同类型的雇员所做的事情也不相同，在给其指派任务时，并不关心其做何种类型的工作，只需要调用`doWork`方法使其工作即可。
```
#import <Foundation/Foundation.h>

//定义一个枚举用于区分雇员类型
typedef NS_ENUM(NSInteger, EmployeeType) {
    EmployeeTypeDeveloper,
    EmployeeTypeDesigner,
    EmployeeTypeFinance,
};

@interface Employee : NSObject

//雇员姓名
@property (nonatomic, copy) NSString *employeeName;

//雇员薪水
@property (nonatomic, strong) NSNumber *employeeSalary;

+ (instancetype)employeeWithType:(EmployeeType)type;

- (void)doWork;

@end
```
在其对外暴露的`employeeWithType`方法里，需要根据不同的`type`返回不同子类对象：
```
+ (instancetype)employeeWithType:(EmployeeType)type {
    switch (type) {
        case EmployeeTypeFinance:
            return [EmployeeFinance new];
            break;
        case EmployeeTypeDesigner:
            return [EmployeeDesigner new];
            break;
        case EmployeeTypeDeveloper:
            return [EmployeeDeveloper new];
            break;
    }
}

- (void)doWork {

}
```
像这种基类接口没有`init`方法时，暗示这样的实例对象不应该由用户直接创建。在`Cocoa`框架中，几乎所有的`collection`类（`NSArray` `NSDictionary`等）都是类簇。
在使用这样的类时，避免下面这种判断方式：
```
if([employee isMemberOfClass:[Employee class]])
或者
if([employee class] == [Employee class])
```
因为类簇产生的实例变量可能是基类的子类，它不一定是该基类的成员。因此这种判断可能永远为`NO`。正确的判断方式如下：
```
if([employee isKindOfClass:[Employee class]])
```
在给类簇增加新的实体子类时（例如给`NSArray`增加新的子类），需要注意一些规则：
* 子类应该继承类簇中的抽象基类
* 子类应该自己定义自己的数据存储方式
* 子类应该覆写超类中指明需要覆写的方法
