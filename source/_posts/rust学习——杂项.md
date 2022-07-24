---
title: rust学习——杂项
date: 2022-04-13 21:48:13
subtitle:
categories:
tags:
index_img: /img/rust.png
banner_img: /img/rust.png
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
## 函数中mut位置
- `fn do1(c: String) {}`：表示实参会将所有权传递给c
- `fn do2(c: &String) {}`：表示实参的不可变引用（指针）传递给c，实参需带& 声明
- `fn do3(c: &mut String) {}`：表示实参可变引用（指针）传递给c，实参需带let mut声明，且传入需带&mut
- `fn do4(mut c: String) {}`：表示实参会将所有权传递给c，且在函数体内c是可读可写的，实参无需mut声明
- `fn do5(mut c: &mut String) {}`：表示实参可变引用指向的值传递给c，且c在函数体内部是可读可写的，实参需带let mut声明，且传入需带&mut
 一句话总结：在函数参数中，冒号左边的部分，如：mut c，这个 mut 是对函数体内部有效；冒号右边的部分，如：&mut String，这个&mut是针对外部实参传入时的形式（声明）说明。
例子:
```rust
fn main() {
    let d1 = "str".to_string();
    do1(d1);

    let d2 = "str".to_string();
    do2(&d2);

    let mut d3 = "str".to_string();
    do3(&mut d3);

    let d4 = "str".to_string();
    do4(d4);

    let mut d5 = "str".to_string();
    do5(&mut d5);
}

fn do1(c: String) {}

fn do2(c: &String) {}

fn do3(c: &mut String) {}

fn do4(mut c: String) {}

fn do5(mut c: &mut String) {}
```
## 方法中的self
- 在impl中`&self`是`self:&Self`的简写
- Self指代被实现方法或特征的结构体类型，self指代此类型的实例

## cfg!
### debug_assertions
`if cfg!(debug_assertions)`debug模式下为true

## 常用编译器属性标记
- `#![allow(unused_variables)]`
- `#[allow(dead_code)]`
## 有用冷门宏
### unimplemented!()
告诉编译器该函数尚未实现，还有todo!(),如下：
```rust
#[allow(dead_code)]
fn read(f: &mut File, save_to: &mut Vec<u8>) -> ! {
    unimplemented!()
}
```
### dbg!()
接受一个表达式，并取走所有权，然后打印文件名，行号等debug信息
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```
