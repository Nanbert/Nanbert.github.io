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
# 字符串
- 支持`*`号，重复字符串
- 三个单引号或双引号，可以支持折行
- `s1 = r'\n\hello''`,不会转义
## 相关函数

|用法|说明|
|:-:|:-:|
|len(str1)|计算字符串长度|
|str1.capitalize()|获得字符串首字母大写的拷贝|
|str1.title()|获得每个单词首字母大写的拷贝|
|str1.upper()|获得字符串变大写的拷贝|
|str1.find('a')|查找子串的位置，如果没有返回-1|
|str1.index('a')|查找子串的位置，如果没有抛出异常|
|str1.startswith('he')|是否以指定的字符串开头|
|str1.endswith('he')|是否以指定的字符串结尾|
|str1.center(50, '\*')|以指定宽度居中并在两侧填充指定字符|
|str1.rjust(50, '\*')|以指定宽度靠右放置|
|str1.ljust(50, '\*')|以指定宽度靠左放置|
|str1.isdigit()|检查字符串是否只由数字组成|
|str1.isalpha()|检查字符串是否只由字母组成|
|str1.isalnum()|检查字符串是否由字母和数字组成|
|str1.strip()|去除左右两侧空格的拷贝|
|str1.lstrip()|去除左两侧空格的拷贝|
|str1.rstrip()|去除右两侧空格的拷贝|

# print
```python
a, b = 5, 10
print('%d * %d = %d' % (a, b, a * b))
```

```python
a, b = 5, 10
print('{0} * {1} = {2}'.format(a, b, a * b))
```

```python
a, b = 5, 10
print(f'{a} * {b} = {a * b}')
```

# list
- 定义：`t = []`
- 转换元组：`tuple1 = tuple(list1)`
## 同时遍历元素和下标
```python
for index, elem in enumerate(list1):
    print(index, elem)
```
## 函数相关

|例子|说明|
|:-:|:-:|
|list1.append(ele)|在末尾增加一个元素|
|list1.insert(1, 400)|在下标为1的元素前加个元素|
|list1.extend(list2)|合并两个列表，等价于list1+=list2|
|list1.remove(value)|去除列表中第一个为value的元素,如果列表中没该元素则抛出异常|
|list1.pop(index1)|去除下标为index1的元素，默认为末尾元素|
|list1.clear()|清空列表|
|list1.reverse()|倒转元素|
|list1.sort(reverse=True)|列表本身排序|
|sorted(list1, reverse=True)|返回列表逆排序的拷贝|

## 切片
- `fruits[::-1]`: 获得倒转元素列表的拷贝
- `fruits[:]`: 获得列表的拷贝

# 生成式和生成器
- 生成式：`f = [x+y for x in 'ABCDE' for y in '1234567']`,耗费空间更多
- 生成器：`f = (x ** 2 for x in range(1,1000))`,不占用额外空间，但是取元素时会内部计算

# 元组
- 定义：`t = ('ss', 'gg')`
- 转换列表：`list1 = list(tuple1)`
- 相比列表的优势:tuple不可变，时空上都优于列表

# 集合
- 定义：`set1 = set()`,`set2 = set((1,2,3,4,3,2))`,`set3=set(range(1,10))`
- set里的元素是排好序的
## 相关函数

|例子|说明|
|:-:|:-:|
|set1.add(value)|增加一个元素|
|set2.update([11,12])|增加一个列表的元素|
|set2.remove(12)|去除值12元素，没有会抛出异常|
|set2.discard(12)|同remove但不会抛出异常|
|set1.pop()|删除末尾元素|
|set1.issuperset(set2)|set2是否包含set1|
|set1.issubset(set2)|set1是否包含set2|

![](/images/python_set.png)

# 字典
## 定义
- `scores = {'骆昊': 95, '白元芳': 78, '狄仁杰': 82}`
- `items1 = dict(one=1, two=2, three=3, four=4)`
- `items2 = dict(zip(['a', 'b', 'c'], '123'))`
- `items3 = {num: num ** 2 for num in range(1, 10)}`
## 函数相关

|例子|说明|
|:-:|:-:|
|scores.update(冷面=67, 方启鹤=85)|增加两组元素|
|scores.get('xx',60)|get方法也是通过键获取对应的值,如果不存在返回60|
|scores.popitem()|去除末尾元素|
|scores.pop('xx', 10)|去除键为xx的元素,如果没有返回10|
|scores.clear()|清空|

# sys
- sys.getsizeof(var1): 打印变量的空间

# yield

# tips
- `__doc__`：xx.__doc__,会打印xx函数或方法的说明
