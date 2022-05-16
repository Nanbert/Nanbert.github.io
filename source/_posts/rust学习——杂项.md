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
## 生命周期初认识
rust会分析所有引用的生命周期,来提高安全性
### 函数中的生命周期
函数的返回值如果是一个引用类型，那么它的生命周期只会来源于：
- 函数参数的生命周期
- 函数体中的某个新建引用的生命周期
第二种情况明显是错误，典型的悬垂引用
### 使用说明
- 格式：`fn func<'a> (para1: &'a type,para2: &'a type)-> &'a type{}`
- 说明：**在通过函数签名指定生命周期参数时，我们并没有改变传入引用或者返回引用的真实生命周期，而是告诉编译器当不满足此约束条件时，就拒绝编译通过。**
- 含义：当传递具体值时，'a的生命周期等于para1和para2(取两个参数的交集)，返回的生命周期也等于'a,若不满足条件就报错
### 结构体中的生命周期
- 格式：
```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```
- 含义：结构体ImportantExcerpt所引用的字符串str必须比改结构体活得长
### 三条默认添加生命周期规则
函数或方法中，参数的生命周期被称为输入生命周期，返回值的生命周期被称为输出生命周期。rust有以下三条规则:
- 每一个引用参数都会获得独自的生命周期  
`fn foo(a:& i32,y:& i32)`编译器自动转成`foo<'a, 'b>(x: &'a i32, y: &'b i32)`
- 若只有一个输入生命周期(函数参数中只有一个引用类型)，那么该生命周期会被赋给所有的输出生命周期，也就是所有返回值的生命周期都等于该输入生命周期  
`fn foo(x: &i32) -> &i32`会转为`fn foo<'a>(x: &'a i32) -> &'a i32`
- 若存在多个输入生命周期，且其中一个是 &self 或 &mut self，则 &self 的生命周期被赋给所有的输出生命周期
### 生命周期约束
- `'a:'b`：a的生命周期大于等于b
- `T:'a`：类型T必须比a生命周期大
## 再借用
可变引用和不可变引用不能同时存在(即作用域不重叠)，但再借用可以避免，如下
```rust
fn main() {
    let mut p = Point { x: 0, y: 0 };
    let r = &mut p;
    // reborrow! 此时对`r`的再借用不会导致跟上面的借用冲突
    let rr: &Point = &*r;

    // 再借用`rr`最后一次使用发生在这里，在它的生命周期中，我们并没有使用原来的借用`r`，因此不会报错
    println!("{:?}", rr);

    // 再借用结束后，才去使用原来的借用`r`
    r.move_to(10, 10);
    println!("{:?}", r);
}
```
## deref特征
- 在表达式中必须显式的解引用，并且不会发生递归的解引用，当指明时，有几层就解几层,并且本质上是`*y`=>`*(y.deref())`
- 隐式的解引用，会递归，直到类型匹配
- 当 T: Deref<Target=U>，可以将 &T 转换成 &U
- 当 T: DerefMut<Target=U>，可以将 &mut T 转换成 &mut U
- 当 T: Deref<Target=U>，可以将 &mut T 转换成 &U
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
