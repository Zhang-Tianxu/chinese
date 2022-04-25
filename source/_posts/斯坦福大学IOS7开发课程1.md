---
title: 斯坦福大学IOS7开发课程1
date: 2021-03-15 07:13:41
tags:
    - IOS开发
    - Objective-C
categories:
    - 客户端开发
    - IOS开发
---
# 斯坦福大学IOS7开发课程1

IOS7开发课程讲了如何利用Objective-C语言开发IOS App，而且涉及到很多必须的基础知识，非常值得学习。

课程1讲了一些关于课程安排和作业的事情，自学就不需要了，只选取下面三个**知识点：**

- IOS里有什么
- MVC开发模型
- Objective-C

<!--more-->

## IOS里有什么

{% img inIOS https://tva1.sinaimg.cn/large/e6c9d24egy1gok87jp5hnj20lq15ctky.jpg 300 700 '"IOS里有什么" "加载失败"' %}

总的来说，可以将IOS涉及的组件分为4组或者说4层：

- Core OS

  最接近硬件的一层，其实就是Unix操作系统，Unix内核中有的功能，Core OS都有，比如：

  - OSX Kernel
  - Power Management
  - Sockets
  - File System

  等等……

- Core Services

  基于Core OS的面向对象层。

  Core OS层的API几乎都是用C写的，为了以*面向对象*的方式编程，加入了Core Services层。这一层包括：

  - Collections
  - File Access
  - Networking
  - Threading
  - SQLite
  - Core Location

  等等

- Media

  用于展现多媒体，包括视频、图片、声音、文件等等，是非常重要的一层。

- Cocoa Touch

  Cocoa的API起源于Mac OS X，已经有三十年左右的历史了，是UI层。对IOS开发者来说是需要花比较多时间来学习的一层。这一层可以用来构建按键、滑块、开关、文字输入、动画等等。包括：

  - Multi-Touch
  - Alerts
  - Web View
  - Camera
  - Controls

  等等。

## MVC开发模型

Model View Controller是一种用于组织应用程序中所有类的策略，将每个类分成Model 阵营、Controller 阵营或者View 阵营中的额一个。

Model用来描述你的程序是什么，以纸牌游戏为例，纸牌、牌桌甚至玩牌的规则都是独立于UI的，应该放在Model阵营中。至于纸牌是如何展示在屏幕上的，是由Controller负责的，Controller负责如何展现Model，以及展现Model的动画等。View是Controller的下属，是Controller用于构建UI的组件。

View是通用的，比如按键、开关等，是所有程序通用的，而Contrller是针对程序设计的，Model则是完全独立于UI的。

### MVC模型中三个阵营之间的通信

#### Controller -> Model

  Controller了解Model的一切，并且可以任意给Model发消息。

#### Controller -> View

  Controller也可以任意给View发送消息，如果Controller有个property指向View，把这个property成为***outlet***。

#### Model <-> View

  永远**不要**让Model和View之间直接通信。因为Model应该是完全独立于UI的，所以不应该给View发消息，而View是通用的，所以也不应该发消息给Model。

#### View -> Controller

View能向Controller发消息么？可以是可以，但是因为View是通用的，它对Controller并不了解，所以他们只能通过约定好的形式向Controller发送消息，有两种方式：

- target action

  Controller在内部建立一个target，并给View一个action。当View执行一些操作，比如按键被按下的时候，view会向Controller内的target发射action，这种方式下，View可以不了解Controller，只需要知道某些操作被执行时向Controller发送action。

- delegate

  target action方式没办法处理非常复杂的通信。比如在一个滚动框中，用户按下手指准备滚动，需要让Controller知道用户要滚动了。因为是否允许用户的滚动请求，View是不知道的，所以需要让Controller代理其执行。有一种特殊的delegate，称为data source。

  ![image-20210314123540992](https://tva1.sinaimg.cn/large/e6c9d24egy1gok7sf8e9mj20xm0jadhu.jpg)

View并不拥有它们展示的数据

#### Model -> Controller

那么model能向Controller发送消息么？

model应该是完全独立于UI的，所以不能直接向Controller发送消息。但有的时候确实有这样的需求，比如Model中数据变化了，需要告诉Controller，这时可以通过广播的方式，向所有感兴趣的人发送消息，其中当然可以包括Controller

![image-20210314123951433](https://tva1.sinaimg.cn/large/e6c9d24egy1gok7tedpf9j20wc0jstw6.jpg)

### MVCs模型

![image-20210314124445333](https://tva1.sinaimg.cn/large/e6c9d24egy1gok7tphhwkj211q0mg4qp.jpg)

## Objective-C

Objective-C的一些基础语法可以到[易百Objective-C教程](https://www.yiibai.com/objective_c/)学习，本课程中只是讲了一些和C/C++等语言不通的地方。和常见的编程语言C++或java相比，Objective-C有个重要的概念“Properties”，在Objective-C中一般不直接读写实例的变量，properties是Objective-C中读写实例的变量的方法，一般由getter函数和setter函数组成。

在Objective-C中，每个类由一个`.h`头文件的头文件和一个`.m`的实现文件。其中，`.h`中是公开的API，`.m`中是私密API和具体的实现。

在`.h`中声明类时，必须给定类的父类，其中`NSObject`是几乎所有类的父类。

```objective-c
// Test.h
@interface Test: NSObject
@end
```

由于使用了`NSObject`作为父类，所以需要引入这个类。

```objective-c
// Test.h
#import <Foundation/NSObject.h>
@interface Test: NSObject
@end
```

在IOS中一般不会只引进这个类，而是将整个`Foundation`框架引进来。

```objective-c
// Test.h
#import <Foundation/Foundation.h>
@interface Test: NSObject
@end
```

在`.m`实现文件中当然要引入头文件，`@implementation`表示类的具体实现。

```objective-c
// Test.m
#import "Test.h"
@implementation Test
@end
```

同时`.m`中可以添加一些私密API

```objective-c
// Test.m
#import "Test.h"
@interface Test()//添加私密API
@end
  
@implementation Test
@end
```

下面介绍Objective-C的Properties概念，通过`@property`在`.h`中添加一个属性

```objective-c
// Test.h
#import <Foundation/Foundation.h>
@interface Test: NSObject
  @property (strong nonatomic) NSString *name;
@end
```

下面详细说一下`property`这行代码的构成：

**首先**是关键字`(strong)`。在Objective-C中，所有的对象都被放在堆中，通过指针使用它们。通过这种方式，Objective-C不需要编程人员手动分配和释放内存地址，那么Objective-C如何知道何时释放内存呢？这就是属性声明中`(stong)`的作用，相应的还有`(weak)`。在内存管理时，只要有任意一个`strong`指针指向内存，内存就会被保留，最有一个`strong`指针被删除时，该地址会**立即**被回收，此时如果还有`weak`指针指向改地址，`weak`指针会被置为空。

对于元类型（int、bool等）的属性，不需要声明`strong`或`weak`，因为这种类型的属性并不存储在堆中，不需要内存管理。

与其他语言不通，在Objective-C中，引用空指针并不会导致程序崩溃，在Objective-C中你甚至可以给空指针发送数据，也不会导致程序崩溃，当然你给空指针发数据，虽然不会导致程序崩溃，也不会执行任何代码，如果发送的消息需要返回，返回值会被设为0。

**然后**是关键字`nonatomic`，非原子操作。这个关键字的意思是属性的getter和setter**不是线程安全的**，在IOS中一般并不需要属性是线程安全的。在声明属性的同时，Objective-C会自动创建getter和setter方法。如果添加了`nonatomic`关键字，自动生成的代码是：

```objective-c
@synthesize name = _name;
- (NSString *)name
{
  return _name;
}
- (void)setName:(NSString *)name
{
  _name = name;
}
```

其中`@synthesize`表示`_name`是`name`的别名。这些代码并不会显示出来，但可以直接使用。使用setter和getter的方法：

```objective-c
myTest.name = @"Hello World";
NSLog(@"%@",myTest.name);
```

当然也可以用中括号的方式调用，但建议使用上面的方式调用setter和getter。

```objective-c
[myTest setName:@"Hello World"];
NSLog(@"%@",[myTest name]);
```

如果不添加`nonatomic`关键字，自动生成的代码会复杂很多，因为需要添加锁。为了简单，一般都会加上`nonatomic`关键字。

**另外**，如果想要重命名属性的getter和setter，可以通过一下方式：

```objective-c
@property (nonatomic, getter = getName, setter = setName) NSString name;
```
