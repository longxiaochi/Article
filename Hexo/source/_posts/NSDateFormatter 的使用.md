---
layout: source/_posts
title: NSDateFormatter的使用
date: 2020-01-20 16:32:53
categories:
- 技术
  - Objective-C
tags: 
  - OC
---
### NSDateFormatter 的基本使用
NSDateFormatter 常用于iOS时间格式处理。现在来简单了解下。系统自带了几种格式：
<!--more-->
``` objective-c
typedef NS_ENUM(NSUInteger, NSDateFormatterStyle) {    
    NSDateFormatterNoStyle = kCFDateFormatterNoStyle,
    NSDateFormatterShortStyle = kCFDateFormatterShortStyle,
    NSDateFormatterMediumStyle = kCFDateFormatterMediumStyle,
    NSDateFormatterLongStyle = kCFDateFormatterLongStyle,
    NSDateFormatterFullStyle = kCFDateFormatterFullStyle
};

```
|  格式  |  日期  |  时间  |
| :----: | :----: | :----: | 
| NSDateFormatterNoStyle | "" | "" |
| NSDateFormatterShortStyle | 2020/1/20 | 下午5:19 |
| NSDateIntervalFormatterMediumStyle | 2020年1月20日 | 下午5:18:48 |
| NSDateIntervalFormatterLongStyle | 2020年1月20日 | GMT+8 下午5:19:51 |
| NSDateIntervalFormatterFullStyle | 2020年1月20日 星期一 | 中国标准时间 下午5:21:00 |

样例：
```Objective-C
NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
dateFormatter.locale = [[NSLocale alloc] initWithLocaleIdentifier:@"zh"];
dateFormatter.dateStyle = NSDateIntervalFormatterMediumStyle;
dateFormatter.timeStyle = NSDateIntervalFormatterMediumStyle;
NSString *dateStr = [dateFormatter stringFromDate:[NSDate date]];
```
### 自定义时间格式
```objective-C
NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
[dateFormatter setDateFormat:@"MMM dd, yyyy"];

dateFormatter.locale = [NSLocale currentLocale];
dateFormatter.timeZone = [NSTimeZone systemTimeZone];
NSString *timeStr = [dateFormatter stringFromDate:date];
```
```
参数参考：
GGG: 公元时代，例如AD公元
yy: 年的后2位
yyyy: 完整年
MM: 月，显示为1-12
MMM: 月，显示为英文月份简写,如 Jan
MMMM: 月，显示为英文月份全称，如 Janualy
dd: 日，2位数表示，如02
d: 日，1-2位显示，如 2
EEE: 简写星期几，如Sun
EEEE: 全写星期几，如Sunday
aa: 上下午，AM/PM
H: 时，24小时制，0-23
K：时，12小时制，0-11
m: 分，1-2位
mm: 分，2位
s: 秒，1-2位
ss: 秒，2位
S：毫秒
zzz：三位字符串表示“时区”（例如GMT）。缩写 Z

```
### NSLocale: 
 NSLocale是一个包含所有地区的语言与文化习俗的基础类。一个NSLocale的实例包含了针对这个地区内特定一群人的所有语言文化基准，其中包括：语言、键盘、数字、日期和时间格式、货币、排序和分类、符号、颜色与头像的使用。
 
 NSLocale与设备的设置有关：  
 系统的设置 -＞ 通用 -＞ 多语言环境 -＞ 区域格式  
 系统的设置 -＞ 通用 -＞ 日期与时间 -＞ 24小时制

1.+ (id)systemLocale : 设备默认的本地化信息。当不想用用户设定的本地化信息时，使用systemLocale对象来设置你想要的效果。
***
2.+ (id)currentLocale : 当前用户设定的本地化信息，即使修改了本地化设定，这个对象也不会改变。currentLocale取得的值可能被缓存在cache中。（不会跟随用户改变设置而改变）要想知道变化，则需要监听：NSCurrentLocaleDidChangeNotification
***
3.+ (id)autoupdatingCurrentLocale: 当前用户设定的本地化信息，此对象总是最新的本地化信息。（随着用户改变设置改变）
***

- 标示符初始化本地化信息
   - [[NSLocale alloc] initWithLocaleIdentifier:@"en_US"]; 指定语言与地区码。
- 参考： 
  1. https://nshipster.cn/nslocale/ 
  2. https://www.jianshu.com/p/38318cfbf986
  3. https://www.jianshu.com/p/0b5b576dcb99#chapter2


### NSTimeZone: 
NSTimeZone与设备的设置有关：系统的设置 -＞ 通用 -＞ 日期与时间 -＞ 时区  

1.+ (NSTimeZone *)systemTimeZone;
设备系统时区，不能代码修改。(大伙的iPhone都是自动默认北京，如在手机‘设置’修改为伦敦，返回伦敦时区)

2.+ (NSTimeZone *)localTimeZone;
本地时区，默认与systemTimeZone一致，可用代码修改。

3.+ (NSTimeZone *)defaultTimeZone;
默认时区，程序中定义的默认时区，默认与默认与systemTimeZone一致，可用代码修改。

4.+ (void)setDefaultTimeZone:(NSTimeZone *)aTimeZone;
用此方法可以改变localTimeZone、defaultTimeZone的值，(systemTimeZone不可改变)

5.[NSTimeZone timeZoneWithName:@"UTC"]: 获取指定时区。

