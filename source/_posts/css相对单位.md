---
title: css相对单位
date: 2018-08-26 16:33:38
tags:
    - css
    - line-height
    - 样式无单位
---

#### css无单位和line-height

##### css无单位
有一些属性可以接收不带单位的数值（意思就是一个不带长度单位的数字），如line-height、z-index和font-weight（700等于bold，400等于normal，如此类推）。你也可以在需要长度单位的地方（如px、em、rem）使用一个不带单位的0，因为长度已经是0了，带不带单位也无所谓了 —— 0px 等于 0% 等于 0em。

***不带单位的0只可以表示长度单位和百分比的值，譬如padding、border和width。而对于一些特殊的情况，如度数（degrees）或者像秒这样基于时间的值（time-based values），是不可以使用不带单位的0的。*** 



##### line-height
line-height属性最特别的地方，在于同时支持带单位和不带单位的值。你应该保持使用不带单位的数值，因为这样就可以从父元素继承。

**浏览器默认的行高取决于用户代理。桌面浏览器（包括火狐浏览器）使用默认值，约为1.2，这取决于元素的font-family** 

代码案例：


父级元素：line-height:150%;font-size:16px;


子元素：  font-size:30px;

* 单位是px（12px): 继承父元素行高12px。

* 单位是% (150% * 16px浏览器默认字体大小)：字元素的行高等于16px * 150% = 24px。

* 单位 （1.5em)：字元素的行高等于16px * 150% = 24px。

* 无单位  (1.5)：字元素的行高等于30px * 150% = 45px。

****总结：有单位时，子元素继承了父元素计算得出的行距；无单位时继承了系数，子元素会分别计算各自行距（推荐使用）。****


#### 参考：
* https://mp.weixin.qq.com/s/ilkmqnwVvPLjiVfEyNadHg
* https://www.zhihu.com/question/20394889
