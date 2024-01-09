---
title: python基础
date: 2024-01-10 03:48:23
tags:
---

# 函数
## 默认参数
python不支持重载，但是支持默认参数
```python
def add(a=0, b=0, c=0):
    return a+b+c
add(1,2)
add(1,2,3)
add(c=50,a=2, b= 2)
```
## 可变参数
```python
def add(*args):
    total = 0
    for val in args:
        total+=val
    return total
```
# 模块
- python中每个文件就代表一个模块,模块名即为文件名
- `__name__`是python中一个隐含的变量，它代表了模块的名字，只有被Python解释器直接执行的模块的名字才是`__main__`
# 作用域
```python
def foo():
    b = 'hello'
    # Python中可以在函数内部再定义函数
    def bar():
        c = True
        print(a)//全局变量
        print(b)//嵌套变量
        print(c)//局部变量
    bar()
    # print(c)  # NameError: name 'c' is not defined
if __name__ == '__main__':
    a = 100 # 不属于任何函数，即为全局变量
    # print(b)  # NameError: name 'b' is not defined
    foo()
```
- 除上述的全局作用域、嵌套作用域、局部作用域外，还存在内置作用域，如input,print,int等
- 查一个变量会按局部作用域、嵌套作用域、全局作用域、内置作用域来找
## global和nonlocal
- global一般用在函数中指示该变量来自全局，如果全局中没有,那么就会定义一个，并置于全局中
- nonlocal则指示变量处于嵌套作用域中
# 闭包
- 闭包可以使一个局部变量的生命周期延长，使其在定义它的函数调用结束后依然可以用它的值
# pip
## matplotlib依赖路径
- numpy->contourpy->cycler->fonttools->kiwisolver->packaging->pillow->pyparsing->six->python_dateutil->matplotlib
