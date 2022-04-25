---
title: Objective-C IOS开发之SQLite数据库
date: 2021-03-10 21:45:33
tags:
    - Objective-C
    - IOS开发
categories:
  - 客户端开发
  - IOS开发
---

IDE: XCode Version 12.4 (12D4e)

参考[IOS开发教程](https://www.tutorialspoint.com/ios/ios_sqlite_database.htm)
## 操作流程

<!--more-->

1. Xcode创建一个APP

2. 选中项目文件，选中TARGETS，然后在框架（frameworks, Libraries, and Embedded Content）中添加**libsqlite3.tbd**（libsqlite3.0.tbd也一样）
{% img addSQLiteLibImg https://tva1.sinaimg.cn/large/008eGmZEgy1gof54maa10j314p0u0af4.jpg 500 600 '"添加SQLite库" "加载失败"' %}
{% img addSQLiteLibImg2 https://tva1.sinaimg.cn/large/008eGmZEgy1gof54tkttyj30ng0qon0f.jpg 500 300 '"添加SQLite库2" "加载失败"' %}

3. 新建Objective-C类（File->New->File）选择Cocoa Touch Class，点击Next，Subclass of选择NSObject，language选Objective-C。
{% img addObjcClassImg  https://tva1.sinaimg.cn/large/008eGmZEgy1gof56z2pxuj30u00xm4ee.jpg 800 300 '"添加Objective-C类" "加载失败"' %}
{% img addObjcClassImg2  https://tva1.sinaimg.cn/large/008eGmZEgy1gof57eshyqj315i0te0y6.jpg 400 300 '"添加Objective-C类2" "加载失败"' %}

4. 将类的名字命名为DBManager，点击Next创建
{% img addDBManager https://tva1.sinaimg.cn/large/008eGmZEgy1gof57v7yg2j31600u0gpo.jpg 400 300 '"创建DBManager" "加载失败"' %}

5. 项目中会增加`DBManager.h`和`DBManager.m`两个文件，其代码是：

   ```objective-c
   //DBManager.h
   
   #import <Foundation/Foundation.h>
   #import <sqlite3.h>
   
   NS_ASSUME_NONNULL_BEGIN
   
   @interface DBManager : NSObject {
       NSString *databasePath;
   }
   +(DBManager*)getSharedInstance;
   -(BOOL)createDB;
   -(BOOL) saveData:(NSString*)registerNumber name:(NSString*)name department:(NSString*)department year:(NSString*)year;
   -(NSArray*) findByRegisterNumber:(NSString*)registerNumber;
   
   @end
   
   NS_ASSUME_NONNULL_END
   
   ```

   ```objective-c
   //DBManager.m
   
   #import "DBManager.h"
   
   static DBManager *sharedInstance = nil;
   static sqlite3 *database = nil;
   static sqlite3_stmt *statement= nil;
   
   @implementation DBManager
   
   +(DBManager*)getSharedInstance{
       if(!sharedInstance) {
           sharedInstance = [[super allocWithZone:NULL]init];
           [sharedInstance createDB];
       }
       return sharedInstance;
   }
   
   -(BOOL)createDB {
       NSString *docsDir;
       NSArray *dirPaths;
       // get the documents directory
       dirPaths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
       docsDir = dirPaths[0];
       // build the path to the database file
       databasePath = [[NSString alloc] initWithString:[docsDir stringByAppendingPathComponent:@"student.db"]];
       BOOL isSuccess = YES;
       NSFileManager* filemgr = [NSFileManager defaultManager];
       
       if([filemgr fileExistsAtPath:databasePath] != NO) {
           const char *dbpath = [databasePath UTF8String];
           if(sqlite3_open(dbpath, &database) == SQLITE_OK) {
               char *errMsg;
               const char *sql_stmt =
               "create table if not exists studentsDetail (regno integer primary key, name text, department text, year text)";
               if(sqlite3_exec(database, sql_stmt, NULL, NULL, &errMsg) != SQLITE_OK) {
                   isSuccess = NO;
                   NSLog(@"Fail to create table");
               }
               sqlite3_close(database);
               return isSuccess;
           } else {
               isSuccess = NO;
               NSLog(@"Fail to open/create database");
           }
       }
       return isSuccess;
   }
   
   -(BOOL)saveData:(NSString *)registerNumber name:(NSString *)name department:(NSString *)department year:(NSString *)year; {
       const char *dbpath = [databasePath UTF8String];
       
       if(sqlite3_open(dbpath, &database) == SQLITE_OK) {
           NSString *insertSQL = [NSString stringWithFormat:@"insert into studentsDetail (regno, name, department, year) values (\"%ld\",\"%@\",\"%@\",\"%@\")",(long)[registerNumber integerValue], name, department, year];
           const char *insert_stmt = [insertSQL UTF8String];
           if(sqlite3_prepare_v2(database, insert_stmt, -1, &statement, NULL) != SQLITE_OK) {
               NSLog(@"Prepare failure:%s",sqlite3_errmsg(database));
           }
           if(sqlite3_step(statement) == SQLITE_DONE) {
               return YES;
           } else {
               return NO;
           }
           
       }
       sqlite3_reset(statement);
       return NO;
   }
   
   -(NSArray*)findByRegisterNumber:(NSString *)registerNumber{
       const char *dbpath = [databasePath UTF8String];
       if(sqlite3_open(dbpath, &database) == SQLITE_OK) {
           NSString *querySQL = [NSString stringWithFormat:@"select name, department, year from studentsDetail where regno = \"%@\"",registerNumber];
           const char *query_stmt = [querySQL UTF8String];
           NSMutableArray *resultArray = [[NSMutableArray alloc] init];
           if(sqlite3_prepare_v2(database, query_stmt, -1, &statement, NULL) == SQLITE_OK) {
               if(sqlite3_step(statement) == SQLITE_ROW) {
                   NSString *name = [[NSString alloc] initWithUTF8String:(const char *)sqlite3_column_text(statement, 0)];
                   [resultArray addObject:name];
                   
                   NSString *department = [[NSString alloc] initWithUTF8String:(const char*)sqlite3_column_text(statement, 1)];
                   [resultArray addObject:department];
                   
                   NSString *year = [[NSString alloc] initWithUTF8String:(const char*)sqlite3_column_text(statement, 2)];
                   [resultArray addObject:year];
                   return resultArray;
               } else {
                   NSLog(@"Not found");
                   return nil;
               }
               sqlite3_reset(statement);
           }
       }
       return nil;
   }
   @end
   
   ```

6. 在`Main.storyboard`中添加如下组件：
{% img result https://tva1.sinaimg.cn/large/008eGmZEgy1gof3hvfqacj30ga0gsgma.jpg 800 300 '"效果" "加载失败"' %}

7. 为所有的输入框新建`Referencing Outlet`

8. 为两个button创建`Touch Up Inside`事件

9. 创建完之后，`ViewController.h`自动被填写为:

      ```objective-c
      #import <UIKit/UIKit.h>
      #import "DBManager.h"//这个是手动添加的
      
      @interface ViewController : UIViewController
      @property (weak, nonatomic) IBOutlet UITextField *findByRegisterNumberTextField;
      @property (weak, nonatomic) IBOutlet UITextField *regNoTextField;
      @property (weak, nonatomic) IBOutlet UITextField *nameTextField;
      @property (weak, nonatomic) IBOutlet UITextField *departmentTextField;
      @property (weak, nonatomic) IBOutlet UITextField *yearTextField;
      @property (weak, nonatomic) IBOutlet UIScrollView *myScrollView;
      
      - (IBAction)findData:(id)sender;
      @property (weak, nonatomic) IBOutlet UIButton *saveData;
      
      
      @end
      ```

10. 在`ViewController.m`中实现`findData`和`saveData`

      ```objective-c
      //  ViewController.m
      
      #import "ViewController.h"
      
      @interface ViewController ()
      
      @end
      
      @implementation ViewController
      
      - (void)viewDidLoad {
          [super viewDidLoad];
          // Do any additional setup after loading the view.
      }
      
      - (IBAction)saveData:(id)sender {
          BOOL success = NO;
          NSString *alertString = @"Data Insertion failed";
          if(_regNoTextField.text.length > 0 && _nameTextField.text.length > 0 && _departmentTextField.text.length > 0 && _yearTextField.text.length > 0) {
              success = [[DBManager getSharedInstance] saveData:_regNoTextField.text name:_nameTextField.text department:_departmentTextField.text year:_yearTextField.text];
          } else {
              alertString = @"Enter all fields";
          }
          if(success == NO) {
              UIAlertController *alert = [UIAlertController alertControllerWithTitle:alertString message:nil preferredStyle:UIAlertControllerStyleAlert];
              UIAlertAction *okAction = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
                  NSLog(@"点击了OK");
              }];
              [alert addAction:okAction];
              [self presentViewController:alert animated:YES completion:nil];
          }
      }
      
      - (IBAction)findData:(id)sender {
          NSArray *data = [[DBManager getSharedInstance] findByRegisterNumber:_findByRegisterNumberTextField.text];
          if(data == nil) {
              UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"Data not found" message:nil preferredStyle:UIAlertControllerStyleAlert];
              UIAlertAction *okAction = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
              }];
              [alert addAction:okAction];
              [self presentViewController:alert animated:YES completion:nil];
              _regNoTextField.text = @"";
              _nameTextField.text =@"";
              _departmentTextField.text = @"";
              _yearTextField.text =@"";
          } else {
              _regNoTextField.text = _findByRegisterNumberTextField.text;
              _nameTextField.text =[data objectAtIndex:0];
              _departmentTextField.text =[data objectAtIndex:1];;
              _yearTextField.text =[data objectAtIndex:2];;
          }
      }
      
      #pragma mart - Text field delegate
      -(void)textFieldDidBeginEditing:(UITextField *)textField {
          [_myScrollView setFrame:CGRectMake(10,50,300,200)];
          [_myScrollView setContentSize:CGSizeMake(300, 350)];
      }
      -(void)textFieldDidEndEditing:(UITextField *)textField {
          [_myScrollView setFrame:CGRectMake(10, 50, 300, 350)];
      }
      -(BOOL) textFieldShouldReturn:(UITextField *)textField {
          [textField resignFirstResponder];
          return YES;
      }
      @end
      ```

      

至此，可以通过save按钮保存数据，find按钮可以查找数据，并显示在各自输入框中

## 知识点

1. `libsqlite3.dylib`vs`libsqlite3.tbd`vs`libsqlite3.0.tbd`

   `.tbd` 在Xcode7后替代了`.dylib` 。而`libsqlite3.tbd`只是`libsqlite3.0.tbd`的链接，也就是两者是一摸一样的，引入任意一个的效果都是一样的。

2. sqlite3的使用

   1. 创建数据库

      SQLite的数据库就是一个文件，创建数据库也就是创建一个文件。

   2. 打开数据库

      打开数据库其实就是用`sqlite3_open`函数打开一个文件。

   3. 执行SQL命令

      SQLite中执行SQL命令有两种方式：

      - `sqlite3_exec()`
      - `sqlite3_prepare_v2()`+`sqlite3_step()`

      两者的区别可以看[stackoverflow](https://stackoverflow.com/questions/27383724/sqlite3-prepare-v2-sqlite3-exec)。

3. 弹出框的使用

   ```objective-c
   UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"警告的Title" message:@"警告的消息"
                              preferredStyle:UIAlertControllerStyleAlert];
   
   UIAlertAction *okAction = [UIAlertAction actionWithTitle:@"OK"
                              style:UIAlertActionStyleDefault
                              handler:^(UIAlertAction * _Nonnull action) {
                                NSLog(@"点击了OK");
                              }];
   UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"Cancel"
                                  style:UIAlertActionStyleCancel
                                  handler:^(UIAlertAction * _Nonnull action) {
                                    NSLog(@"点击了Cancel");
                                  }];
   [alert addAction:okAction];
   [alertController addAction:cancelAction];
   
   [self presentViewController:alert animated:YES completion:nil];
   ```

   
