---
title: 斯坦福大学IOS7开发课程4
date: 2021-03-29 22:27:07
tags:
    - IOS开发
    - Objective-C
categories:
    - 客户端开发
    - IOS开发
---
这节课主要讲了**Objecitive-C的几个细节**和**Foundation**库中的一些类

本节课主要介绍iOS开发

- NSObject
- NSArray
- NSMutableArray
- NSNumber
- NSValue
- NSData
- NSDate
- NSSet
- NSMutableSet
- NSOrderedSet
- NSMutableOrderedSet
- NSDictionary
- NSMutableDictionary
- NSUserDefaults
- NSRange
- UIColor
- UIFont
- UIFontDescriptor
- NSAttributedString
- NSMutableAttributedString

<!--more-->

# Objecitive-C的几个细节

## 对象的创建

1. 利用Alloc & init创建对象
2. 利用类的方法创建对象
3. 或者上述两者同时使用
4. 让其他对象帮忙创建类的对象
   ```objc
   - NSString's -- (NSString *)stringByAppendingString:(NSString *)otherString
   - NSArray's -- (NSString *)componentsJoinedByString:(NSString *)separator;
   - NSString's & NSArray's -- (id)mutableCopy
   ```

## nil

向nil发送消息是可以的，但是不会执行任何代码

## 动态绑定

Objective-C有个**非常重要**的类型——**id**，id是一种类型，意味“指向未知类型对象的**指针**”。

**在运行过程中，所有对象的指针都是id类型的。**

用id来指定变量类型，**就是动态绑定**。

但是在编译过程中，如果你指定对象的类型，编译器可以帮助发现bug。

**那么动态绑定安全么？**

动态绑定导致可以向对象发送未定义的信息，比如该对象类型为定义的方法，当然这会导致程序崩溃。

如果在代码中指定对象的类型，这种错误就可以在编译过程中被发现，防止程序在执行过程中崩溃，比如，`NSString *s  = @'x'`，这种情况下，如果给s发送非NSString的方法，编译器可以发现错误。

当然也可以使用：`id obj = s`、`NSArray *a = obj`，但是这样**编译器没办法帮忙检查错误**，所以存在一定危险。

另外，永远不要使用`id *`，因为**id本身就是个指针**。

```objc
@interface Vehicle
- (void)move;
@end
@interface Ship : Vehicle
- (void)shoot;
@end
```

```objc
Ship *s = [[Ship alloc] init];
[s shoot];
[s move];

Vehicle *v = s; // 合法
[v shoot]; // 编译器会给出警告，因为它以为v是Vehicle
          //但是运行时不会崩溃，因为v实际上是一个Ship

id obj = ...; // 任意定义obj，比如NSString；
[obj shoot;] // 编译器不会给出警告，因为它知道shoot函数确实存在，同时它又不知道obj的类型，因此编译器无法给出警告。
// 但是，实际运行中，程序会崩溃。
[obj someMethodNameThatNoObjectAnywhereRespondsTo];
// 这个时候编译器会给出警告，因为它知道没有任何函数叫someMethodNameThatNoObjectAnywhereRespondsTo

NSString *hello = @"hello";
[hello shoot]; // 编译器会给出警告，因为NSString没有shoot函数
Ship *helloShip = (Ship *)hello; // 编译器不会给出警告 
[helloShip shoot];// 导致运行时崩溃
[(id)hello shoot]; //编译器不会给出警告，但是运行时会崩溃
```

**什么时候会用到危险的动态绑定（id）**

1. Objective-C允许在同一个collection（比如NSArray）中有不同类型的类，想这么做时，需要用到动态绑定。 
2. 当希望使用MVC中的blind/structured通信（比如delegation）时。

**为了把动态绑定变得安全**，需要使用Introspection和Protocols

- Introspection

  在运行时询问id是什么类型或询问可以向其发送的信息。

  继承自NSObject的所有对象都有下面三个函数：

  1. `isKindOfClass:`

     判断对象是不是某种类，包含继承的情况

     ```objc
         if([obj isKindOfClass:[NSString class]]) {
             NSString *s = [(NSString*)obj stringByAppendingString:@"xyzzy"];
         }
     ```
     比如在纸牌匹配游戏中`PlayingCard.m`的`match`函数的源代码为：
     ```objc
     - (int)match:(NSArray *)otherCards
      {
         int score = 0;
       if([otherCards count] == 1) {
             PlayingCard *otherCard = [otherCards firstObject];
           if([self.suit isEqualToString:otherCard.suit]) {
                 score = 1;
             } else if (self.rank == otherCard.rank) {
                 score = 4;
             }
         }
         return score;
     }
     ```
     可以改为：

     ```objc
     - (int)match:(NSArray *)otherCards
     {
         int score = 0;
         if([otherCards count] == 1) {
             id card = [otherCards firstObject];
             if([card isKindOfClass:[PlayingCard class]]) {
                 PlayingCard *otherCard = (PlayingCard *)card;
                 if([self.suit isEqualToString:otherCard.suit]) {
                     score = 1;
                 } else if (self.rank == otherCard.rank) {
                     score = 4;
                 }
             }
         }
         return score;
     }
     ```

  2. `isMemberOfClass:`

     判断对象是不是某种类，不包括继承的情况

  3. `respondsToSelector:`

     判断对象能否相应某个函数
     
     ```objc
         if([obj respondsToSelector:@selector(shoot)]) {
             [obj shoot];
         } else if ([obj respondsToSelector:@selector(shootAt:)]) {
             [obj shootAt:target];
         }
     ```
     
  - 在objective-C中，SEL是selector的一种类型

    ```objc
    SEL shootSelector = @selector(shoot);
    SEL shootAtSelector = @selector(shootAt:);
    SEL moveToSelelctor = @selector(moveTo:withPenColor);
    ```

  - 给定SEL，可以要求对象执行selector

    ```objc
    [obj performSelector:shootSelector];
    [obj performSelector:shootAtSelector withObject:coordinate];
    ```

    ```objc
    [array makeObjectsPerformSelector:shootSelector];
    // cool, huh?
    [array makeObjectsPerformSelector:shootAtSelector withObject:target];
    // target is an id  
    ```

- Protocols

  不指定指针指向的对象类型，但是指定其实现的函数。

# Foundation库

## NSObject

在iOS SDK中 ，NSObject几乎是所有类的基类。

NSObject包含一个非常有用的函数`- (NSString *)description`，会返回对类的描述，所以最好在自己实现的类中对其重写。该函数一般用在两个地方：

- 用NSLog打印出来，`NSLog(@"array contents are  %@", myArray);`，用于debug

- 并不是所有的对象都实现了`- (id)copy`和`- (id)mutableCopy`，在未实现它们的对象中调用，会抛出异常

- 你向可修改对象（比如MutableArray）发送copy，返回值并不是可修改对象，而是获得一个不可修改的对象。

## NSArray

NSArray是对象的有序集合，不可修改（immutable），一旦创建，不能添加或删除对象。

只要NSArray本身在内存的堆中，其中所有的对象都有strong指针指向它们。

一般通过`@[]`手动创建NSArray，NSArray中包括以下一些关键函数：

```objc
- (NSUInteger)count;
- (id)objectAtIndex:(NSUInteger)index; //如果index超出边界，程序崩溃
- (id)lastObject; //如果数组为空，返回nil，不会导致程序崩溃
- (id)firstObject; //如果数组为空，返回nil，不会导致程序崩溃

- (NSArray *)sortedArrayUsingSelector:(SEL)aSelector;
- (void)makeObjectsPerformsSelector:(SEL)aSelector withObject:(id)selectorArgument;
- (NSString *)componentsJoinedByString:(NSString *)separator;
```

## NSMutableArray

是NSArray的可变版本（mutable），继承了NSArray的所有函数

一般通过`alloc/init`创建，或者通过以下函数创建

```objc
+ (id)arrayWithCapacity:(NSUInteger)numItems;
+ (id)array;//[NSMutableArray array] 等价于 [[NSMutableArray alloc] init]
```

对于NSArray和NSMutableArray两种数组，可以用for-in的方式遍历：

Objective-C支持for-in形式的数组遍历。需要注意的是，for-in形式的数组遍历默认会进行强制类型转换，比如：

```objc
NSArray *myArray = ...;// 任意类型数组
for (NSString *string in myArray) { //编译器不知道数组中的数据类型
  double = value = [string doubleValue];// 如果string的类型并不是NSString，会导致程序崩溃
}
```

那么如果数组中的数据类型不同，如何进行遍历呢？可以使用id

```objc
NSArray *myArray = ...;// 同样，任意类型的数组
for (id obj in myArray) {
  // 通过某些方法确认obj是你想要的类型，防止程序崩溃
  if([obj isKindOfClass:[NSString class]]) {
    // 这是确认了obj的类型是NSString
  }
}
```

## NSNumber

NSNumber是对元类型int、float、double、BOOL、enum等的封装。**当你想把这些元类型的数据放入同一个数组时很有用。**

## NSValue

类似于NSNumber，但是用来封装一些非对象、非元类型的数据，例如c structs。

## NSData

用于存储bits

## NSDate

用于存储日期，相关的类型还有NSCalendar，NSDateFormatter，NSDateComponents。

## NSSet/NSMutableSet

## NSOrderedSet/NSMutableOrderedSet

## NSDictionary

存储键值对，不可修改。

NSDictionary可以用`@{key1:value1, key2:value2}`创建，比如：

```objc
NSDictionary *colors = @{ @"green" : [UIColor greenColor],
                        @"blue" : [UIColor blueColor],
                        @"red" : [UIColor redColor]};

NSString *colorString = @"red";
UIColor *colorObject = colors[colorString];
```

## NSMutableDictionary

NSDictionary的可修改版本，出了继承了NSDictionary所有的函数，还有下面几个重要函数：

```objc
- (void)setObject:(id)anObject forKey:(id)key;
- (void)removeObjectForKey:(id)key;
- (void)removeAllObjects;
- (void)addEntriesFromDictionary:(NSDictionary *)otherDictionary;
```

Dictionary和NSMutableDictionary可用如下方法遍历：

```objc
NSDictionary *myDictionary = ...;
for (id key in myDictionary) {
  // do something with key here
  id value = [myDictionary objectForKey:key];
  // do something with value here
}
```

## NSUserDefaults

介绍NSUserDefaults之前，需要先介绍一个术语：***Property List***

Property List是一个iOS开发中常用术语，在一些API中会用到，比如：

```objc
- (void)writeToFile:(NSString *)path atomically:(BOOL)atom;
```

这个函数只能发送给NSArray或NSDictionary，并且其中只能包含Property List对象。

- **概念：**

  Property List可以理解为只包含NSArray、NSDictionary、NSNumber、NSString、NSDate、NSDate的一个集合，并且Property List是一个递归的概念。

  比如一个由NSString组成的NSArray，就是一个Property List

  一个由NSArray组成的NSArray是不是一个Property List要看子NSArray是不是Property List，如果是，那组成的NSArray就是一个Property List，否则不是。

  对于NSDictionary，只有当其所有key和value都是Property List的时候才是Property List

  那么，假设有一个NSArray，其中包含了多个NSDictionary，这些NSDictionary的key都是NSString，value都是NSNumber，那么这个NSArray也是个Property List

NSUserDefaults就是Property List的轻量级存储，可以看成一个持久存储的NSDictionary。算不上一个数据库，通常只用来存储小数据，比如用户设置。

利用standardUserDefaults读写：

```objc
[[NSUserDefaults standardUserDefaults] setArray:rvArray forKey:@"RecentlyViewed"];
```
```objc
- (void)setDouble:(double)aDouble forKey:(NSString *)key;
- (NSInteger)integerForKey;(NSString *)key; // NSInteger is a typedef to 32 or 64 bit int
- (void)setObject:(id)obj forKey:(NSString *)key; //obj must be a Property List
-(NSArray *)arrayForKey:(NSString *)key; // will return nil if value for key is not NSArray
```

更改后一定要使用synchronize进行同步，才能写入永久内存。

```objc
[[NSUser Defaults standardUserDefaults] synchronize];
```

## NSRange

是一个类似C语言中struct的结构，用于指定string和数组的界限，其结构如下：

```objc
typedef struct {
  NSUInteger location;
  NSUInteger length;
} NSRange;
```

使用方法如下：

```objc
NSString *greeting = @"hello world";
NSString *hi = @"hi";
NSRange r = [greeting rangeOfString:hi];
if(r.location == NSNotFound) {
  //没能在greeting中找到hi
}
```

## UIColor

一个非常简单的类，用于表示颜色。可以用RGB或者HSB等方法初始化。

## UIFont

字体在UI设计中非常重要，iOS在不同截面显示的字体变化很大，所以要重视UIFont这个类。

对于展示内容而言，使用Font**最好的方法**是调用`preferredFontForTextStyle`

````objc
UIFont *font = [UIFont preferredFontForTextStyle:UIFontTextStyleBody];
````

除了`UIFontTextStyleBody`，还有`UIFontTextStyleHeadline`、`UIFontTextStyleCaption1`、`UIFontTextStyleFootnote`等等。

对于按键内容等，可以使用系统字体：

```objc
+ (UIFont *)systemFontOfSize:(CGFloat)pointSize;
+ (UIFont *)boldSystemFontOfSize:(CGFloat)pointSize;
```

**不要将系统字体用于内容。**

## UIFontDescriptor

字体是由艺术家设计的，并没有特定的准则，有些字体甚至没有加黑。尽管如此，UIFontDescriptor尝试对所有字体进行分类。

## NSAttributedString

iOS开发中经常需要显示一些带有特殊样式的文本，比如说带有下划线、删除线、斜体、空心字体、背景色、阴影以及图文混排（一种文字中夹杂图片的显示效果）。

通常想要实现这些效果要使用到iOS的Foundation框架提供的NSAttributedString类，NSAttributedString类中有许多属性，不同属性对应不同的文本样式。本文主要对这些属性做一个解释说明，并会结合实际代码来应用它们。

可以将NSAttributedString想象成NSString，其每个字母有个叫做attributes的NSDictionary。 

NSAttributedString**不是**NSString的继承，所以不能使用NSString的函数。

## NSMutableAttributedString

相对于NSAttributedString，NSMutableAttributedString更加常用。

添加或者设置字母对应attributes的方法：

```objc
- (void)addAttributes:(NSDictionary *)attributes range:(NSRange)range;
- (void)setAttributeds:(NSDictionary *)attributes range:(NSRange)range;
-(void)removeAttribute:(NSString *)attributeName range:(NSRange)range;
```

```objc
UIColor *yellow = [UIColor yellowColor];
UIColor *transparentYellow = [yellow colorWithAlphaComponent:0.3];
@{ NSFontAttributeName:
     [UIFont preferredFontWithTextStyle:UIFontTextStyleHeadline],
   NSForegroundColorAttributeName : [UIColor greenColor], // 字体颜色
   NSStrokeWidthAttributeName : @-5,
   NSStrokeColorAttributeName : [UIColor redColor],
   NSUnderlineStyleAttributeName : @(NSUnderlineStyleNone), // 设置字体描边
   NSBackgroundColorAttributeName : transparentYellow // 设置字体背景颜色
 }
```

![image-20210329221812138](https://tva1.sinaimg.cn/large/008eGmZEgy1gp14yomq5ij30ds0oggrt.jpg)

attributed strings用在哪里？

```objc
// UIButton
- (void)setAttributedTitle:(NSAttributedString *)title forState:...;

// UILable
@property (nonatomic, strong) NSAttributedString *attributedText;

// UITextView
@property (nonatomic, readonly) NSTextStorage *textStorage;
```



