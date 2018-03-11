---
layout: post
title: 重定向
date: 2015-03-27
categories: Technology
tags: [重定向]
---
《Unix环境高级编程》第三章中的一道习题：请说明以下两条命令的区别。
```
./a.out > outfile 2>&1
./a.out 2>&1 > outfile
```
这里用到我们通常所说的重定向，但是自己以前一直理解的有出入，现在记录如下。
首先我们用代码来验证以上两条命令。`C版本代码`如下：
<!-- more -->
```C
#include <stdio.h>
#include <iostream>
int main() {
    printf("output to stdout\n");
    fprintf(stderr, "output to stderr\n");
    return 0;
}
```
首先我们执行第一条命令`./a.out > outfile 2>&1`，`output:`(~为提示符)
```
~ ./a.out > outfile 2>&1
~ cat outfile
output to stderr
output to stdout
```
这里说明bash是从左向右执行的，我们来分析`./a.out > outfile 2>&1`。`./a.out > outfile`会创建outfile（如果存在outfile则截断），并且将`a.out`的标准输出与`outfile`关联。`2>&1`则将`文件描述符2`的文件指针指向`文件描述符1`所关联的文件表项，也就是调用`dup2(1, 2)`。**0代表标准输入，1代表标准输出，2代表标准错误**。其结果就是程序将正常的输出和标准错误都输出到`outfile`中。但是，我考虑`outfile`中的内容应该为如下顺序：
```
output to stdout
output to stderr
```
为什么与程序的输出不一致呢？这里引出`缓冲区`的概念。
### 缓冲区
在内存空间中预留了一定的存储空间，这些存储空间用来缓冲输入或输出的数据，这部分预留的空间就叫做缓冲区。缓冲区根据其对应的是输入设备还是输出设备，分为输入缓冲区和输出缓冲区。缓冲区用在输入输出设备和CPU之间，用来缓存数据。它的目的是使低速的输入输出设备和高速的CPU能够协调工作，避免低速的输入输出设备占用CPU。
缓冲区分类：
- 全缓冲
当填满标准I/O缓存后才进行实际I/O操作。全缓冲的典型代表是对磁盘文件的读写。
- 行缓冲
在这种情况下，输入输出遇到回车才执行`I/O`操作。典型代表标准输入和标准输出。
- 不带缓冲
就是不进行缓冲，直接输出。比如标准错误。

下面接着考虑我们的程序语句`printf("output to stdout\n");`执行时，放到缓冲区。此时接着执行` fprintf(stderr, "output to stderr\n");`，标准错误没有缓冲直接输出`output to stderr`，然后在程序退出前刷新所有缓冲，接着输出`output to stdout`。修改程序手动刷新缓冲区，代码如下：
```C
#include "apue.h"
#include <stdio.h>
#include <iostream>
int main() {
    printf("output to stdout\n");
    fflush(stdout);
    fprintf(stderr, "output to stderr\n");
    return 0;
}
```
`output`：
```
output to stdout
output to stderr
```
下面我们考虑第二条命令`./a.out 2>&1 > outfile`，这里第一步执行`dup(1, 2)`，也就是标准输出的文件指针指向标准错误；第二步程序的标准输出重定向到`outfile`。也就是说标准错误(文件描述符2)的内容会输出到终端(文件描述符1关联)，而标准输出的内容会输出到`outfile`中。验证如下：
```
~ ./a.out 2>&1 > outfile
output to stderr
~ cat outfile
output to stdout
```
### 总结
> n > file creates/truncates file and associates it to file descriptor n. If n is not specified, 1 (i.e. standard output) is assumed.
n>&m copies (using dup2()) file descriptor m onto n.
So if you write ./a.out 2>& >outfile, then the standard output descriptor is first copied onto the stderr descriptor, and then stdout is redirected to outfile.
You can see those redirection operators as assignments if you like:
2>& >file would be read as fd2 := fd1; fd1 := "file", which is not the same as >file 2>& which is fd1 := "file"; fd2 := fd1

### 参考资料
[REDIRECTION](http://man7.org/linux/man-pages/man1/bash.1.html#REDIRECTION)
[一个长考的重定向问题](http://www.cnblogs.com/zhaoyl/archive/2012/10/22/2733418.html)
[C语言缓冲区(缓存)详解](http://c.biancheng.net/cpp/html/2413.html)
[How should I understand “> outfile 2>&1” and “ 2>&1 > outfile”?](http://stackoverflow.com/questions/25229772/how-should-i-understand-outfile-21-and-21-outfile)
