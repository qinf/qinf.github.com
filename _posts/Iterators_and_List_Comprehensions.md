title: 迭代器和列表解析
date: 2015-05-20
categories: Python
tags: [Python核心编程]
---

### range()和xrange()的区别：

range()返回一个列表，而xrange()返回一个可迭代的对象(不创建列表的完整拷贝)。

### 迭代器

迭代器就是有一个next()方法的对象，而不是通过索引来计数。当你需要下一项，调用next()就可以获得。当全部取完是引发一个StopIteration异常。如下：
<!-- more -->

```Python
for i in seq:
    do_something(i)
```
**实际上是这样工作的**
```Python
fetch = iter(seq)
while True:
    try:
        i = fetch.next()
    except StopIteration:
        break
    do_somthing(i)
```
在迭代可变对象时，修改迭代对象会引发错误。

### 列表解析 vs 生成器表达式

列表解析：`[expr for iter_var in iterable if cond_expr]`
生成器表达式: `(expr for iter_var in iterable if cond_expr)`
两者之间的区别是：列表解析生成所有的数据来创建一个列表(但数据量大的时候会影响性能)。而生成器表达式并不是真正创建数字列表，而是返回一个生成器，这个生成器在每次计算出一个条目后，把这个条目产生(yield)出来。生成器表达式使用"延迟计算"，在内存上更有优势。
书中关于生成器表达式的例子：

- 计算文件中单词的长度
```Python
sum(len(word) for line in data for word in line.split())`
```
- 交叉配对
```Python
rows = [1, 2, 3, 17]
def cols():
    yield 56
    yield 2
    yield 1
x_product_pairs = ((i, j) for i in rows for j in cols())
for pair in x_product_pairs:
    print pair
# result
#(1, 56)
#(1, 2)
#(1, 1)
#(2, 56)
# ...
```
- 列表解析获取文件中最长的行的长度
```Python
f = open('/pat/to/file', 'r')
allLineLens = [len(x.strip() for x in f]
f.close()
return max(allLineLens)
```
- 生成器表达式
```Python
f = open('/path/to/file/', 'r')
loggest = max(len(x.strip()) for x in f)
f.close()
return loggest
```
