---
title: rust学习——智能指针
date: 2022-06-05 10:07:21
subtitle:
categories:
tags:
cover:
---
## deref特征
- 在表达式中必须显式的解引用，并且不会发生递归的解引用，当指明时，有几层就解几层,并且本质上是`*y`=>`*(y.deref())`,y.deref()返回一个引用，返回值会产生所有权问题
- 函数和方法中只有引用类型的实参才能触发自动解引用
- 隐式的解引用，会递归，直到类型匹配
- 当 T: Deref<Target=U>，可以将 &T 转换成 &U
- 当 T: DerefMut<Target=U>，可以将 &mut T 转换成 &mut U
- 当 T: Deref<Target=U>，可以将 &mut T 转换成 &U
- 要实现 erefMut必须要先实现Deref特征：`pub trait DerefMut: Deref`
### 手动实现
```rust
struct MyBox<T> {
    v: T,
}

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox { v: x }
    }
}

use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.v
    }
}

use std::ops::DerefMut;

impl<T> DerefMut for MyBox<T> {
    fn deref_mut(&mut self) -> &mut Self::Target {
        &mut self.v
    }
}

fn main() {
    let mut s = MyBox::new(String::from("hello, "));
    display(&mut s)
}

fn display(s: &mut String) {
    s.push_str("world");
    println!("{}", s);
}
```
## Rc与Arc
Rc用于单线程，Arc用于多线程，都是通过引用计数，允许一个数据资源同时拥有多个所有者。**本质上是指向底层数据的不可变的引用，无法通过它来修改数据,需要通过其他类型的配合，比如Mutex,RefCell**
- 分别通过`use std::sync::Arc;`,`use std::rc::Rc`,引入
### 创建
`let a = Rc::new(String::from("hello"));`
### clone
`let b = Rc::clone(&a)`
### 查看计数
`Rc::strong_count(&a)`
## Cell与RefCell
- 这两个可以在拥有不可变引用的同时修改目标数据。
- Cell和RefCell的区别在于，Cell<T>使用于T实现了Copy的情况,
- 这个只是跳过编译阶段的检查（貌似就这个功能），运行阶段如果违反（可变引用与不可变引用同时存在）该报错还报错
- `use std::cell::Cell`,`use std::cell::RefCell`
### 定义
- `let c = Cell::new("hello")`
- `let s = RefCell::new(String::from("hello"))`
### 复制获得
- `let one = c.get()`
- `let s1 = s.borrow()`
### 更改
- `c.set("你好")`
- `let s1 = s.borrow_mut()`
### 内部可变性
简而言之，方法中self不可变，但是该方法内部可以调用RefCell达到改变的目的：
```rust
use std::cell::RefCell;
pub trait Messenger {
    fn send(&self, msg: String);
}

pub struct MsgQueue {
    msg_cache: RefCell<Vec<String>>,
}

impl Messenger for MsgQueue {
    fn send(&self, msg: String) {
        self.msg_cache.borrow_mut().push(msg)
    }
}

fn main() {
    let mq = MsgQueue {
        msg_cache: RefCell::new(Vec::new()),
    };
    mq.send("hello, world".to_string());
}
```
### from_mut解决借用冲突
- `Cell::from_mut`,将`&mut T`转为`&Cell<T>`
- `Cell::as_slice_of_cells`,将`&Cell<[T]>`转为`&[Cell<T>]`
