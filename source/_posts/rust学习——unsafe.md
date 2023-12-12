---
title: rust学习——unsafe
date: 2022-06-18 14:48:12
subtitle:
categories:
tags:
index_img: /images/rust.png
banner_img: /images/rust.png
---
unsafe的主要功能提供以下五个标题功能，不能关闭rust的任何其他安全检查（会关闭内存安全方面的检查）,unsafe也只应该用于这5方面，其他尽可能用安全代码
## 裸指针
- 创建裸指针是安全的行为，而解引用裸指针才是不安全的行为 
- 可以绕过 Rust 的借用规则，可以同时拥有一个数据的可变、不可变指针，甚至还能拥有多个可变的指针
- 并不能保证指向合法的内存
- 可以是 null
- 没有实现任何自动回收
- 标准库的智能指针都可以通过into_raw转成裸指针
```rust
let r1= &num as *const i32
unsafe{
	println!("r1 is: {}",*r1);
}
```
## 调用一个unsafe或外部函数
- 一个函数内部含有unsafe内容，这个函数无需是unsafe声明，即安全抽象包裹unsafe代码
### 调用c标准库
```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```
### 编写可以被调用的rust
```rust

#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```
- 将其编译成一个共享库，然后链接到 C 语言中。
- `#[no_mangle]`它用于告诉 Rust 编译器：不要乱改函数的名称。
## 访问或修改一个可变的静态变量
## 实现了一个unsafe特征
## 访问union中字段
## 一些实用库
rust-bindgen,cbindgen,cxx,Miri,Clippy,Prusti,模糊测试
