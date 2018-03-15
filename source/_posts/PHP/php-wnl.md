---
title: 万年历接口
date: 2018-01-29 18:30:30
category: [Web]
tags: [PHP]
toc: true
---

使用PHP写的一个万年历接口

## 说明
支持的年份为1900-2100（农历），支持公历和公历互转。接口基地址：http://api.ieclipse.cn/wnl/ 主要功能列表

- 公历转农历
- 农历转公历
- 支持公历和农历节日
- 支持干支纪年、纪月、纪日、纪时
- 支持农历闰月

## 示例json

``` yaml
{
    "code":0, //响应码，为0时表示接口正常响应
    "msg":"ok",
    "time":1517217808,
    "data":{
        "isToday":false,
        "sYear":"2018", //公历年
        "sMonth":"03", //公历月
        "sDay":16, //公历日
        "sWeek":5, //公历周几， 0－6，0表示周日
        "sHour":17, //公历小时
        "sFestival":"", //公历节日，如元旦
        "lYear":2018, //农历年
        "lMonth":1, //农历月
        "lDay":29, //农历日
        "lFestival":"",//农历节日，如春节
        "isLeap":false, //农历闰月
        "hzYear":"二〇一八", //农历年大写
        "hzMonth":"正", //农历月汉字,如果为闰月，前面有一个闰字
        "hzDay":"廿九",//农历日
        "cWeek":"五",//汉字星期几
        "cYear":"戊戌", //汉字干支年
        "cMonth":"乙卯",//汉字干支月
        "cDay":"丁未",//汉字干支日
        "cAnimal":"狗",//生肖年
        "cTerms":"",//农历节气，如立春
        "cMnumber":1431,
        "cDnumber":43183,
        "cHour":"己酉" //汉字干支时
    } 
}
```

## 接口说明
### 请求参数
- date: 日期和时间，以`-`分隔，如2018-01-29-18，2019-01-29
- l: 是否农历闰月，当date为农历时有效，如`l=true`

### 响应
- cHour: 当参数包含小时时，有此字段
- cAnimal: 农历日期对应的生肖，遇到农历立春时，生肖会变化。
- isLeap: 是否农历闰月

## 公历转农历
http://api.ieclipse.cn/wnl/lunar

## 农历转公历
http://api.ieclipse.cn/wnl/solar

## 应用
- [生辰助手](http://www.ieclipse.cn/birthday-tool)