---
title: rust学习——生命周期
date: 2022-05-26 12:40:30
subtitle:
categories:
tags:
cover:
---
## 生命周期几个注意点
- 在存在多个**引用**时，编译器有时会无法自动推导生命周期，此时就需要我们手动去标注，通过为参数标注合适的生命周期来帮助编译器进行借用检查分析
- 标记生命周期只是为了取悦编译器，让编译器不要难为我们
## 函数中的生命周期
- 函数参数的手动标注的生命周期表示：参数至少活得和`'a`一样久，至于谁更久，无法得知
- 函数的返回值如果是一个引用类型，那么它的生命周期只会来源于：
	- 函数参数的生命周期
	- 函数体中的某个新建引用的生命周期
第二种情况明显是错误，典型的悬垂引用
### 使用说明
- 格式：`fn func<'a> (para1: &'a type,para2: &'a type)-> &'a type{}`
- 说明：**在通过函数签名指定生命周期参数时，我们并没有改变传入引用或者返回引用的真实生命周期，而是告诉编译器当不满足此约束条件时，就拒绝编译通过。**
- 含义：当传递具体值时，'a的生命周期等于para1和para2(取两个参数的交集)，返回的生命周期也等于'a,若不满足条件就报错
### 三条默认添加生命周期规则
函数或方法中，参数的生命周期被称为输入生命周期，返回值的生命周期被称为输出生命周期。rust有以下三条规则(你可以手动标注生命周期改变一下规则):
- 每一个引用参数都会获得独自的生命周期
`fn foo(a:& i32,y:& i32)`编译器自动转成`foo<'a, 'b>(x: &'a i32, y: &'b i32)`
- 若只有一个输入生命周期(函数参数中只有一个引用类型)，那么该生命周期会被赋给所有的输出生命周期，也就是所有返回值的生命周期都等于该输入生命周期
`fn foo(x: &i32) -> &i32`会转为`fn foo<'a>(x: &'a i32) -> &'a i32`
- 若存在多个输入生命周期，且其中一个是 &self 或 &mut self，则 &self 的生命周期被赋给所有的输出生命周期
## 结构体中的生命周期
- 结构体中存在引用，就必须手动标注生命周期
- 格式：
```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```
- 含义：结构体ImportantExcerpt所引用的字符串str必须比改结构体活得长
## 生命周期约束
- `'a:'b`：a的生命周期大于等于b
例子:
```rust
impl<'a: 'b, 'b> ImportantExcerpt<'a> {
    fn announce_and_return_part(&'a self, announcement: &'b str) -> &'b str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```
等价于：
```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part<'b>(&'a self, announcement: &'b str) -> &'b str
    where
        'a: 'b,
    {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```
- `T:'a`：类型T必须比a生命周期大
## 再借用
可变引用和不可变引用不能同时存在(即作用域不重叠)，但再借用可以避免,但是你不能在它的生命周期内再使用原来的借用，如下:
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
## 困惑
圣经那本书，最后一个复杂例子中方法中使用引用，会自动推断参数引用的生命周期的和引用内容生命周期一样(第一个例子，更是返回类型来推断参数类型！！！),可能生命周期交集的算法只适用于函数,最后一个例子也叫我们可以直接给&mut self一个不等于实际本身内容的生命周期解决,感觉是手动标注来决定它们的生命周期，与函数的相矛盾。
## 静态生命周期
- `&'static`生命周期针对的仅仅是引用，而不是持有该引用的变量，变量仍要遵循相应的作用域规则，例子如下：
```rust
use std::{slice::from_raw_parts, str::from_utf8_unchecked};

fn get_memory_location() -> (usize, usize) {
  // “Hello World” 是字符串字面量，因此它的生命周期是 `'static`.
  // 但持有它的变量 `string` 的生命周期就不一样了，它完全取决于变量作用域，对于该例子来说，也就是当前的函数范围
  let string = "Hello World!";
  let pointer = string.as_ptr() as usize;
  let length = string.len();
  (pointer, length)
  // `string` 在这里被 drop 释放
  // 虽然变量被释放，无法再被访问，但是数据依然还会继续存活
}

fn get_str_at_location(pointer: usize, length: usize) -> &'static str {
  // 使用裸指针需要 `unsafe{}` 语句块
  unsafe { from_utf8_unchecked(from_raw_parts(pointer as *const u8, length)) }
}

fn main() {
  let (pointer, length) = get_memory_location();
  let message = get_str_at_location(pointer, length);
  println!(
    "The {} bytes at 0x{:X} stored: {}",
    length, pointer, message
  );
  // 如果大家想知道为何处理裸指针需要 `unsafe`，可以试着反注释以下代码
  // let message = get_str_at_location(1000, 10);
}
```
- `T:'static`意味着接收方可以安全地持有T（即使用引用）,因为任何都是总是小于全局生命周期的,不存在悬垂引用,一般用如下用法：
```rust
use std::fmt::Debug;
fn print_it<T: Debug + 'static>( input: &T) {
    println!( "'static value passed in is: {:?}", input );
}

fn main() {
    let i = 5;

    print_it(&i);
}
```
**这里使用&T,函数体内更本没有直接使用T,因此就没去检查T的生命周期约束，他只要确保&T生命周期符合即可（肯定符合,确保不会出现悬垂引用？）**
而不是下面的用法：
```rust
use std::fmt::Debug;
fn print_it<T: Debug + 'static>( input: T) {
    println!( "'static value passed in is: {:?}", input );
}
fn print_it1( input: impl Debug + 'static ) {
    println!( "'static value passed in is: {:?}", input );
}



fn main() {
    let i = 5;
//这里会报错
    print_it(&i);
    print_it1(&i);
}
```
