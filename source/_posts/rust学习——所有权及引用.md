---
title: rust学习——所有权及引用
date: 2022-05-15 12:21:42
subtitle:
categories:
tags:
index_img: /img/rust.png
banner_img: /img/rust.png
---
## 定义
获取变量的引用，称之为借用  
如下代码
```rust
fn main() {
    let s1 = String::from("hello");
    let s=&s1;
}
```
本质如下图所示：
![](/img/rust_borrow.jpg)
`&s1`创建了一个指向s1的引用，但是并不拥有它,所以引用离开作用域后，不会被丢弃
## 一些tips
- 引用的作用域的结束位置为最后一次使用的位置
- 同一作用域，特定数据只能有一个可变引用
- 可变引用与不可变引用不能同时存在
- 同一时刻，你只能拥有一个可变引用，要么任意多个不可变引用
## Copy特征
如果一个类型拥有Copy特征，一个旧的变量在被赋值给其他变量后仍然可用，任何基本类型的组合可以Copy,不需要分配内存或某种形式资源的类型是可以Copy的。如下类型是可以Copy:
- 所有整数类型，比如u32。
- 布尔类型，bool，它的值是true和false。
- 所有浮点数类型，比如f64。
- 字符类型，char。
- 元组，当且仅当其包含的类型也都是Copy的时候。比如，(i32,i32)是Copy的，但 (i32,String)就不是。
- 不可变引用 &T ，例如转移所有权中的最后一个例子，但是注意:**可变引用&mut T是不可以Copy的**
## Clone特征
- 所有引用都实现了Clone特征
