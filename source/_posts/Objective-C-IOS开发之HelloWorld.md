---
title: Objective-C IOS开发之HelloWorld
date: 2021-03-9 17:15:11
tags:
    - Objective-C
    - IOS开发入门
categories:
    - 客户端开发
    - IOS开发
    - Objective-C
---
IDE: XCode Version 12.4 (12D4e)
<!--more-->
1. 新建App

![image-20210311120542111](https://tva1.sinaimg.cn/large/008eGmZEgy1gofu3u9dcoj314y0ten0w.jpg)



2. 给项目命名为`HelloWorld`，Interface选择`Storyboard`，Language选择`Objective-C`。

![image-20210311120644241](https://tva1.sinaimg.cn/large/008eGmZEgy1gofu4wmsj2j314u0t6424.jpg)

3. 打开文件`Main.storyboard`，添加一个`Label`和`Button`组件。
   ![](https://tva1.sinaimg.cn/large/008eGmZEgy1gofubs0rgnj31iw0u0e02.jpg)

![image-20210311121521358](https://tva1.sinaimg.cn/large/008eGmZEgy1gofudvnpftj312u0u0tcs.jpg)

4. 打开两个面板，一个显示`Main.storyboard`，另一个显示`ViewController.h`
   ![](https://tva1.sinaimg.cn/large/008eGmZEgy1gofugj8ui9j31iq0fsdk5.jpg)
   
   

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gofuiw4hgvj31hx0u07i1.jpg)

5. 为Label新建`New Referencing Outlet`（右键点击Label，点击`New Referencing Outlet`后的点不松开，拖到`ViewController.h`中。将新的`Referencing Outlet`命名为`helloLabel`。
   ![](https://tva1.sinaimg.cn/large/008eGmZEgy1gofukdlay3j31gt0u0h9a.jpg)

6. 为Button添加`Touch Up Inside`事件，将事件命名为`showHelloWorld`。
   ![](https://tva1.sinaimg.cn/large/008eGmZEgy1gog2epczyxj30wg0u0h1n.jpg)
   这时，`ViewController.h`的代码变为：
   
   ```objective-c
   //  ViewController.h
   
   #import <UIKit/UIKit.h>
   
   @interface ViewController : UIViewController
   
   @property (weak, nonatomic) IBOutlet UILabel *helloLabel;
   - (IBAction)showHelloWorld:(id)sender;
   
   @end
   ```
   同时，`ViewController.m`中也自动添加了`- (IBAction)showHelloWorld(id)sender {}`函数，在其中添加`_helloLabel.text = @"Hello World";`

至此，最简单的IOS App开发完成。



