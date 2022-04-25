---
title: 斯坦福大学IOS7开发课程3
date: 2021-03-22 15:04:47
tags:
    - IOS开发
    - Objective-C
categories:
    - 客户端开发
    - IOS开发
---
# 斯坦福大学IOS7开发课程3

课程3主要内容是继续之前的纸牌游戏。上节课实现的纸牌游戏App只是实现了简单的纸牌翻转，这节课使其变得真正可玩。

主要分为三个部分：

1. 先回顾上节课编写的App
2. 对上节课的App改进
3. 使游戏变得可玩

<!--more-->

## 上节课实现的纸牌翻转App

1. **将背景颜色设置为绿色**

   打开`Main.storyboard`，选中View后，在属性栏里将`Background`的颜色设置为绿色。

{% img setBackgroundColor https://tva1.sinaimg.cn/large/008eGmZEly1goo58d9362j30ee0oe773.jpg 300 700 '"设置Background颜色" "加载失败"' %}

2. **添加纸牌正反面的图片**

   在网上获取纸牌正反面图片，比如：

{% img CardBack https://tva1.sinaimg.cn/large/008eGmZEly1goo59iuj4rj30dw0jgqdw.jpg 100 250 '"纸牌背面" "加载失败"' %}

{% img CardFront https://tva1.sinaimg.cn/large/008eGmZEly1goo59sknnzj30dw0jg74b.jpg 100 250 '"纸牌正面" "加载失败"' %}


   选中`Assets.xcassets`，将两张图片拖至其中。
{% img addImage https://tva1.sinaimg.cn/large/008eGmZEly1goo5bh0htpj31e00d4adb.jpg 600 200 '"添加图片到Assets.xcassets" "加载失败"' %}

3. **添加按钮，并将按钮的背景图片改为纸牌正面图片（CardFront）， 文字改为“♠️A”**

{% img setButtonBackground https://tva1.sinaimg.cn/large/008eGmZEly1goo5fk47bsj31sg0r4wpk.jpg 600 300 '"设置按钮背景图片" "加载失败"' %}

4. **添加一个`Label`，显示纸牌翻转的次数**

5. **为按钮添加“Touch Up Inside”事件，命名为`touchCardButton`**

6. **为`Label`在`ViewController.m`的`@interface ViewController() @end`之间添加“Referencing Outlets“，命名为`flipsLabel`。**

{% img addOutlets https://tva1.sinaimg.cn/large/008eGmZEly1gooawynczug30v00ienpd.gif  600 300 '"为Label添加Outlet" "加载失败"' %}

7. **另外，在`ViewController.m`的`@interface ViewController() @end`之间添加int属性`flipCount`**

8. **在`ViewController.m`中实现touchCardButton事件函数**

   ```objective-c
   - (IBAction)touchCardButton:(UIButton *)sender {
       if ([sender.currentTitle length]) {
           [sender setBackgroundImage:[UIImage imageNamed:@"CardBack"] forState:UIControlStateNormal];
           [sender setTitle:@"" forState:UIControlStateNormal];
       } else {
           Card *card = [self.deck drawRandomCard];
           if(card) {
           [sender setBackgroundImage:[UIImage imageNamed:@"CardFront"] forState:UIControlStateNormal];
           [sender setTitle:@"A♣️" forState:UIControlStateNormal];
           }
       }
       self.flipCount++;
   }
   ```

9. **重构属性`flipCount`的`setter`函数。**

   ```objective-c
   - (void)setFlipCount:(int)flipCount
   {
       _flipCount = flipCount;
       self.flipsLabel.text = [NSString stringWithFormat:@"Flips: %d",self.flipCount];
   }
   ```

**此时App完成，代码如下：**

```objective-c
// ViewController.h

#import <UIKit/UIKit.h>

@interface ViewController : UIViewController
- (IBAction)touchCardButton:(UIButton *)sender;

@end
```

```objective-c
// ViewController.m
#import "ViewController.h"

@interface ViewController ()
@property (weak, nonatomic) IBOutlet UILabel *flipsLabel;
@property (nonatomic) int flipCount;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
}

- (IBAction)touchCardButton:(UIButton *)sender {
    if ([sender.currentTitle length]) {
        [sender setBackgroundImage:[UIImage imageNamed:@"CardBack"] forState:UIControlStateNormal];
        [sender setTitle:@"" forState:UIControlStateNormal];
    } else {
        [sender setBackgroundImage:[UIImage imageNamed:@"CardFront"] forState:UIControlStateNormal];
        [sender setTitle:@"A♣️" forState:UIControlStateNormal];
    }
    self.flipCount++;
}

- (void)setFlipCount:(int)flipCount
{
    _flipCount = flipCount;
    self.flipsLabel.text = [NSString stringWithFormat:@"Flips: %d",self.flipCount];
}
@end
```

最终效果如下：

{% img result1 https://tva1.sinaimg.cn/large/008eGmZEly1goob73x6nkg30j816ykjm.gif  300 700 '"最终效果1" "加载失败"' %}



## 对该App改进

上节课完成的App只能实现纸牌的翻转，每次翻转显示的纸牌都是同一张。那么如何让每次翻转，随机显示牌堆中的一张纸牌呢？其实也很简单，方法如下：

1. **在`ViewController.m`中引入`Deck.h`**

2. **在`ViewController.m`的`@interface ViewController() @end`之间添加`Deck*`属性`deck`**

3. **在`ViewController.m`中引入`PlayingCardDeck.h`**

4. **修改属性`deck`的`getter`，当`_deck`为空时，通过`[[PlayingCardDeck alloc]init]`创建牌堆，然后再返回Deck。**

   此时`ViewController.m`的代码如下：

   ```objective-c
   //  ViewController.m
   
   #import "ViewController.h"
   #import "Deck.h"
   #import "PlayingCardDeck.h"
   
   @interface ViewController ()
   @property (weak, nonatomic) IBOutlet UILabel *flipsLabel;
   @property (nonatomic) int flipCount;
   @property (strong, nonatomic) Deck *deck;
   @end
   
   @implementation ViewController
   
   - (void)viewDidLoad {
       [super viewDidLoad];
       // Do any additional setup after loading the view.
   }
   
   - (Deck *)deck
   {
       if(!_deck)
           _deck = [self createDeck];
       return _deck;
   }
   
   - (Deck *)createDeck
   {
       return [[PlayingCardDeck alloc]init];
   }
   
   
   - (IBAction)touchCardButton:(UIButton *)sender {
       if ([sender.currentTitle length]) {
           [sender setBackgroundImage:[UIImage imageNamed:@"CardBack"] forState:UIControlStateNormal];
           [sender setTitle:@"" forState:UIControlStateNormal];
       } else {
           [sender setBackgroundImage:[UIImage imageNamed:@"CardFront"] forState:UIControlStateNormal];
           [sender setTitle:@"A♣️" forState:UIControlStateNormal];
       }
       self.flipCount++;
   }
   
   - (void)setFlipCount:(int)flipCount
   {
       _flipCount = flipCount;
       self.flipsLabel.text = [NSString stringWithFormat:@"Flips: %d",self.flipCount];
   }
   @end
   ```

5. **在`touchCardButton`函数中，利用`deck`的`drawRandomCard`函数随机抽取一张纸牌。**

6. **将随机抽取出的Card的内容显示出来。需要注意的是，牌堆里的牌抽完以后要停止抽牌。**

完成后`ViewController.m`的代码如下：

```objective-c
// ViewController.m

#import "ViewController.h"
#import "Deck.h"
#import "PlayingCardDeck.h"

@interface ViewController ()
@property (weak, nonatomic) IBOutlet UILabel *flipsLabel;
@property (nonatomic) int flipCount;
@property (strong, nonatomic) Deck *deck;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
}

- (Deck *)deck
{
    if(!_deck)
        _deck = [self createDeck];
    return _deck;
}

- (Deck *)createDeck
{
    return [[PlayingCardDeck alloc]init];
}

- (IBAction)touchCardButton:(UIButton *)sender {
    if ([sender.currentTitle length]) {
        [sender setBackgroundImage:[UIImage imageNamed:@"CardBack"] forState:UIControlStateNormal];
        [sender setTitle:@"" forState:UIControlStateNormal];
        self.flipCount++;
    } else {
        Card *card = [self.deck drawRandomCard]; // 从牌堆抽随机取一张Card
        if(card) { // 牌堆里的牌抽完以后停止抽牌
            [sender setBackgroundImage:[UIImage imageNamed:@"CardFront"] forState:UIControlStateNormal];
            [sender setTitle:card.contents forState:UIControlStateNormal]; // 将Card的内容展示出来
            self.flipCount++;
        }
    }
}

- (void)setFlipCount:(int)flipCount
{
    _flipCount = flipCount;
    self.flipsLabel.text = [NSString stringWithFormat:@"Flips: %d",self.flipCount];
}
@end
```

效果如下：

{% img result2 https://tva1.sinaimg.cn/large/008eGmZEly1goobso7hl7g30j816yx6r.gif  300 700 '"最终效果2" "加载失败"' %}



## 使该游戏变得真正可玩

接下来我们让游戏变得真正可玩儿。在MVC开发模型中，游戏的逻辑属于Model，Model和UI是完全独立的，所以在编写过程中不需要考虑UI。

### Model开发

首先添加一个新的类`CardMatchingGame`，这个类就是MVC模型中的Model，在编码`CardMathingGame`的过程中，需要思考需要哪些公开的API。

1. 首先是构造函数，因为需要传进一些参数，比如牌的张数，所以新建构造函数`initWithCardCount`，指定构造函数（designated initializer）
2. 公开的属性 score，但同时我们不希望别人能够随意修改socre属性，因此需要在公开API中将其设置为readonly，同时在.m文件中的私密API中声明为readwrite。其实readwrite用的不多，因为默认情况下就是readwrite，只有在公开API只读的时候才会用到。
3. 允许用户通过index选中纸牌`chooseCardAtIndex`
4. 通过index从牌堆中获取Card，`CardAtIndex`

```objective-c
// CardMatchingGame.h

#import <Foundation/Foundation.h>
#import "Deck.h"

NS_ASSUME_NONNULL_BEGIN

@interface CardMatchingGame : NSObject

// designated initializer
- (instancetype)initWithCardCount:(NSUInteger)count usingDeck:(Deck *)deck;
- (void)chooseCardAtIndex:(NSUInteger)index;
- (Card *)cardAtIndex:(NSUInteger)index;
@property (nonatomic, readonly) NSInteger score;
@end

NS_ASSUME_NONNULL_END
```

接下来开始在`CardMatchingGame.m`中开始实现:

1. 利用关键字`readwrite`使得属性score在`CardMatchingGame.m`中可写。

   ```objc
   @interface CardMatchingGame()
   @property (nonatomic, readwrite) NSInteger score;
   @end
   ```

2. 新建属`cards`，用于存储游戏中的纸牌。

   ```objc
   @interface CardMatchingGame()
   @property (nonatomic, readwrite) NSInteger score;
   @property (nonatomic, strong) NSMutableArray *cards; // of Card
   @end
   ```

3. 修改属性`cards`的`getter`函数

   属性的默认初始值为nil，当属性`cards`的指针为nil时，创建一个`NSMutableArray`

   ```objective-c
   -(NSMutableArray *)cards
   {
       if(!_cards) {
           _cards = [[NSMutableArray alloc] init];
       }
       return _cards;
   }
   ```

4. 实现构造函数`initWithCardCount`

   这个构造函数是designated initializer，也就是说使用这个类的用户必须调用这个构造函数，否则类无法被正确的初始化。使用构造函数init会返回nil。这个信息要传递给用户，所以以注释的形式写在公开API中

   ```objective-c
   -(instancetype) initWithCardCount:(NSUInteger)count usingDeck:(Deck *)deck
   {
       self = [super init];
       if(self) {
           for(int i = 0;i < count;i++) {
               Card *card = [deck drawRandomCard];
               if(card) {
                   [self.cards addObject:card];
               } else {
                   self = nil;
                   break;
               }
           }
       }
       return self;
   }
   ```

5. 实现函数`cardAtIndex`。

   ```objective-c
   - (Card *)cardAtIndex:(NSUInteger)index
   {
       return index < [self.cards count] ? self.cards[index] : nil;
   }
   ```

6. 实现函数`chooseCardAtIndex`

   这个函数是游戏的关键。表示某个card被选中后的处理方式，包括匹配过程

   ```objective-c
   static const int PENALTY_SCORE = 1;
   static const int COST_OF_CHOOSE = 1;
   
   -(void)chooseCardAtIndex:(NSUInteger)index
   {
       Card *card = [self cardAtIndex:index];
       if(!card.isMatched) {
           // if choosen card is not matched
           if(card.isChosen) { // if the card is already chosen
               card.chosen = NO; // filp the card back
           } else { // match it against another card
               // here, only match two cards, but may make it match multiple cards in later
               for(Card *otherCard in self.cards) {
                   if(otherCard.isChosen && !otherCard.isMatched) { // Bingo, find the card we want to match
                       int matchScore = [card match:@[otherCard]]; // try to match it
                       if(matchScore == 0) { // not match
                           self.score -= PENALTY_SCORE; // penalty of un match
                           otherCard.chosen = NO; // flip the other card back
                       } else { // match
                           self.score += matchScore;
                           card.matched = YES;
                           otherCard.matched = YES;
                       }
                       break;
                   }
               }
               card.chosen = YES;
               self.score -= COST_OF_CHOOSE;
           }
       } else {
           // if choosen card is already matched
           // do nothing
       }
   }
   ```

### UI开发

到目前，完成了App中Model的实现，接下来是实现App的UI。

1. 在UI中创建多张纸牌，可以通过复制粘贴的方式完成

2. 创建OutletCollections，添加所有纸牌。

   创建方法，右键点击某个button，按住“New Referencing Outlet Collections“，拖到`ViewController.m`中。

   添加其他button的方法，按住Ctrl键，点击button并拖到`ViewController.m`中

{% img addOutletCollections https://tva1.sinaimg.cn/large/008eGmZEly1gopcewkpqrg30n80v41l3.gif  300 700 '"添加OutletCollections" "加载失败"' %}

3. 在`ViewController.m`中引入`CardMatchingGame.h`

4. 新增`CardMatchingGame`的属性`game`

   `@property (strong, nonatomic) CardMatchingGame *game;`

5. 重写属性`game`的`getter`

   ```objc
   - (CardMatchingGame *)game{
       if(!_game) {
           _game = [[CardMatchingGame alloc] initWithCardCount:[self.cardButtons count] usingDeck:[self createDeck]];
       }
       return _game;
   }
   ```
   
6. 在`ViewController.m`中创建函数`titleForCard`

   ```objec
   - (NSString *)titleForCard:(Card *)card
   {
       return card.isChosen ? card.contents : @"";
   }
   ```

7. 在`ViewController.m`中创建函数`backgroundImageForCard`

   ```objc
   - (UIImage *)backgroundImageForCard:(Card *)card
   {
       return [UIImage imageNamed:card.isChosen ? @"CardFront" : @"CardBack"]; // CardFront & CardBack are name of card image
   }
   ```

8. 在`ViewController.m`中创建函数`updateUI`用于同步Model和UI。

   ```objc
   - (void)updateUI
   {
       for (UIButton *cardButton in self.cardButtons) {
           int cardIndex = [self.cardButtons indexOfObject:cardButton];
           Card *card = [self.game cardAtIndex:cardIndex];
           [cardButton setTitle:[self titleForCard:card] forState:UIControlStateNormal];
           [cardButton setBackgroundImage:[self backgroundImageForCard:card] forState:UIControlStateNormal];
           cardButton.enabled = !card.isMatched;
       }
   }
   ```

9. 创建Touch Up Inside事件，命名为touchCardButton，并将所有button加入其中，方法类似于步骤2.

10. `.m`文件中实现`touchCardButton`

    ````objc
    - (IBAction)touchCardButton:(UIButton *)sender {
        int cardIndex = [self.cardButtons indexOfObject:sender];
        [self.game chooseCardAtIndex:cardIndex];
        [self updateUI];
    }
    ````

到这个时候App已经可以运行了，运行效果如下：

{% img result3 https://tva1.sinaimg.cn/large/008eGmZEly1gosmu8s2gmg30j01261kx.gif  300 700 '"最终效果3" "加载失败"' %}

可以正常运行，但是两张黑桃却没有正常匹配，问题出在哪里？

问题在我们在[课程2](https://zhang-tianxu.github.io/chinese/2021/03/15/%E6%96%AF%E5%9D%A6%E7%A6%8F%E5%A4%A7%E5%AD%A6IOS7%E5%BC%80%E5%8F%91%E8%AF%BE%E7%A8%8B2/)中实现的`Card`类中的匹配函数：

```objective-c
// Card.m
- (int)match:(NSArray *)otherCards
{
  int score = 0;
  for(Card *card in otherCards) {
    if([card.contents isEqualToString:self.contents])
      score = 1;
  }
    return score;
}
```

这显然不是一个完整的匹配逻辑。

所以需要在`PlayingCard`类的`.m`文件中重构`match`函数。

```objc
- (int)match:(NSArray *)otherCards
{
    int score = 0;
    if([otherCards count] == 1) {
        PlayingCard *otherCard = [otherCards firstObject];
        if([self.suit isEqualToString:otherCard.suit]) { // 如果花色匹配
            score = 1;
        } else if ([self.rank == otherCard.rank]) { // 如果数字匹配
            score = 4;
        }
    }
    return score;
}
```

现在完成了匹配规则，最后一个任务是添加一个`Label`用于展示分数。

1. 在UI中增加一个Label

2. 为Label增加一个Outlet

3. 在`updateUI`中更新score Label。

   ```objective-c
   - (void)updateUI
   {
       for (UIButton *cardButton in self.cardButtons) {
           int cardIndex = [self.cardButtons indexOfObject:cardButton];
           Card *card = [self.game cardAtIndex:cardIndex];
           [cardButton setTitle:[self titleForCard:card] forState:UIControlStateNormal];
           [cardButton setBackgroundImage:[self backgroundImageForCard:card] forState:UIControlStateNormal];
           cardButton.enabled = !card.isMatched;
       }
       self.scoreLabel.text = [NSString stringWithFormat:@"Score: %d", self.game.score];
   }
   ```

   

{% img result4 https://tva1.sinaimg.cn/large/008eGmZEly1gosov9rkvfg30j0126kjl.gif  300 700 '"最终效果4" "加载失败"' %}

至此，一个可玩的纸牌匹配游戏App已经完成，大家可以试着让App可以同时匹配三张及以上的纸牌。
