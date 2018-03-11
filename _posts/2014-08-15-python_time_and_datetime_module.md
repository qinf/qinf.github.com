---
layout: post
title: Python time模块和datetime模块的使用
date: 2014-08-15
modfied:  
categories: Technology
tags: [python, time, datetime]
author: Junzhong Qin
---

## `time 模块`
### `time`模块内置函数
`time模块`是与平台相关的，该模块中大部分函数调用标准`C`平台同名库函数。该模块中的函数不处理1970年以前和2038年(Unix)以后的时间和日期。下面列举模块中的一些函数：
- `time.asctime([t])`
<!-- more -->
接受一个元祖或者由`gmtime()`或`localtime()`返回的`struct_time`为参数（默认参数），返回一个24个字符的格式化字符串。
```python
import time
print time.asctime()
# output, have the same ouput with argument time.localtime()
'Sun Sep 14 15:00:53 2014'
```
- `time.clock()`
用以浮点数计算的秒数返回当前的CPU时间。用来衡量不同程序的耗时，比time.time()更有用。
```pythoon
t0 = time.clock()
time.sleep(5)
print time.clock() - t0
```
- `time.ctime([secs])`
作用相当于asctime(localtime(secs))，未给参数相当于asctime()。
- `time.gmtime([secs])`
将浮点数时间戳转换成时间元祖`time_struct`。默认参数是`time.time()`。
```python
print time.gmtime(time.time())
```
- `time.localtime([secs])`
返回一个时间元祖，类似`time.gmtime()`。
- `time.mktime(tupletime)`
`time.localtime()`的反过程。接受时间元祖返回时间戳浮点数。
```python
print time.mktime(time.localtime())
```
- `time.sleep(secs)`
线程暂停运行`secs`秒。
- `time.strftime(format([, t]))`
接收一个时间元组，并返回以可读字符串表示的当地时间，格式由fmt决定。

> | 参数 | 说明 |
| ------- | :------ |
|%a - %A | 本地时间周几 简写 - 全称|
| %b - %B | 本地时间月份 简写 - 全称 |
| %c | 本地日期时间简写 格式：'09/14/14 15:38:48' |
| %d | 每月的第几天[01, 31] |
| %H | 小时[00, 23] |
| %I | 小时[01, 12] |
| %j | 一年中的第几天 [001 - 366] |
| %m | 月份[01 - 12] |
| %M | 分钟[00, 59] |
| %p | AM / PM |
| %S | 十进制秒[00, 61] |
| %U | 一年中的第几周，周日是一周中的第一天 |
| % w| 十进制表示每周的第几天[0(Sunday), 6]|
| %W| 一年中的第几周，周一为一周中的第一天 |
| %x | 本地日期表示 |
| %X| 本地时间表示|
| %y | 两位数字表示年[00, 99] |
| %Y | 四位数字表示年 |
| %Z | 时区名称 |
| %% | 字符`%`字面值 |  

- `time.strptime(str,fmt='%a %b %d %H:%M:%S %Y')`
根据fmt的格式把一个时间字符串解析为时间元组。
- `time.time()`
返回时间戳，1920纪元后的浮点秒数。
- `time.tzset()`
根据环境变量TZ重新初始化时间相关设置。
### `time`模块属性
- `time.timezone`
属性time.timezone是当地时区（未启动夏令时）距离格林威治的偏移秒数（>0，美洲;<=0大部分欧洲，亚洲，非洲）。
- `time.tzname`
属性time.tzname包含一对根据情况的不同而不同的字符串，分别是带夏令时的本地时区名称，和不带的。
### 使用举例
- 以格式`%Y-%m-%d %H:%M:%S`输出年-月-日 时:分:秒
```python
def Time(seconds = 0):
      seconds = float(seconds)
      if seconds == 0:
          string = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
      else:
          string = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(seconds))
      return string
```
测试输出：
```python
>>> print Time()
2014-09-15 00:26:54
>>> print Time(0)
2014-09-15 00:26:59
>>> print Time(1)
1970-01-01 08:00:01
>>> print Time(-1)
1970-01-01 07:59:59
```
- 以`%Y-%m-%d`格式输出年月日
```python
def Date(seconds = 0):
      seconds = float(seconds)
      if seconds == 0:
          string = time.strftime("%Y-%m-%d", time.localtime())
      else:
          string = time.strftime("%Y-%m-%d", time.localtime(seconds))
      return string
```
- 时间差计算，计算距离给定时间相距N分钟的时间
```
def get_n_minuts_before_time(n, format_time):
      format ="%Y-%m-%d %H:%M:%S"
      result=datetime.datetime(*time.strptime(format_time, format)[:6])-datetime.timedelta(minutes= n)
      return result.strftime(format)
# 测试
print Time()
print get_n_minuts_before_time(10, Time())
print get_n_minuts_before_time(-10, Time())
# output
2014-09-14 17:01:53
2014-09-14 16:51:53
2014-09-14 17:11:53
```
