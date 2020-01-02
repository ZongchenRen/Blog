---
title: 数字滚动插件CountUp
date: 2020-01-02 22:32:29
categories: 前端
tags:
- 数字滚动
---

{% asset_img image01.gif  %}

<!-- more -->

# 数字滚动插件CountUp

给下面的数字添加滚动功能：

```html
<body>
  <div class="num">50</div>
<body>
```

首先引入jq和js插件

```html
<script type="text/javascript" src="jquery-1.11.1.min.js"></script>
<script type="text/javascript" src="jquery.countUp.js"></script>
```

然后，给JQuery 添加方法，设置参数

```html
<script type="text/javascript">
    //调用案例，需要在被调用的标签内 写上最终的数值
    jQuery(function($) {
        $(".timer").countTo({
            lastSymbol:" %", //显示在最后的字符
            from: 0,  // 开始时的数字
            speed: 2000,  // 总时间
            refreshInterval: 10,  // 刷新一次的时间
            beforeSize:0, //小数点前最小显示位数，不足的话用0代替 
            decimals: 2,  // 小数点后的位数，小数做四舍五入
            onUpdate: function() {
            },  // 更新时回调函数
            onComplete: function() {
                for(i in arguments){
                    console.log(arguments[i]);
                }
            }
        });
    });
</script>
```

[官方地址](http://inorganik.github.io/countUp.js/)