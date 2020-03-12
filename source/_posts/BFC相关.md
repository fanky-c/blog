---
title: BFC相关
date: 2020-03-12 15:00:41
tags:
    - BFC
    - 清除浮动
    - 元素重叠
---

### 什么是BFC?
1. BFC(block formatting context) 块级格式化上下文

### 如何产生BFC?
1. html根元素
2. float不为none（默认值）
3. display:table-cell/inline-block
4. overflow不为visible（默认值）
5. position不为static/releative


### BFC的作用？
1. 解决不和浮动元素重叠（浮动元素跟随非浮动元素）
```html
<div style="float:left;"></div>
<div></div>
```
2. 清除元素内部浮动

3. 解决上下相邻2个元素重叠
