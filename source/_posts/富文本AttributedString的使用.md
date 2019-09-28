---
title: 富文本AttributedString的使用
date: 2019-09-26 11:44:47
tags:
   - macos富文本
   - AttributedString
---

### 介绍
AttributedString可以分为NSAttributedString和NSMutableAttributedString两种。在使用中通过将AttributedString赋值给控件的 attributedText 属性来添加文字样式

### 使用方式

#### 使用方式一

1. 添加超链接、设置颜色
```object-c
NSString *string = @"欢迎使用,这是测试哦";
NSMutableAttributedString *colorTitle = [[NSMutableAttributedString alloc] initWithString:string];

//设置超链接
[colorTitle addAttributes:@{
                                NSUnderlineStyleAttributeName:@1,
                                NSLinkAttributeName: [NSURL URLWithString:@"https://baidu.com"],
                                } range:NSMakeRange(string.length-9, 4)];

//设置字体大小
[colorTitle addAttribute:NSFontAttributeName value:[NSFont systemFontOfSize:13] range:NSMakeRange(0, [string length])];

//设置文字颜色
[colorTitle addAttribute:NSForegroundColorAttributeName value:[NSColor redColor] range:NSMakeRange(17, 7)];

//设置文字背景色
[colorTitle addAttribute:NSBackgroundColorAttributeName value:[NSColor whiteColor] range:NSMakeRange(17, 7)];

//添加下划线
[colorTitle addAttribute:NSUnderlineStyleAttributeName value:[NSNumber numberWithInteger:NSUnderlineStyleSingle] range:NSMakeRange(17, 7)];

NSTextField.attributedStringValue = colorTitle;
```


#### 使用方式二
创建属性字典，并将各种属性初始化。赋值，并利用方法appendAttributedString:添加入NSMutableAttributedString，将其赋给控件的attributedText属性。

```object-c
//初始化NSMutableAttributedString
NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc]init];

//设置字体格式和大小
NSString *str0 = @"欢迎使用,这是测试哦";
NSDictionary *dictAttr0 = @{NSFontAttributeName:[NSFont systemFontOfSize:14]};
NSAttributedString *attr0 = [[NSAttributedString alloc]initWithString:str0 attributes:dictAttr0];
[attributedString appendAttributedString:attr0];



//设置阴影属性，取值为NSShadow对象
NSString *str7 = @"欢迎使用,这是测试哦";
NSShadow *shadow = [[NSShadow alloc]init];
shadow.shadowColor = [NSColor redColor];
shadow.shadowBlurRadius = 1.0f;
shadow.shadowOffset = CGSizeMake(1, 1);
NSDictionary *dictAttr7 = @{NSShadowAttributeName:shadow};
NSAttributedString *attr7 = [[NSAttributedString alloc]initWithString:str7 attributes:dictAttr7];
[attributedString appendAttributedString:attr7];


//设置字体倾斜度 NSObliquenessAttributeName
NSString *str12 = @"设置字体倾斜度";
NSDictionary *dictAttr12 = @{NSObliquenessAttributeName:@(0.5)};
NSAttributedString *attr12 = [[NSAttributedString alloc]initWithString:str12 attributes:dictAttr12];
[attributedString appendAttributedString:attr12];


//段落样式
NSMutableParagraphStyle *paragraph = [[NSMutableParagraphStyle alloc]init];
//行间距
paragraph.lineSpacing = 10;
//段落间距
paragraph.paragraphSpacing = 20;
//对齐方式
paragraph.alignment = NSTextAlignmentLeft;
//添加段落设置
[attributedString addAttribute:NSParagraphStyleAttributeName value:paragraph range:NSMakeRange(0, attributedString.length)];

NSTextField.attributedStringValue = attributedString;  //uilabel则是attributedText

```