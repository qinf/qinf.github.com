---
layout: post
title: Python中的深拷贝与浅拷贝
date: 2015-05-30
categories: Python
tags: [Python核心编程]
---

`Python`中对象的赋值实际上是简单的对象引用。也就是说，当你创建了一个对象，然后将其赋值给另一个变量时，`Python`只是拷贝了这个对象的引用。
比如我们创建一个`person`数据结构来表示一个账户。然后为小红和小明分别拷贝一份。分别使用切片和工厂方法进行拷贝。
<!-- more -->
```Python
>>> person = ['name', ['savings', 100.0]]
>>> hubby = person[:]   # slice copy
>>> wifey = list(person)    # factory function copy
>>> [id(x) for x in person, hubby, wifey]   # use id() to identify different object
[140705159386824, 140705159444456, 140705159444960]
>>> hubby[0] = 'xiaoming'
>>> wifey[0] = 'xiaohong'
>>> hubby, wifey
(['xiaoming', ['savings', 100.0]], ['xiaohong', ['savings', 100.0]])
>>> hubby[1][1] = 50.0  # change xiaoming's savings
>>> hubby, wifey
(['xiaoming', ['savings', 50.0]], ['xiaohong', ['savings', 50.0]])
```
我们发现小红的账户的钱也变少了，这是为什么呢？原因是我们只做了浅拷贝。对一个对象进行**浅拷贝**就是创建了一个类型跟原对象一样，其内容是原对象的引用。也就是说拷贝出来的对象本身是新的，但是内容与原有对象一样。
序列对象的默认拷贝类型是浅拷贝。通常以下方式进行的都是浅拷贝：

 1. 完全切片`[:]`
 2. 利用工程函数,比如`list()`、`dict()`
 3. 使用`copy`模块的`copy`函数
另一个关于上面账户的问题，为什么当小红的名字被赋值时，小明的名字没有改变？这是因为在这个结构中(`['name', ['savings', 100.0]]`)第一个对象是字符串，字符串是一个不可变对象。而第二个对象是一个列表，列表是可变对象。当进行浅拷贝时，字符串被显示的进行拷贝，创建了新的字符串，而列表知识原来对象的一个拷贝。我们可以使用内建函数`id()`来看一下：
```Python
>>> [id(x) for x in hubby]
[140705159487088, 140705159386104]  # the second obj has the same id
>>> [id(x) for x in wifey]
[140705159486704, 140705159386104]
```
关于这个列子我们用图显示如下：
![深拷贝][2]
![浅拷贝][3]

要得到一个对象的深拷贝需要使用`copy.deepcopy()`函数。如下展示：
```Python
>>> person = ['name', ['savings', 100.0]]
>>> hubby = person
>>> import copy
>>> wifey = copy.deepcopy(person)
>>> [id(x) for x in person, hubby, wifey]
[140705159445032, 140705159445032, 140705159445104]
>>> hubby[0] = 'xiaoming'
>>> wifey[0] = 'xiaohong'
>>> hubby, wifey
(['xiaoming', ['savings', 100.0]], ['xiaohong', ['savings', 100.0]])
>>> hubby[1][4] = 50.0
>>> hubby, wifey
(['xiaoming', ['savings', 50.0]], ['xiaohong', ['savings', 100.0]])
>>> [id(x) for x in hubby]
[140705159486704, 140705159444384]  # hubby's savings and wifey's savings have different id
>>> [id(x) for x in wifey]
[140705159486656, 140705159446400]
```
关于拷贝操作的一些说明：

 1. 非容器类型(比如数字、字符串和其他“原子”类型的对象，像代码、类型和`xrange`对象等)没有被拷贝一说，浅拷贝是用完全切片操作完成的。
 2. 如果元组变量只包含原子类型对象，则对它不能进行深拷贝。如果我们的账户类型改为元祖类型，即使使用深拷贝也只能得到一个浅拷贝。
```Python
>>> person = ['name', ('savings', 100.0)]
>>> newPerson = copy.deepcopy(person)
>>> [id(x) for x in person]
[140705160392304, 140705159355280]
>>> [id(x) for x in newPerson]
[140705160392304, 140705159355280]
>>> person = ['name', ('savings', 100.0)]
>>> newPerson = copy.deepcopy(person)
>>> [id(x) for x in person]
[140705160392304, 140705159355352]  # person's savings and newPerson's savings have same id 
>>> [id(x) for x in newPerson]
[140705160392304, 140705159355352]
```


  [1]: http://7xj0l3.com1.z0.glb.clouddn.com/deep_copy.jpg
  [2]: http://7xj0l3.com1.z0.glb.clouddn.com/deep_copy.jpg
  [3]: http://7xj0l3.com1.z0.glb.clouddn.com/shallow_copy.jpg
  [4]: http://7xj0l3.com1.z0.glb.clouddn.com/deep_copy.jpg
