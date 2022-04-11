---
title: rust学习——字符串
date: 2022-04-10 04:21:11
subtitle:
categories:
tags:
cover:
---
## 字符串的性质
- rust的字符是Unicode类型，因此字符占据4个字节，但是字符串（无论String还是&str）是UTF-8编码，也就是字符串中的字符所占的字节数是变化的(1-4)
- 对字符串的切片操作一定落在utf-8字符的边界上，例如中文在utf-8占用3个字节，一定取3的整数，否则会崩溃
- 由于是utf-8无法像其他语言一样使用中括号索引单个字符
- &str是硬编码，String为可变、可增长，在作用域结束自动调用drop,释放堆上空间
## String与&str的转换
### &str->String
- `String::from("hello,world")`
- `"hello,world".to_string()`
### String->&str
- 取引用`&s`
- `s.as_str()`
## String的常用操作
### 追加
改变原本字符
- 追加字符：`s.push('r')`
- 追加字符串：`s.push_str("ust")`
### 插入
改变原本字符
- 插入字符：`s.insert(5,',')`
- 插入字符串：`s.insert_str(5,"i like")`
### 替换
#### replace
- 适用于String和&str，替换所有匹配的，返回新的字符串
- `s.replace("old pattern","new pattern")`
#### replacen
- 适用于String和&str，指定替换的个数，返回新的字符串
- `s.replacen("old pattern","new pattern",2)`
#### replace_range
- 该方法改变原来字符串，第一个参数指定范围，第二个是新串
- `s.replace_range(7..8,"R")`
### 删除
#### pop
- 删除并返回字符串的最后一个字符的Option类型（若字符串为空，返回None），改变原来字符串
- 会自动定位最后一个字符的起始字节
- `s.pop()`
#### remove
- 删除指定位置的字符，返回是该字符的字符串，如果不是合法的字符边界，会出错
- 会改变原来的字符串
- `s.remove(3)`
#### truncate
- 截断指定位置开始到最后的所有内容，如果位置不是合法边界会出错，会改变原内容
- `s.truncate(3)`
#### clear
- 清空原来的字符串
- `s.clear()`
### 连接
### `+`或`+=`
- 本质上是调用`fn add(self, s: &str)->String`
- 所以操作符左边为String,会发生所有权转移(离开函数实际上就丢失了)，右边只能是&str类型
- `+=`会发生原来的字符串会丢失所有权，绑定到一个新的字符串上
### format!
- 适用于String和&str
- `let s = format!("{} {}!",s1,s2)`
## UTF-8字符串的遍历
### chars--直观意义上的字符遍历
- 该方法以Unicode字符方式遍历，是人类直观的映像
```C
//会分别输出"中国人"三个字
for c in "中国人".chars() {
    println!("{}", c);
}
```
### bytes--按底层字节遍历
- 按底层字节数组遍历
```C
fn main() {
for b in "中国人".bytes() {
    println!("{}", b);
}
```
