---
title: 求钟表时针和分钟夹角问题
---

这是一道我面试中遇到的算法题，我觉得蛮有意思的所以写出来与大家分享一下。


### 问题

给定一个时间值，需要计算出表盘上时针与分针的之间的夹角度数。

### 思路

虽然题目中只有一个条件，但其实已经有许多隐含条件包括在内。

我们要得出的是时针和分针之间的角度，在一个钟表上因为钟表是圆形，所以我们得出第二个已知条件，表盘360°；第三个已知条件，表盘上有 12 个小时刻度；第三个已知条件，分针走一圈时针走一个刻度。

虽然题目里没有给出以上条件，但常识告诉我们这些条件。

拿到这些条件我们的计算就简单了：

首先计算得出一些我们需要的数值，如每增加一小时时针走多少度，分针增加一分钟走多少度，分针每走一分钟带动时针走多少度。

得到这些值后我们与给定的时间进行计算：

小时 * 每增加一小时时针走多少度 = 整点时间时时针度数

分钟 * 分针每走一分钟带动时针走多少度 = 分针带动时针度数

整点时间时时针度数 + 分针带动时针度数 = 此时时针度数

分钟 * 分针增加一分钟走多少度  = 分针度数

此时时针度数 - 分针度数 = |夹角度数|

取绝对值就得出了它们之间的夹角度数

### 实现代码

Python 版

```python
"""
给定一个时间值算出时针与分针直接的夹角度数
"""
import time


def calculate(times):

    hour = times.tm_hour - 12 if times.tm_hour > 12 else times.tm_hour
    minute = times.tm_min

    # 钟表总度数
    timekeeper_angle = 360
    # 时间刻度份数
    scale = 12
    # 每份时间刻度占用的度数
    one_scale = timekeeper_angle / scale
    # 分针每走1分钟时针转动的度数
    one_relate_scale = one_scale / 60
    # 分针每走1分钟转动的度数
    one_minute_scale = timekeeper_angle / 60

    # 小时*每份度数 得 整点时间的时针度数
    integral_scale = hour * one_scale
    # 分针带动时针走的度数
    relate_scale = minute * one_relate_scale

    # 时针真正的起始点度数
    relay_hour_scale = integral_scale + relate_scale

    # 分钟的现在度数
    relay_minute_scale = minute * one_minute_scale

    print(abs(relay_hour_scale - relay_minute_scale))


calculate(time.localtime(time.time()))
```

PHP 版

```php
<?php

function calculate($time){

    $hour = (int) date('H', $time);
    $hour =  $hour > 12 ? $hour - 12 : $hour;
    $minute = (int) date('m', $time);

    // 钟表总度数
    $timekeeperAngle = 360;
    // 时间刻度份数
    $scale = 12;
    // 每份时间刻度占用的度数
    $oneScale = $timekeeperAngle / $scale;
    // 分针每走1分钟时针转动的度数
    $oneRelateScale = $oneScale / 60;
    // 分针每走1分钟转动的度数
    $oneMinuteScale = $timekeeperAngle / 60;

    // 小时*每份度数 得 整点时间的时针度数
    $integralScale = $hour * $oneScale;
    // 分针带动时针走的度数
    $relateScale = $minute * $oneRelateScale;

    // 时针真正的起始点度数
    $relayHourScale = $integralScale + $relateScale;

    // 分钟的现在度数
    $relayMinuteScale = $minute * $oneMinuteScale;

    echo abs($relayHourScale - $relayMinuteScale);
}

calculate(time());
```

