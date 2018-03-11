---
layout: post
title: 数据抽样问题  
date: 2015-06-10 15:30:16  
categories: Technology  
tags: [大数据]
---
## 问题1 海量数据中随机抽取一行
> 题目来源于《编程珠玑(第二版) p125.10》

题目：如何从n个对象（可以依次看到这n个对象，但事先不知道n的值）中随机选择一个？具体来说，如何在事先不知道文本行数的情况下读取该文件，从中随机抽取一行？  
**解析**：该抽样叫[蓄水池抽样](https://zh.wikipedia.org/wiki/水塘抽樣)。因为不可以事先知道n的大小，所以不能直接使用随机数。因为是随机抽样，所以每行被抽取的概率为1/n。我们以1的概率抽取第一行存储为s，以1/2的概率抽取第二行并替换s...以1/n的概率抽取第n行，并替换s。下面证明第k行被抽取为结果的概率为：
<!-- more -->
![](http://7xj0l3.com1.z0.glb.clouddn.com/%202015-08-27.8.50.58.png)  
由此证明可知每行都会以等概率对机会被抽取到。`Python`代码实现如下:  

```Python
import sys
import random
def randomSampling():
    s = ''
    i = 1
    for line in sys.stdin:
        if i == 1:
            s = line.strip()
        else:
            if (random.randint(1, i) == 1): # 以1/k的概率被抽取到
                s = line.strip()
        i = i + 1
    return s
if __name__ == '__main__':
    print randomSampling()
```
## 问题2 海量数据中抽取k行
**题目**：从海量文本中抽取k行，使得每行被抽取的概率相等。  
**分析**：使用蓄水池抽样，先抽取k行并保存到集合set中。以k/k＋1的概率抽取第k＋1行并替换掉set[0]，以k/k+2的概率抽取第k＋2行并替换掉第set[1]行。以k/k+m对概率抽取第k+m行，并替换掉set[(k+m)%k]行。每行依然是等概率选取，看如下公式：![](http://7xj0l3.com1.z0.glb.clouddn.com/QQ20150827-2@2x.png)Python代码实现如下：

```Python
import sys
import random

def randomSampling(k):
    myset = []
    i = 1
    for line in sys.stdin:
        if i <= k:
            myset.append(line.strip())
        else:
            if (random.randint(1, i) <= k):
                myset[i % k] = line.strip()
        i += 1
    return myset

if __name__ == '__main__':
    for line in randomSampling(3):
        print line.strip()
```
## 其他
推荐看一下[`GoCalf`](http://www.gocalf.com/blog/)的几篇博客，写的很不错。  
[单次遍历，带权随机选取问题](http://www.gocalf.com/blog/weighted-random-selection.html)  
[单次遍历，等概率随机选取问题](http://www.gocalf.com/blog/random-selection.html)  
[等概率随机排列数组（洗牌算法）](http://www.gocalf.com/blog/shuffle-algo.html)
