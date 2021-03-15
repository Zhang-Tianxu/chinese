---
title: 斯坦福大学IOS7开发课程2
date: 2021-03-15 20:52:38
tags:
    - IOS开发入门
    - Objective-C
    - Xcode
categories:
    - 客户端开发
    - IOS开发
    - Objective-C
---
# 斯坦福大学IOS7开发课程2

这节课就着手开始实现"Card Matching Game"，纸牌匹配游戏。

具体来说就是实现涉及的各种类，以及XCode的简单使用。

<!--more-->

## Card类

下面代码是[课程1](https://zhang-tianxu.github.io/chinese/2021/03/15/%E6%96%AF%E5%9D%A6%E7%A6%8F%E5%A4%A7%E5%AD%A6IOS7%E5%BC%80%E5%8F%91%E8%AF%BE%E7%A8%8B1/)结束时的Card类，包括`Card.h`和`Card.m`：

```objective-c
// Card.h
#import <Foundation/Foundation>
@interface Card : NSObject
@property (strong nonatomic) NSString *contents;

@property (nonatomic, getter=isChosen) BOOL chosen;
@property (nonatomic, getter=isMatched) BOOL matched;
- (int)match:(NSArray *)otherCards;
@end
```



```objective-c
// Card.m
#import "Card.h"
@interface Card()
@end
@implementation Card

- (int)match:(NSArray *)otherCards
{
  int score = 0;
  for(Card *card in otherCards) {
    if([card.contents isEqualToString:self.contents])
      score = 1;
  }
}
@end
```

## Deck类

这节课首先添加另外一个类：整幅牌Deck，下面是上节课学的类的基本结构

```objective-c
// Deck.h
#import <Foundation/Foundation.h>

@interface Deck : NSObject
@end
```

```objective-c
// Deck.m
#import "Deck.h"
@interface Deck()
@end
@implementation Deck
@end
```

接下来为Deck类添加两个基础的方法:

- 向Deck中添加牌的`addCard`，其中atTop表示是否将新的牌放在牌堆的顶部。
- 另一个是从Deck中随机抽取牌的`drawRandomCard`：

```objective-c
// Deck.h
#import <Foundation/Foundation.h>
#import "Card.h"

@interface Deck : NSObject
- (void)addCard:(Card *)card atTop:(BOOL)atTop;

- (Card *)drawRandomCard;
@end
```

如果希望`addCard`的参数`atTop`是**可选参数**，在Objective-C中唯一的方法就是声明一个新的函数，这个函数也叫`addCard`，但是没有参数`atTop`。这两个`addCard`是两个不同的函数，相互之间并没有关联。

```objective-c
// Deck.h
#import <Foundation/Foundation.h>
#import "Card.h"

@interface Deck : NSObject
- (void)addCard:(Card *)card atTop:(BOOL)atTop;
- (void)addCard:(Card *)card;

- (Card *)drawRandomCard;
@end
```

为了存储牌堆中的牌，需要在私密API中添加一个属性，类型是可变数组`NSMutableArray`。mutable意味着可以向数组中添加或删除数据，而普通的`NSArray`是不能修改的，一旦被建立，不能添加数据也不能删除。

**在Objective-C中声明数组没办法指定数据类型。**

```objective-c
// Deck.m
#import "Deck.h"
@interface Deck()
@property (strong, nonatomic) NSMutableArray *cards; // of Card
@end
@implementation Deck
  
- (void)addCard:(Card *)card atTop:(BOOL)atTop
{
  if (atTop) {
    [self.cards insertObject:card atIndex:0];
  } else {
    [self.cards addObject:card];
  }
}

- (void)addCard:(Card *)card
{
  [self addCard:card atTop:NO];
}

- (Card *)drawRandomCard
{
  
}
@end
```

根据上节课说到构建属性时自动生成的getter和setter，新建一个Deck类后，其所有的变量和属性都***会被自动初始化为0/nil***，属性`cards`是一个空指针，因此调用`addCard`时虽然不会导致程序的崩溃，但是也不能正常工作。

那么怎么解决这个问题呢？方法是重写属性的getter函数，在getter中添加一个判断结构，属性cards自动生成的getter是：

```objective-c
- (NSMutableArray *)cards
{
  return _cards;
}
```

为了在其中添加一个判断逻辑，需要手动重写这个getter：

```objective-c
// Deck.m
#import "Deck.h"
@interface Deck()
@property (strong, nonatomic) NSMutableArray *cards; // of Card
@end
@implementation Deck

// 为了处理cards为空指针的情况，手动重写getter
- (NSMutableArray *)cards
{
  if(!_cards) _cards = [[NSMutableArray alloc] init];
  return _cards;
}

- (void)addCard:(Card *)card atTop:(BOOL)atTop
{
  if (atTop) {
    [self.cards insertObject:card atIndex:0];
  } else {
    [self.cards addObject:card];
  }
}

- (void)addCard:(Card *)card
{
  [self addCard:card atTop:NO];
}

- (Card *)drawRandomCard
{
  
}
@end
```

接下来实现`drawRandomCard`

```objective-c
// Deck.m
#import "Deck.h"
@interface Deck()
@property (strong, nonatomic) NSMutableArray *cards; // of Card
@end
@implementation Deck

- (  NSMutableArray *)cards
{
  if(!_cards) _cards = [[NSMutableArray alloc] init];
  return _cards;
}

- (void)addCard:(Card *)card atTop:(BOOL)atTop
{
  if (atTop) {
    [self.cards insertObject:card atIndex:0];
  } else {
    [self.cards addObject:card];
  }
}

- (void)addCard:(Card *)card
{
  [self addCard:card atTop:NO];
}

- (Card *)drawRandomCard
{
  Card *randomCard = nil;
  
  if([self.cards count]) {
    unsigned index = arc4random() % [self.cards count];
    randomCard = self.cards[index];
    [self.cards removeObjectAtIndex:index];
  }
  
  return randomCard;
}
@end
```

至此，Deck类已经完成。

## PlayingCard类

接下来再添加一个类：`PlayingCard`，同样的，基本结构如下：

```objective-c
// PlayingCard.h
#import "Card.h"
@interface PlayingCard : Card // 终于有个类的父类不是NSObject了
  
@end
```

```objective-c
// PlayingCard.m
#import "PlayingCard.h"
@implementation PlayingCard
  
@end
```

给类添加两个属性`suit`和`rank`，前者表示牌的花色“桃（hearts）”、“（方片）diamons”、“（梅花）clubs”，后者表示1到13。另外在`.m`文件中，重写父类属性content的getter。

```objective-c
// PlayingCard.h
#import "Card.h"
@interface PlayingCard : Card

@property (strong, nonatomic) NSString *suit;
@property (nonatomic) NSUInteger rank;
  
@end
```

```objective-c
// PlayingCard.m
#import "PlayingCard.h"
@implementation PlayingCard
- (NSString *)contents
{
  return [NSString stringWithFormat:@"%d%@",self.rank, self.suit];
}
@end
```

注意到，字符串前面有个`@`，这***表示把字符串变成一个字符串类***。其中`%@`表示一个对象，当然可以是字符串。

重写父类属性contents的getter之后，获取contents内容会返回“数字 + 花色”，比如“3红桃”、“1梅花”、“13方片”等。但是在纸牌中，我们一般会把1说成A，11说成J等……，为了符合这个习惯，上面的contents重写可以改成下面的代码：

```objective-c
// PlayingCard.m
#import "PlayingCard.h"
@implementation PlayingCard
- (NSString *)contents
{
  NSArray *rankStrings = @[@"?",@"A",@"2",@"3",@"4",@"5",@"6",@"7",@"8",@"9",@"10",@"J",@"Q",@"K"];
  // 将0-13表示为?,1,2,...,J,Q,K
  return [rankStrings[self.rank] stringByAppendingString:self.suit];
}
@end
```

将rank的0设置为”？“是因为objective-C默认将rank初始化为0，”？“表示这是未知的，没有经过设置的。那如果花色没经过设置也会显示”？“就更好了，解决方法也是重写suit属性的getter函数。

```objective-c
// PlayingCard.m
#import "PlayingCard.h"
@implementation PlayingCard
- (NSString *)contents
{
  NSArray *rankStrings = @[@"?",@"A",@"2",@"3",@"4",@"5",@"6",@"7",@"8",@"9",@"10",@"J",@"Q",@"K"];
  // 将0-13表示为?,1,2,...,J,Q,K
  return [rankStrings[self.rank] stringByAppendingString:self.suit];
}

- (NSString *)suit
{
  return _suit?_suit:@"?";
}
@end
```

suit的选择应该只有四种，为了防止suit被设置为其他值，还要重写suit属性的setter：`setSuit`
```objective-c
// PlayingCard.m
#import "PlayingCard.h"
@implementation PlayingCard
- (NSString *)contents
{
  NSArray *rankStrings = @[@"?",@"A",@"2",@"3",@"4",@"5",@"6",@"7",@"8",@"9",@"10",@"J",@"Q",@"K"];
  // 将0-13表示为?,1,2,...,J,Q,K
  return [rankStrings[self.rank] stringByAppendingString:self.suit];
}
@synthesize suit = _suit;
// 因为重构了suit的getter和setter，所以要手写@synthesize
- (void)setSuit:(NSString *)suit
{
  if([@[@"♥️",@"♦️",@"♠️",@"♣️"] containsObject:suit]) {
    _suit = suit;
  }
}

- (NSString *)suit
{
  return _suit?_suit:@"?";
}
@end
```

值得注意的是，`setSuit`中的第一个`@`表示创建新的数组，在这里，每次判断都会新建这个数组。为了性能和代码简介，可以新建函数来判断setter收到的suit是否有效。实际上，这种改变对性能的提升是极其有限的。

```objective-c
// PlayingCard.m
#import "PlayingCard.h"
@implementation PlayingCard
- (NSString *)contents
{
  NSArray *rankStrings = @[@"?",@"A",@"2",@"3",@"4",@"5",@"6",@"7",@"8",@"9",@"10",@"J",@"Q",@"K"];
  // 将0-13表示为?,1,2,...,J,Q,K
  return [rankStrings[self.rank] stringByAppendingString:self.suit];
}
@synthesize suit = _suit;
// 因为重构了suit的getter和setter，所以要手写@synthesize

+ (NSArray *)validSuits
{
  return @[@"♥️",@"♦️",@"♠️",@"♣️"];
}
- (void)setSuit:(NSString *)suit
{
  if([[PlayingCard validSuits] containsObject:suit]) {
    // 类的函数的调用方法。
    _suit = suit;
  }
}

- (NSString *)suit
{
  return _suit?_suit:@"?";
}
@end
```

这里在函数实现前面第一次出现了`+`符号，这个符号表示这个函数是**类的函数**（而不是对象的函数）。

***一般只在两种情况下使用类的函数：***

1. 工具函数（utility method），比如这里的`validSuits`。
2. 创建类的函数，比如`stringWithFormat`。

 对rank属性做同样的检查和优化，并添加一个公开API`maxRank`返回rank的最大值，比如现在是13。

```objective-c
// PlayingCard.h
#import "Card.h"
@interface PlayingCard : Card

@property (strong, nonatomic) NSString *suit;
@property (nonatomic) NSUInteger rank;

+ (NSArray *)validSuits;
+ (NSUInteger)maxRank;
  
@end
```

```objective-c
// PlayingCard.m
#import "PlayingCard.h"
@implementation PlayingCard
- (NSString *)contents
{
  NSArray *rankStrings = [PlayingCard rankStrings];
  // 将0-13表示为?,1,2,...,J,Q,K
  return [rankStrings[self.rank] stringByAppendingString:self.suit];
}
@synthesize suit = _suit;
// 因为重构了suit的getter和setter，所以要手写@synthesize

+ (NSArray *)validSuits
{
  return @[@"♥️",@"♦️",@"♠️",@"♣️"];
}
- (void)setSuit:(NSString *)suit
{
  if([[PlayingCard validSuits] containsObject:suit]) {
    // 类的函数的调用方法。
    _suit = suit;
  }
}

- (NSString *)suit
{
  return _suit?_suit:@"?";
}

+ (NSArray *)rankStrings
{
  return @[@"?",@"A",@"2",@"3",@"4",@"5",@"6",@"7",@"8",@"9",@"10",@"J",@"Q",@"K"];
}
+ (NSUInteger)maxRank
{
  return [[self rankStrings] count]-1;
}
- (void)setRank:(NSUInteger)rank
{
  if(rank <= [PlayingCard maxRank]) {
    _rank = rank;
  }
}
@end
```

## PlayingCardDeck类

接下来可以开始玩牌了，创建一个新的类`PlayingCardDeck`，继承自`Deck`类，但是需要**重写构造函数**：

```objective-c
// PlayingCardDeck.h
#import "Deck.h"

@interface PlayingCardDeck : Deck
@end
```

```objective-c
// PlayingCardDeck.m
#import "PlayingCardDeck.h"
#import "PlayingCard.h"

@implementation PlayingCardDeck
- (instancetype)init // instancetype类只在init中使用
{
  self = [super init];
  // 调用父类的构造函数
  if(self) { 
    // 父类正常完成构造，继续子类的构造函数
    // 如果父类无法完成构造，将不执行子类的构造代码
    for (NSString *suit in [PlayingCard validSuits]) {
      for (NSUInteger rank = 1; rank <= [PlayingCard maxRank]; rank++) {
        PlayingCard *card = [[PlayingCard allo] init];
        card.rank = rank;
        card.suit = suit;
        [self addCard:card];
      }
    }
  }
  return self;
}
@end
```

## XCode的简单使用

课程剩余的部分就是以纸牌游戏为例，简单介绍XCode的使用，开发一个简单的App，App内容是显示纸牌，点击纸牌将其翻转。

由于课程介绍的是XCode 5，笔者记笔记的时候已经是XCode 12.4了，有了不小的变化，参考笔者另一篇笔记（[Objective-C IOS开发之HelloWorld](https://zhang-tianxu.github.io/chinese/2021/03/09/Objective-C-IOS%E5%BC%80%E5%8F%91%E4%B9%8BHelloWorld/)），应该也不难完成，就不再具体介绍实现了。

相较于HelloWorld这个App，课程中实现的App另外涉及了以下知识点：

- 图像添加

  直接将图片拖到`Assets.xcassets`文件夹中

- button背景图片的设置

  点击button，在属性中点击`Background`下拉菜单，就会显示上一步添加的图片选项，以及一些原始icon。

- Action中的sender其实就是触发事件的View对象

- 类的添加

  添加类的方法，XCode 12中其实就是新建`Cocoa Touch Class`。
