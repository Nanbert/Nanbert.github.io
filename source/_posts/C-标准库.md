---
title: C++标准库
date: 2019-08-10 21:22:59
subtitle:
categories: C++
tags:
cover:
---
1.顺序容器
|方法|说明|
|:-:|:-:|
|c.resize(n)|调整c的大小为n个元素。要么多出的元素被丢弃,要么新添加默认值的元素|
|c.resize(n,t)|调整c的大小为n个元素。任何新添加的元素初始为t|
|c.shrink_to_fit()|只适用于vector,string和deque。将capacity()减少与size()相同大小|
|c.capacity()|只适用于vector和string,返回c可以保存多少元素|
|c.reserve(n)|只适用于vector和string,分配至少容纳n个元素的内存空间|
2.lambda
格式:`[capture list] (parameter list) -> return type {function body}`,capture list可以为空，里面一般是包含此lambda的函数的(非static)局部变量,该函数外的变量可以在函数体内直接用,return type可以省略。
3.优先队列
格式:`priority_queue<Type,Container,Functional>`
例子:`priority_queue<int,vector<int>,myCompare>`
