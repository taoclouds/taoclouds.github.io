title: 【读书笔记】Effective Objective-C 2.0
date: 2018/09/10
categories:
- iOS 开发
tags:
- Objective-C
- 编程
---

# 了解 Objective-C 的起源
Objective-C 的语法由 [Smalltalk](https://zh.wikipedia.org/wiki/Smalltalk) 演化而来，属于消息型语言。使用消息结构的语言，在运行时执行的代码由运行环境来决定，这点与函数调用型语言不同。因此，Objective-C 有一套运行时组件来完成对象的动态绑定、消息的发送与接收等等一系列工作，它在运行时才会对对象的类型做检查。这套运行时组件(称作 Runtime机制)在程序运行时将程序代码粘合起来，对于提升程序性能来说十分方便。

Objective-C 是C语言的超集，即意味着符合 C语言规范的代码在 Objective-C 下仍然适用。因此，对于 Objective-C 和 C 的内存模型就要理解清楚，这样才能写出高质量的代码。
```
NSString *someStr = @"someStr";
NSString *otherStr = someStr;
```
声明一个 `NSString *`指针，用于指向一个字符串对象`@"someStr"`，该对象是存储在堆中的，而指向它的指针`someStr`是存储在栈上的。

分配在堆中的内存是需要手动进行管理的，例如 C语言提供的`malloc`和`free`函数，分别用于申请和释放内存。在 Objective-C 中，有一套内存管理机制(称为自动引用计数)专门来管理分配在堆中的内存。对于直接使用栈空间存储的变量(例如非 Objective-C 对象)，如 CGRect 变量，它是一个结构体：
```
struct CGRect {
    CGPoint origin;
    CGSize size;
};
typedef struct CGRect CGRect;
```
使用结构体而不使用对象的原因是一个 iOS 应用程序将会大量使用到此结构体，如果改成对象来做只会降低性能，因为对象的创建还需要额外的开销，如分配合是否内存。

# 在类的头文件中尽量少引入其他头文件
在引入头文件的过程中，重复引用多次头文件或者引入冗余的头文件会导致增加编译时间。因此尽量将引入头文件的时机延后，在确定需要使用的时候再在`.m`文件中引入：
```
@class XXX;   代替  #import "XXX.h"
```
这种方法称为向前声明，这种方式也解决了两个类互相引用的问题：当 A类中用到 B类的对象，B类中也用到了 A类的对象时，使用`#include`的方式引入会造成两个类的循环引用，当然使用`#import`的方式会避免循环引用的问题，但是这样会导致两个类中有一个类无法被正确编译。

还有另外一种情况，就是如果某个类遵守一项协议，那么就需要在该类的`.h`文件里引入该头文件。此时，可以将引入头文件的声明移到分类中去做。或者将协议单独放到一个头文件中，然后再引入。
# 多用字面量语法，少用与之等价的方法
字面量语法，即形如：
```
NSString *str = @"strstr";
NSNumber *number = @(10);
NSArray *array = @[str,number];
```
这类初始化一个对象的方法，当然也可以通过 `alloc init`的方式去初始化。使用字面量语法可以使得代码更加简洁。需要注意的是在使用字面量语法创建一个数组或者一个字典的时候，数组中的元素不能是 nil。 如果为 nil 则编译器会直接报异常警告，这也侧面反映了使用字面量语法的好处（使用 alloc 方法会在遇到 nil 就停下）。可以使用`NSNull`把`nil`包装一下。
# 使用类型常量代替`#define`预处理指令
使用预处理指令，编译器只是对其做一个简单的替换，并不会对其做类型检查，一个可取的办法是在定义常量的时候，尽量使用下面这种方式：
```
static const NSTimeInterval kAnimationDuration = 0.3;
```
这种方式定义的常量包含了类型信息，清楚地描述了常量的含义。对于代码阅读和编译都是有好处的。

其次是对于常量的声明定义应该尽量放在`.m`文件里，因为如果放在头文件中，则在引入头文件时，一不小心就会有命名重复的情况出现（因为 OC 中没有 namespace）。同时，必须要加`static`关键字，它表示要声明的变量是一个静态变量，这样编译器就不会在外部为它创建一个外部符号，避免了在外部同名变量冲突的情况。
实际上，对于`static`和`const`同时存在的情况下，编译器对于这种方式声明的变量也是仅仅做一个简单的替换，但是这种替换是带有类型信息的。

有一种情况需要对外公开某个变量，例如对于通知的注册和发送，都需要一个字符串来表示这么一个唯一的通知，此时这个通知名需要对外暴露，使之可见，但是具体的字符串是什么是不需要对外暴露的。
```
在.m 文件内声明：
NSString * const WTColorChangedNotification = @"WTColorChangedNotification";
在.h 文件中暴露：
extern NSString * const WTColorChangedNotification = @"WTColorChangedNotification";
```
使用`extern`关键字声明的常量，它会放在全局符号表中，以便可以将它在文件外使用，当编译器识别了`extern`关键字，便会默认在全局符号表中会有这个变量的符号，即允许其他地方使用此常量。这类常量只允许定义一次，并且在定义时注意命名，使用类名作为前缀可以避免冲突。

# 使用枚举表示状态、选项、状态码
枚举类型在项目中多用来表示状态码或者某些可用数值类型来表示的选项。C语言中的枚举类型定义如下：
```
enum WTSomeEnumState {
    WTSomeEnumStateConfirm,
    WTSomeEnumStateCancel,
    WTSomeEnumStateUncertain,
};
enum WTSomeEnumState state = WTSomeEnumStateConfirm;
```
枚举的取值从0开始，逐步递增。为了避免每次定义枚举变量时都要写`enum ***`,可以使用`typedef`关键字。
```
typedef enum WTSomeEnumState WTSomeEnumState;
```
C++ 11标准规定，可以指明使用何种数据类型来保存枚举类型的变量。原来的定义枚举的方式(上面 C语言那种)由编译器去判断具体的枚举类型到底是`int`还是`char`。有了这个 C++新特性，可以在声明枚举时，指定枚举的数据类型。
```
enum WTSomeEnumState : NSInteger {
    WTSomeEnumStateConfirm,
    WTSomeEnumStateCancel,
    WTSomeEnumStateUncertain,
};
```
对于定义选项的枚举类型，尤其是当选项可以组合时，选项之间可以通过按位或操作符来组合：
```
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```
这里说明一下`1 << 0 1 << 1`等等的用法。它们的意思是把1左移多少位，为什么要这么做呢，这是为了方便各个枚举项之间的组合以及是否启用了某个枚举项。举例说明：
[]
正如上图所示，对于每个枚举项，它所代表的数值类型是由1挨个左移得到的，因此，如果我们想组合`UIViewAutoresizingFlexibleWidth` 和 `UIViewAutoresizingFlexibleHeight`，那么就可以这样：`UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight`，因为是或操作，因此相应的位为1。在判断某个视图是否启用了某个枚举项时，只需要`if(view.resizing & UIViewAutoresizingFlexibleHeight)` ，采用按位与操作，结果为`true`表示启用，`false`为未启用。

枚举还可以用在 `switch` 语句中：
```
switch (self.status) {
    WTSomeEnumStateConfirm:
    //handle
    break;
    WTSomeEnumStateCancel:
    //handle
    break;
    WTSomeEnumStateUncertain:
    //handle
    break;
}
```
需要注意的是：在`switch`分支中使用时，不必加上`default`分支，这会给其加上一个多余的`default`状态，编译器会报错的。
