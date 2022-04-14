---
title: rust学习——杂项
date: 2022-04-13 21:48:13
subtitle:
categories:
tags:
cover:
---

## 文档注释
### 注意点
- 文档注释需要位于lib类型的包中，如`src/lib.rs`
- 文档注释可以使用markdown语法
- 被注释的对象需要使用pub对外可见
### 格式
- 针对函数和结构体：行注释`///`和块注释`/** ... */`
- 针对包和模块级别：行注释`//!`和块注释`/*! ... */`
- 包和模块级别的必须加到最上方
### 文档测试
`cargo test`命令可以运行包含在文档注释中的代码块，当然得包含assert等相关语句。
- 得使用完整的路径来调用函数，因为是在独立线程中运行的
- 如果测试用例造成panic,且这panic是预期结果，得使用如下:
```rust
/// ```rust,should_panic
/// // panics on division by zero
/// world_hello::compute::div(10,0);
/// ```
```
- 隐藏测试内容：当我们不想在文档中展示测试内容时可以使用`#`来隐藏
```rust
/// ```
/// # //该行被隐藏
/// 该行正常
/// ```
```
### 文档跳转
该功能可以实现对外部项的链接或本地代码的跳转
```rust
/// 跳转到标准库[`Option`],[`std::future`]
/// 跳转到自己的结构体[`crate::MySpecialFormatter`]
pub struct MySpecialFormatter;
```
- 同名项的跳转
```rust
/// 跳转到结构体  [`Foo`](struct@Foo)
pub struct Bar;

/// 跳转到同名函数 [`Foo`](fn@Foo)
pub struct Foo {}

/// 跳转到同名宏 [`foo!`]
pub fn Foo() {}

#[macro_export]
macro_rules! foo {
  () => {}
}
```

### 文档搜索别名
在搜索时，会跳到相应的位置
```rust
#[doc(alias = "x")]
#[doc(alias = "big")]
pub struct BigX;

#[doc(alias("y", "big"))]
pub struct BigY;
```
### 命令
- `cargo doc`直接生成HTML文件，放在target/doc目录下
- `cargo doc --open`在浏览器中查看

## tips
- rust中无法直接为外部类型实现外部特征，可以使用`newtype`:
```rust
struct Array(Vec<i32>);

use std::fmt;
impl fmt::Display for Array {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "数组是：{:?}", self.0)
    }
}
fn main() {
    let arr = Array(vec![1, 2, 3]);
    println!("{}", arr);
}
```
