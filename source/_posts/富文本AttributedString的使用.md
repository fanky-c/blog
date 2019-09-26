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
