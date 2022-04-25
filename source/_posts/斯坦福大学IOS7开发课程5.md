---
title: 斯坦福大学IOS7开发课程5
date: 2021-04-25 13:37:05
tags:
    - IOS开发
    - Objective-C
categories:
    - 客户端开发
    - IOS开发
---
这节课主要讲解：

1. View Controller的**生命周期**
2. **NotificationCenter**的相关知识
3. 组件UITextView的简单使用

并通过Demo展示

<!--more-->

## ViewController的声明周期

ViewController 的生命周期可以看作一系列函数，ViewController是UIViewController的子类。

生命周期的开始是创建，创建后会涉及到：

1. viewDidLoad()
2. viewWillLayoutSubviews()
3. viewDidLayoutSubviews()
4. viewWillAppear()
5. viewDidAppear()
6. viewWillDisappear()
7. viewDidDisappear()
8. didReceiveMemoryWaring()

接下来**详细介绍**这几个生命周期：

## viewDidLoad()

在viewDidLoad()之前会执行创建的动作，但是我们一般不会在创建中执行操作。

创建完成之后，outlets被设置好。

viewDidLoad()只会被**调用一次**，**是用来初始化controller非常好的位置**。

需要注意的是此时view的几何结构（geometry）还没完成，**还无法确定App是运行在6寸的iPhone上还是10寸的iPad上**。所以不要在viewDidLoad()中初始化和尺寸、几何结构相关的操作。

## viewWillLayoutSubviews()

当view的几何结构发生改变时（比如屏幕从竖着变成横着），（ios7及以上）在展示之前会调用viewWillLayoutSubviews()

## viewDidLayoutSubviews()

## viewWillAppear()

view展现在屏幕之前，会调用viewWillAppear()函数。

当使用多MVC模型时，会有多个view，此时某个view的viewWillAppear()可能会被调用多次，所以只执行一次的初始化代码不要放在这里，而应该放在viewDidLoad()。

基于这个理解，有些操作需要放入viewWillAppear()执行，比如Model中的数据在view不可见时被更改了，当view再次展现的时候，需要同步Model和UI。

另外，和viewDidLoad()相对的，这里已经完成了view几何结构的初始化，因此可以执行和几何结构相关的一些操作了。



在生命周期`viewWillAppear`中，必须调用`[super viewWillAppear:animated]`，调用位置无关紧要，你可以在开始调用也可以在结束时调用。

`viewWillDisappear`也是一样。

## viewDidAppear()

view展现在屏幕上之后，调用viewDidAppear()

## viewWillDisappear()

view即将消失时调用viewWillDisappear()，此时最好停止使用非必要的资源，比如动画等等。

```objc
- (void)viewWillDisappear:(BOOL)animated
{
  [super viewWillDisappear:animated]; // 在所有viewWill/Did中都要调用super函数
  // 记住滑动的位置
  [self rememberScrollPosition];
  // 做一些清理工作
  [self saveDataToPermanentStore];
}
```

## viewDidDisappear()

view消失以后，调用viewDidDisappear()

## didReceiveMemoryWaring()

内存不够了，可能是因为你的App，**也可能是手机上的其他App造成的**。

这是你应该试图清理内存。

为什么是我要清理内存？因为操作系统有权限kill应用，如果内存不够了，而你的App又占用了很多内存，很有可能被iOS杀死。 



## awakeFromNib()

（说实话，没完全听懂，先记下来，以后用到再详细研究一下吧。）

这个方法会发送给storyboard中绘制出来的所有对象，包括controller

如果进入到Controller的init方法，就一定也会进入到`awakeFromNib`。

尽可能不要将代码放在`awakeFromNib`中。

```objc
- (void)setup {...};  // do something which can't wait until viewDidLoad
- (void)awakeFromNib { [self setup]; }
// uIViewController's designated initializer is initWithNibName:bundle:(ugh!)
- (instancetype)initWithNibName:(NSString *)name bundle:(NSBundle *)bundle
{
  // 如果使用awakeFromNib，为了保证正确性，也要加上这三行代码
  self = [super initWithNibName:name bundle:bundle];
  [self setup];
  return self;
}
```



```flow
st=>start: instantiated(实例化)
op=>operation: 调用awakeFromNib()
op2=>operation: outlets设置完成
op3=>operation: 调用viewDidLoad()
op4=>operation: 几何结构确定下来以后,\
调用viewWillLayoutSubviews()和viewDidLayoutSubviews()
op5=>operation: (对着MVC再屏幕中出现和消失，下面几个生命周期可能会被多次调用)
op6=>operation: 调用viewWillAppear()和viewDidAppear()
op7=>operation: 每当几何结构改变（比如屏幕旋转）都会调用\
viewWillLayoutSubviews()和viewDidLayoutSubviews()
op8=>operation: 如果设置了屏幕自动旋转，还会调用will/didRotatedTo/From
op9=>operation: 调用viewWillDisappear()和viewDidDisappear()
op10=>operation: 如果手机内存吃紧，调用didReceiveMemoryWarning()
e=>end
st->op->op2->op3->op4->op5->op6->op7->op8->op9->op10->e
```

<img src="https://developer.apple.com/library/archive/referencelibrary/GettingStarted/DevelopiOSAppsSwift/Art/WWVC_vclife_2x.png" style="zoom:30%;" />






## 组件UITextView简介

组件[UITextView](https://developer.apple.com/documentation/uikit/uitextview)类似于UILable，但是可以展示多行，具有可选可编辑可滚动等特点。

## NotificationCenter相关知识

接下来讲一讲NSNotification

- Notifications也就是之前在MVC中称为radio station的概念。

- NSNotificationcenter

  通过`[NSNotificationCenter defaultCenter]`获取默认Notification Center。

  如果想监听这个radio station，那么发送如下消息：

  ```objc
  - (void)addObserver:(id)observer  // 需要接收通知的对象
             selector:(SEL)methodToInvokeIfSomethingHappens
                 name:(NSString *)name  // station的名称
               object:(id)sender;  // 你感兴趣的变更，如果填写nil，标识任何人发生变更，都会通知你。指定某个sender，可以只接收指定对象发来的通知。
  ```

- 当有广播的时候，observer会被通知：

  ```objc
  - (void)methodToInvokeIfSomethingHappens:(NSNotification *)notification {
    notification.name;  // 传递的station的名称
    notification.object;  // 给你发送通知的对象
    notification.userInfo;  // 有关发生了什么的特定信息
  }
  ```

- 结束后记得关闭radio station

  ```objc
  [center removeObserver:self]; //关闭所有radio station
  // 或者
  [center removeObserver:self name:UIContentSizeCategoryDidChangeNotification object:nil];  //关闭指定radio station
  ```

  如果离开堆栈时，没有关闭radio station，notification center可能会继续发送notification，并**导致App崩溃**！

  因为NSNotificationCenter有一个指向你的unsafe retained指针。

  关闭radio station最好的位置是在MVC离开屏幕的时候，实在不行，可以在dealloc中关闭，dealloc会在离开堆栈之前调用：

  ```objc
  - (void)dealloc {
    // be careful in this method! can't access properties! you are almost gone from heap!
    [[NSNotificationCenter defaultcenter] removeObserver:self];
  }
  ```

关于NotificationCenter，一个常用的例子是字体的变化。当用户在系统设置中改变字体大小时，可以利用NotificationCenter使App中的字体也跟着变化。

```objc
NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
[center addObserver:self
           selector:@selector(preferredFontsSizeChanged:)
               name:UIContentsizeCategoryDidChangeNotification
             object:nil]; // this station's broadcasts aren't object-specific

- (void)preferredFontsSizeChanged:(NSNotification *)notification {
  // reset fonts of objects using preferred fonts
}
```


## Demo

目的：了解attributed text（富文本）、生命周期以及Notification。

1. 新建App，在storageBoard中加入：

   - TextView
   - label
   - 6 buttons

   如下图：

   ![image-20210423160322521](https://tva1.sinaimg.cn/large/008i3skNgy1gptqmdv9uwj30i40zs41r.jpg)

2. 为TextView和Label添加property:

   ```objc
   @interface ViewController()
   @property (weak, nonatomic) IBOutlet UITextView *body;
   @property (weak, nonatomic) IBOutlet UILabel *headline;
   @end
   ```

3. 为buttons添加三个函数

   - 四个色块的button共用一个函数，函数的作用是：在textView中选中一些文字，点击色块button后，选中的文字变成色块对应的颜色

     ```objc
     - (IBAction)changeBodySelectionColorToMatchBackgroundOfButton:(UIButton *)sender {
         [self.body.textStorage addAttribute:NSForegroundColorAttributeName value:sender.backgroundColor range:self.body.selectedRange];
     }
     ```

   - outline按钮对应的函数，作用是给选中文字添加红色描边。

     ```objc
     - (IBAction)outlineBodySelection:(id)sender {
         [self.body.textStorage addAttributes:@{ NSStrokeWidthAttributeName : @-3, NSStrokeColorAttributeName: [UIColor redColor] } range:self.body.selectedRange];
     }
     ```

     添加描边需要添加两个属性，一个用来设置描边的宽度，一个用来设置描边的颜色。

     `NSStrokeWidthAttributeName`用来设置描边的宽度，设置为整数会让字体变成中空，设置为负数会保持字体填充。

     `NSStrokeColorAttributeName`用来设置描边的颜色。

   - unOutline按钮对应的按钮，作用是删除字体的描边。

     ```objc
     - (IBAction)unOutlineBodySelection:(id)sender {
         [self.body.textStorage removeAttribute:NSStrokeWidthAttributeName range:self.body.selectedRange];
     }
     ```

     直接利用`removeAttribute`删除`NSStrokeWidthAttributeName`属性，颜色就不用管了，没有宽度颜色也就无法显示了。
   
4. 设置outline按钮的字体描边

   这样产不多就是这节课要介绍的关于富文本的知识点了。接下来加入一些生命周期的元素。

   现在想要让outline按钮的文字变成描边的，我们可不想在设置一个按钮来让outline按钮outline，而是想一开始它就是描边的。

   这个时候就用到生命周期了，可以在`viewDidLoad()`函数中实现：

   先为button添加outlet，`@property (weak, nonatomic) IBOutlet UIButton *outline;`，然后在`viewDidLoad()`函数中编写代码：

   ```objc
   - (void)viewDidLoad {
       [super viewDidLoad];
       // Do any additional setup after loading the view.
       NSMutableAttributedString *title = [[NSMutableAttributedString alloc] initWithString:self.outline.currentTitle];
       [title setAttributes:@ {
           NSStrokeWidthAttributeName : @3,
           NSStrokeColorAttributeName : self.outline.tintColor}
                      range:NSMakeRange(0, [title length])];
       [self.outline setAttributedTitle:title forState:UIControlStateNormal];
   }
   ```
   
5. 利用NSNotificationCenter让App中字体跟随系统字体，用户改变系统字体时，App中字体也随之变动。

   1. 首先实现一个将字体设置为系统对应字体的方法：

      ```objc
      - (void)usePreferredFonts {
          self.body.font = [UIFont preferredFontForTextStyle:UIFontTextStyleBody];
          self.headline.font = [UIFont preferredFontForTextStyle:UIFontTextStyleHeadline];
      }
      ```

      并将其包装一层：

      ```objc
      - (void)preferredFontsChanged:(NSNotification *)notification {
          [self usePreferredFonts];
      }
      ```

   2. 在生命周期`viewWillAppear()`中添加NotificationCenter

      ```objc
      - (void)viewWillAppear:(BOOL)animated {
          [super viewWillAppear:animated];
          [[NSNotificationCenter defaultCenter] addObserver:self
                     selector:@selector(preferredFontsChanged:)
                         name:UIContentSizeCategoryDidChangeNotification
                       object:nil];
      }
      ```

   3. 然后记得关闭radio station

      ```objc
      - (void)viewWillDisappear:(BOOL)animated {
          [super viewWillDisappear:animated];
          // [[NSNotificationCenter defaultCenter] removeObserver:self]; // 这种方法也可以，删除所有的notification，但不够优雅，尽量使用下面的方法，指定具体的notificationCenter
          [[NSNotificationCenter defaultCenter] removeObserver:self
                                                          name:UIContentSizeCategoryDidChangeNotification
                                                        object:nil];
      }
      ```

   4. 最后要考虑一种情况，NotificationCenter在view消失时被关闭，如果在这之后用户修改了系统的字体，那么view再次appear时，App字体会和系统不一致。因此，需要在`viewWillAppear`中，调用`[self usePreferredFonts]`来同步外部世界的字体。


